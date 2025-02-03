# install
debian packages were throwing some errors I didn't care to debug, and I prefer
being on latest dev branch anyway, so going the `pip` route -- it also allows
me to bypass `systemd`, and just let Docker bring it up

https://deluge.readthedocs.io/en/latest/intro/01-install.html

First, we just need a place to put the virtualenv & whatever else configs we need to create
```
kenneth@fado ~ $ mkdir -p ~/git/deluge-headless
kenneth@fado ~ $ python3 --version
Python 3.11.2
kenneth@fado ~ $ cd ~/git/deluge-headless/
kenneth@fado ~/git/deluge-headless $ python3 -m venv ./venv-3.11
kenneth@fado ~/git/deluge-headless $ . ./venv-3.11/bin/activate
(venv-3.11) kenneth@fado ~/git/deluge-headless $ which pip
/home/kenneth/git/deluge-headless/venv-3.11/bin/pip
(venv-3.11) kenneth@fado ~/git/deluge-headless (main) $ pip install -U setuptools wheel
(venv-3.11) kenneth@fado ~/git/deluge-headless (main) $ pip install deluge
```
init a repo..
```
(venv-3.11) kenneth@fado ~/git/deluge-headless $ git init
Initialized empty Git repository in /home/kenneth/git/deluge-headless/.git/
(venv-3.11) kenneth@fado ~/git/deluge-headless $ echo '/venv*' >> .gitignore
(venv-3.11) kenneth@fado ~/git/deluge-headless $ git add .gitignore
(venv-3.11) kenneth@fado ~/git/deluge-headless $ git commit -m "initial commit"
[main (root-commit) e779ea0] initial commit
 1 file changed, 1 insertion(+)
  create mode 100644 .gitignore
  (venv-3.11) kenneth@fado ~/git/deluge-headless (main) $ 
```

