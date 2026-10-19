
Upgrading this repo which depends on a terribly out-of-date set of gems

## dependencies
Some ruby headers are required, on debian-like systems make sure you have:
```sh
$ sudo apt install ruby-dev
```

Ruby packages needed are:
```sh
$ cat Gemfile
source "https://rubygems.org"
gem "ffi", ">= 1.9.24"
gem "jekyll", ">= 3.7.4"
gem "minima", "~> 2.0"
gem "nokogiri", ">= 1.8.5"
gem "rubyzip", ">= 1.3.0"
gem "github-pages", group: :jekyll_plugins
```

## local vendor bundler config
Configure `bundler` to use gems installed in the current directory, not system libs:
```sh
$ bundle config --local path vendor
# I don't know if both previous & following commands are necessary.
# Previous was used before issuing `bundler install --binstubs --path vendor`
# Following two come from deprecation warnings about `--binstubs` & `--path` flags
$ bundle config set path 'vendor'
$ bundle binstubs --all
```
It's been years since I've kept up with ruby gem management, make sure `disable_shared_gems` is true:
```sh
$ bundler config list
Settings are listed in order of priority. The top value will be used.
bin
Set for your local app (/home/kenneth/git/kennethd.github.io/.bundle/config): "bin"

disable_shared_gems
Set for your local app (/home/kenneth/git/kennethd.github.io/.bundle/config): true

path
Set for your local app (/home/kenneth/git/kennethd.github.io/.bundle/config): "vendor"
``` 
And `.bundle/config` should look something like:
```sh
$ cat .bundle/config 
---
BUNDLE_PATH: vendor
BUNDLE_BIN: bin
BUNDLE_DISABLE_SHARED_GEMS: '1'
```

## upgrade packages
Move old `Gemfile.lock` out of the way
```sh
$ mv Gemfile.lock Gemfile.lock.OLD
```

Upgrade bundler itself in the local `vendor` directory
```sh
$ gem install --install-dir ./vendor  bundler
```
Check we are invoking expected version of `bundler`
```sh
$ ./bin/bundle --version 
Bundler version 2.6.3
```

Install gems
```sh
$ ./bin/bundle install 
Updating files in vendor/cache
Bundle complete! 6 Gemfile dependencies, 99 gems now installed.
Bundled gems are installed into `./vendor`
```

Check BUNDLED WITH value of new `Gemfile.lock`
```sh
$ tail Gemfile.lock
DEPENDENCIES
  ffi (>= 1.9.24)
  github-pages
  jekyll (>= 3.7.4)
  minima (~> 2.0)
  nokogiri (>= 1.8.5)
  rubyzip (>= 1.3.0)

BUNDLED WITH
   2.6.3
```

## update .gitignore & add missing files (first-time only)
Configure `git` to ignore locally installed `vendor` libs, add new config
```
echo /vendor >>.gitignore
echo /bin >>.gitignore
echo '/Gemfile.lock.*' >>.gitignore
git add .bundle/config
git add Gemfile.lock
```

## test build
Start the Jekyll app locally to make sure it all looks good
```sh
$ bundle exec jekyll serve --host=0.0.0.0
onfiguration file: /home/kenneth/git/kennethd.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /home/kenneth/git/kennethd.github.io
       Destination: /home/kenneth/git/kennethd.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.399 seconds.
 Auto-regeneration: enabled for '/home/kenneth/git/kennethd.github.io'
    Server address: http://0.0.0.0:4000/
  Server running... press ctrl-c to stop.
```
If you are running the server on the same device you are testing from (127.0.0.1),
you won't need the `--host 0.0.0.0` argument; I am testing across my home network

## resources

 * [https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)

