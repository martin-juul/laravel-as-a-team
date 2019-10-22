# Introduction

Opinionated style guide for maximizing readability and less buggy code.

## Laravel Support Matrix

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 5.5 (LTS) | August 30th, 2017 | August 30th, 2019 | August 30th, 2020 |
| 5.6 | February 7th, 2018 | August 7th, 2018 | February 7th, 2019 |
| 5.7 | September 4th, 2018 | March 4th, 2019 | September 4th, 2019 |
| 5.8 | February 26th, 2019 | August 26th, 2019 | February 26th, 2020 |
| 6 (LTS) | September 3rd, 2019 | September 3rd, 2021 | September 3rd, 2022 |

## How to read this book

First you should read the laravel documentation, on the version you're currently or going to work with.

* [6.x](https://laravel.com/docs/6.x)
* [5.8](https://laravel.com/docs/5.8)
* [5.7](https://laravel.com/docs/5.7)
* [5.6](https://laravel.com/docs/5.6)
* [5.5](https://laravel.com/docs/5.5)

Now read it again. No seriously, do it.

## Tooling

Then we'll configure your dev environment.

### IDE

 Sublime and it's clones does NOT properly interpret the language.

[nikic](https://nikic.github.io/aboutMe.html) arguably **the** most beloved of the the contributors to php-src, works at Jetbrains improving the IDE.

PHPStorm is the only _php_ oriented IDE which has first class language support.

Newer version like release candidates are already implemented _before_ release date on php.net

It'll also auto match the code style used in the project.

**⚠ Use PHPStorm. End of discussion. ⚠**

#### Plugins

* [Laravel](https://plugins.jetbrains.com/plugin/7532-laravel/)
* [.​env files support](https://plugins.jetbrains.com/plugin/9525--env-files-support/)
* [Makefile support](https://plugins.jetbrains.com/plugin/9333-makefile-support/)
* [PHP Annotations](https://plugins.jetbrains.com/plugin/7320-php-annotations/)
* [Php Inspections ​(EA Extended)](https://plugins.jetbrains.com/plugin/7622-php-inspections-ea-extended-/)
* [String Manipulation](https://plugins.jetbrains.com/plugin/2162-string-manipulation/)
* [Symfony Support](https://plugins.jetbrains.com/plugin/7219-symfony-support/)

Make sure Editorconfig is enabled. It's bundled with the IDE.

## PHP Versions

In PHPStorm set the language level to the one defined in your laravel projects `composer.json`

_Preferences -> Languages & Frameworks -> PHP -> PHP language level_

## Homestead or Valet?

In production the app will run under Linux. Usually in a docker container, based on Alpine (barebones busybox distro) or debian-slim.
PHP on windows is no bueno.

You could use docker on macOS, but performance would be seriously lacking. This is a known issue with this stack.
Symfony relies heavily on IO performance, in dev environments as the service container is recompiled on every request.
This is also why it's compiled together with config, views and routes in production.

* [#2707 - Docker for Mac + Symfony too slow](https://github.com/docker/for-mac/issues/2707)

You'll also need at least a database (Postgres), cache (Redis) and probably an smtp server.

It makes most sense to use [Laravel Homestead](https://laravel.com/docs/6.x/homestead) - it's a vm with batteries included.

I recommend installing it on a [Per Project Installation](https://laravel.com/docs/6.x/homestead#per-project-installation) basis, so everyone has the same box on their machine.

### Installing Homestead on macOS

First install homebrew, if you don't have it. Open up a terminal and run:

```shell script
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Next we need virtualbox, the extension package and vagrant

```shell script
brew cask install cask virtualbox
brew cask install cask virtualbox-extension-pack
brew cask install vagrant
```

Then we'll install a couple of nifty plugins to vagrant

__vagrant-hostsupdater__

Updates your hostsfile automagically, so you can access your vm with vm-hostname.test
```shell script
vagrant plugin install vagrant-hostsupdater
```

__vagrant-vbguest__

Makes sure the virtual machine has the same version, of virtualbox tools and your host.
```shell script
vagrant plugin install vagrant-vbguest
```

### Performance tip

When adding folders in `Homestead.yml` configure it to use `nfs` to get optimal io performance.

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```
