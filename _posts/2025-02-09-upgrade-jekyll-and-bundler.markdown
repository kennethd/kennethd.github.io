
Upgrading this repo which depends on a terribly out-of-date set of gems
```
kenneth@fado ~/git/kennethd.github.io (master) $ cat Gemfile
source "https://rubygems.org"
gem "ffi", ">= 1.9.24"
gem "jekyll", ">= 3.7.4"
gem "minima", "~> 2.0"
gem "nokogiri", ">= 1.8.5"
gem "rubyzip", ">= 1.3.0"
gem "github-pages", group: :jekyll_plugins
```

Some ruby headers are required, on debian-like systems make sure you have:
```
sudo apt install ruby-dev
```

Configure `bundler` to use gems installed in the current directory, not system libs:
```
bundle config --local path vendor
bundler install --binstubs --path vendor
```
`.bundle/config` should look something like:
```
kenneth@fado ~/git/kennethd.github.io (master) $ cat .bundle/config 
---
BUNDLE_PATH: vendor
BUNDLE_BIN: bin
BUNDLE_DISABLE_SHARED_GEMS: '1'
```

Upgrade bundler itself in the local `vendor` directory
```
gem install --install-dir ./vendor  bundler
```
Check we are invoking expected version of `bundler`
```
kenneth@fado ~/git/kennethd.github.io (master) $ ./bin/bundle --version 
Bundler version 2.6.3
```

Move old `Gemfile.lock` out of the way
```
mv Gemfile.lock Gemfile.lock.OLD
```

Install gems
```
kenneth@fado ~/git/kennethd.github.io (master) $ ./bin/bundle install 
Updating files in vendor/cache
Bundle complete! 6 Gemfile dependencies, 99 gems now installed.
Bundled gems are installed into `./vendor`

kenneth@fado ~/git/kennethd.github.io (master) $ tail Gemfile.lock
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

Configure `git` to ignore locally installed `vendor` libs, add new config
```
echo /vendor >>.gitignore
echo /bin >>.gitignore
echo '/Gemfile.lock.*' >>.gitignore
git add .bundle/config
git add Gemfile.lock
```

