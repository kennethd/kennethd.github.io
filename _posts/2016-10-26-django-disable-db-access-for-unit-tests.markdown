---
layout: post
title:  "Disable DB Access for Speedy Unit Tests"
date:   2016-10-26 11:00:00 -4000
categories: testing
---

# Disable Django DB Access for Speedy Unit Tests

A recurring problem in my career over the past few years has been introducing
testing into legacy django apps which tend to suffer the problems of a typical
"monolythic app".

Django comes with great built-in support for integration-style tests, with
in-memory databases standing in for actual databases, all of your migrations
running anew per test, and so on.

I have yet to meet a django app with well-enough maintained migrations for
this to be possible; first because maintaining historical rebuilding of your
external resources tends to be a low priority in environments which embrace
the framework for its enabling of fast-moving iterations, as it probably
should be, and second because it makes running the tests so slow as to
discourage developers from actually running them.

Recently finding myself in such an environment again, I had created certain
pieces of code I would like to unit test (not integration, simply test certain
heavily shared individual functions to ensure that their behavior not
inadvertently change when working on the specific requirements of one of the
users of said functions).

Inspired by a few
[blog](http://blog.celerity.com/how-to-write-speedy-unit-tests-in-django-part-1-the-basics)
[posts](http://blog.celerity.com/unit-testing-django-fake-it-til-you-make-it)
about speeding up tests in django, in part by
[disabling db access](http://codeinthehole.com/writing/disable-database-access-when-writing-unit-tests-in-django/)
and a few about
[mock](http://www.mattjmorrison.com/2011/09/mocking-django.html)
[strategies](https://www.toptal.com/python/an-introduction-to-mocking-in-python)
I created the following base class for my unit tests:

{% highlight python lineno %}

import mock
import unittest

import gas

from django.db.backends.utils import CursorWrapper

disabled_cursor = mock.Mock()
disabled_cursor.side_effect = RuntimeError("db access disabled")
disabled_cursor.WRAP_ERROR_ATTRS = CursorWrapper.WRAP_ERROR_ATTRS

# raise error if test code attempts to access db
@mock.patch("django.db.backends.utils.CursorWrapper", disabled_cursor)
@mock.patch("django.db.backends.utils.CursorDebugWrapper", disabled_cursor)
class BaseGasVolumesTestCase(unittest.TestCase):

    # this method intentionally not mock.patch()ed to prevent db access,
    # it will run for all subclasses, ensuring none can access the db
    def test_db_access_raises_error(self):
        from gas.queries import get_clients
        with self.assertRaises(RuntimeError) as ctxt:
            cus = get_clientutilities()
        self.assertEqual(ctxt.exception.message, "db access disabled")

{% endhighlight %}

Each of your tests then must mock out each of the methods it invokes which
would otherwise touch the database:

{% highlight python %}
# raise error if test code attempts to access db
@mock.patch("django.db.backends.utils.CursorWrapper", disabled_cursor)
@mock.patch("django.db.backends.utils.CursorDebugWrapper", disabled_cursor)
class TestGasVolumesForecastedDemand(BaseGasVolumesTestCase):

    @mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.save', side_effect=None)
    @mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.save', side_effect=None)
    def test_save_forecasteddemand(self, monthly_save, daily_save):

        # behavior when insert is performed on db

        with mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_isnew_true):
            (fd, isnew) = save_forecasteddemand('daily', '2016-10-31', 48, 87, 10, 'Utility Group', 99)
            self.assertTrue(isinstance(fd, DailyForecastedDemand))
            self.assertTrue(isnew)

        with mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_isnew_true):
            (fd, isnew) = save_forecasteddemand('monthly', '2016-10-01', 48, 87, 310, 'Utility Group')
            self.assertTrue(isinstance(fd, MonthlyForecastedDemand))
            self.assertTrue(isnew)
            with self.assertRaises(AttributeError):
                max_dths = fd.max_dths

        # behavior when record exists and values haven't changed

        with mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_isnew_false):
            (fd, isnew) = save_forecasteddemand('daily', '2016-10-31', 48, 87, 10, 'Utility Group', 99)
            self.assertTrue(isinstance(fd, DailyForecastedDemand))
            self.assertFalse(isnew)

        with mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_isnew_false):
            (fd, isnew) = save_forecasteddemand('monthly', '2016-10-01', 48, 87, 310, 'Utility Group')
            self.assertTrue(isinstance(fd, MonthlyForecastedDemand))
            self.assertFalse(isnew)
            with self.assertRaises(AttributeError):
                max_dths = fd.max_dths

        # behavior when record exists and non-key values have changed

        with mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_raises_integrity_error):
            with mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.objects.get',
                            side_effect=get):
                (fd, isnew) = save_forecasteddemand('daily', '2016-10-31', 48, 87, 10, 'Utility Group', 99)
                self.assertTrue(isinstance(fd, DailyForecastedDemand))
                self.assertFalse(isnew)


        with mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_raises_integrity_error):
            with mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.objects.get',
                            side_effect=get):
                (fd, isnew) = save_forecasteddemand('monthly', '2016-10-01', 48, 87, 310, 'Utility Group')
                self.assertTrue(isinstance(fd, MonthlyForecastedDemand))
                self.assertFalse(isnew)
                with self.assertRaises(AttributeError):
                    max_dths = fd.max_dths


    @mock.patch("gas.volumes.forecasteddemand.get_utility_by_id", return_value=CONED)
    @mock.patch("gas.volumes.forecasteddemand.get_client_by_id", return_value=None)
    def test_fetch_demand_for_client_raises_bad_creds(self, client_by_id, utility_by_id):
        gas.volumes.forecasteddemand.get_logincredentials._creds = MOCK_LOGINCREDENTIALS
        with self.assertRaises(ValueError) as ctxt:
            fds = fetch_demand_for_client('2016-08-01', 14, -1, dummy_generator)
        self.assertEqual(ctxt.exception.message[:27], 'No login creds for (-1, 14)')


    @mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.save', side_effect=None)
    @mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.save', side_effect=None)
    @mock.patch('gas.volumes.forecasteddemand.DailyForecastedDemand.objects.get_or_create', side_effect=get_or_create_isnew_true)
    @mock.patch("gas.volumes.forecasteddemand.get_utility_by_id", return_value=CONED)
    @mock.patch("gas.volumes.forecasteddemand.get_client_by_id", return_value=ALPHA)
    @mock.patch("gas.volumes.forecasteddemand.get_logincredentials", return_value=ALPHACONED_LOGINCREDENTIALS)
    def test_fetch_demand_for_client(self, creds, client_by_id, utility_by_id,
                                             save_func, monthly_save, daily_save):

        with mock.patch('gas.volumes.forecasteddemand.MonthlyForecastedDemand.objects.get_or_create',
                        side_effect=get_or_create_isnew_true):
            fds = fetch_demand_for_client('2016-10-01',
                                          CONED['utility_id'],
                                          ALPHA['client_id'],
                                          alpha_coned_usage_generator)
            # (31 daily * 3 ugs) + (1 month * 3 ugs) = 96
            self.assertEqual(96, len(fds))
            for fd in fds:
                if isinstance(fd, DailyForecastedDemand):
                    if fd.utility_group == 'UG One':
                        self.assertEqual(2.0, fd.decatherms)
                    elif fd.utility_group == 'UG Tw0':
                        self.assertEqual(3.0, fd.decatherms)
                    elif fd.utility_group == '':
                        self.assertEqual(5.0, fd.decatherms)
                elif isinstance(fd, MonthlyForecastedDemand):
                    if fd.utility_group == 'UG One':
                        self.assertEqual(2.0 * 31, fd.decatherms)
                    elif fd.utility_group == 'UG Tw0':
                        self.assertEqual(3.0 * 31, fd.decatherms)
                    elif fd.utility_group == '':
                        self.assertEqual(5.0 * 31, fd.decatherms)
{% endhighlight %}

