---
layout: post
title: Managing Ruby Versions On MacOS  
subtitle:
author: dch
categories: 
tags: macos ruby
sidebar: []
---

Managing ruby installations on MacOS can be a pain, as the OS ships with default system versions that are outdated. 

After using [`rbenv`](https://github.com/rbenv/rbenv) to manage non-system rubies for a couple of years, I've finally settled on what I find to be a simpler approach: using a combination of [`chruby`](https://github.com/postmodern/chruby) and [`ruby-install`](https://github.com/postmodern/ruby-install).

### Installing Packages

Both packages can be installed via [homebrew](https://brew.sh) or built from source using the tarballs on github.

~~~sh
brew install chruby ruby-install
~~~

### Installing Rubies

At this point, the ruby binary is still system as demonstrated with `which ruby`. It's also apparent that the system version is quite outdated:

~~~sh
% which ruby
/usr/bin/ruby

% ruby --version
ruby 2.6.10p210 (2022-04-12 revision 67958) [universal.arm64e-darwin21]
~~~

Running `ruby-install` can install alternative rubies, storing them in the `~/.rubies/` directory.

~~~sh
% ruby-install 3.1.0
...
...
>>> Successfully installed ruby 3.1.0 into /Users/{$user}/.rubies/ruby-3.1.0
~~~

***A Note on MacOS Monterey / Ventura...***

It seems ruby-install has trouble installing some versions of ruby on MacOS Monterey, throwing errors and terminating the build:

~~~sh
% ruby-install 2.7.3
...
...
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [../../.ext/arm64-darwin21/strscan.bundle] Error 1
make[1]: *** [ext/strscan/all] Error 2
make: *** [build-ext] Error 2
!!! Compiling ruby 2.7.3 failed!
~~~

This can be circumvented by adding the `-- --enable-shared` flag to the ruby-install command[^1]

~~~sh
% ruby-install 2.7.3 -- --enable-shared
...
...
>>> Successfully installed ruby 2.7.3 into /Users/{$user}/.rubies/ruby-2.7.3
~~~

### Environment Setup

A list of installed rubies can be viewed by invoking `chruby` with no arguments, with the current ruby version marked with an asterisk. 

Switching versions is done simply by passing chruby the desired version number.

~~~sh
% chruby         
   ruby-2.7.3
 * ruby-2.7.7
   ruby-3.1.1
   ruby-3.1.2

% chruby 2.7.3
% chruby
 * ruby-2.7.3
   ruby-2.7.7
   ruby-3.1.1
   ruby-3.1.2

% which ruby
/Users/{$user}/.rubies/ruby-2.7.3/bin/ruby
% ruby --version
ruby 2.7.3p183 (2021-04-05 revision 6847ee089d) [arm64-darwin21]
~~~

### Persisting Ruby Settings

It's very simple to persist ruby settings with chruby. To set a preferred ruby to be used across sessions, first include a path to `chruby.sh` in shell resource file.  

~~~sh
% echo source /opt/homebrew/opt/chruby/share/chruby/chruby.sh >> ~/.zshrc
~~~

*replace .zshrc with .bash_profile if using bash, etc.*

Then set the default ruby by appending `chruby {$version}` to the resource file.
~~~sh
% echo chruby 2.7.3 >> ~/.zshrc
~~~

After reloading the session, the default ruby should be set to the ruby specified in the resource file. 


[^1]: https://www.rubyonmac.dev/how-to-install-ruby-on-macos-12-6-apple-silicon