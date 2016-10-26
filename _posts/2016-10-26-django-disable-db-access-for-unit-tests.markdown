---
layout: post
title:  "Disable DB Access for Unit Tests"
date:   2016-10-26 11:00:00 -4000
categories: testing
---

# Disable Django DB Access for Unit Tests

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


