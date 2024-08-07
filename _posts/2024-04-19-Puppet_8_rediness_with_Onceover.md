---
title: Puppet 8 rediness with Onceover
date: 2024-05-09 11:00:00 -0000
categories: [Puppet Enterprise]
tags: [Puppet, Open Source Puppet, Onceover, PE, OSP]
---

# Puppet 8 readiness with Onceover

Puppet 8 has been released, with it comes security enhancements, the dropping of deprecated features, and performance improvements, you can read more about Puppet 8 on this [blog post](https://www.puppet.com/blog/puppet-8).

In this blog post, well look at how you can take your existing Puppet 7 code and use [Onceover](https://github.com/voxpupuli/onceover), a free testing tool, to test it against Puppet 8 to prepare for your migration.

## Puppet 8

Of the many changes to Puppet 8, a few will have a direct impact on your current Puppet code.
Strict mode will now be enforced, for example, in Puppet 7 you could add a string to an integer, ```"1" + 1```, this will now fail.
Legacy facts, which have been deprecated for some time, have been removed. 
Hiera 3 has also been removed.

## Onceover

The simple question we want to answer is "will my Puppet 7 code compile into a catalog on Puppet 8" and for that, [Onceover](https://github.com/voxpupuli/onceover) is the perfect tool. If you haven't used [Onceover](https://github.com/voxpupuli/onceover) before, it's a stand alone tool that you can run on any machine, and test that the roles and profiles in your control-repo will compile into catalogs.

### Install Ruby 3

The first thing we'll need to do is install Ruby 3, it's a requirement of Puppet 8. Follow these steps to install it [Installing Ruby](https://www.ruby-lang.org/en/documentation/installation/).

If you're on your own workstation you may want to consider using [rbenv](https://github.com/rbenv/rbenv) to allow you to quickly switch between versions of Ruby that your other project may depend upon.

### Install Onceover

The first step in installing [Onceover](https://github.com/voxpupuli/onceover) is to clone your control-repo onto your system.

Then install install bundler ```gem install bundler``` this is required to read the Gemfile we'll produce in the next step to install the required Gems.

In the root of your control-repo create a Gemfile, this specifies the main Gems we want to install and their versions. If you wanted to use a different version of Puppet, just change the version in the Gemfile. If you host Gems internaly such as Artifatory, you can specify the source in here too.

```
# Gemfile
source "https://rubygems.org"
gem 'puppet', '~> 8.4'
gem 'onceover', '~> 3.22'
```

Run ```bundle install``` to install all the gems and their dependencies.

Run ```bundle exec onceover init``` to initialise the control-repo, one of the things this does is create the .onceover directory and puts the [Onceover](https://github.com/voxpupuli/onceover) config file, onceover.yaml, into the spec directory.

### Run a test

To keep things simple, my control-repo contains one role.

```
# site-modules/role/manifests/example.pp
class role::example {
  include profile::coercion.pp
}
```

And that role will contain one profile, this profile contains some code that would compile a catalog under Puppet 7, but not under Puppet 8.

```
# site-modules/profile/manifests/coercion.pp
class profile::coercion {
  $result = '1' + 1
}
```

The last job is to modify the [Onceover](https://github.com/voxpupuli/onceover) configutation file, to keep this simple I just want it to compile the example role on Ubuntu 20.04.

```
classes:
  - role::example

nodes:
  - Ubuntu-20.04-64

node_groups:
  non_windows_nodes:
    - Ubuntu-20.04-64

test_matrix:
  - all_nodes:
      classes: 'all_classes'
      tests: 'spec'
```

Finaly, to run the test, run ```onceover run spec```.
This will return the following error, which is expected, as Puppet 8 enforces [strict mode](https://www.puppet.com/docs/puppet/8/upgrading-from-puppet7-to-puppet8.html#upgrading-from-puppet7-to-puppet8-legacy-strict-mode) on coercion. 
```
role::example: F

role::example: failed
  errors:
    Evaluation Error: The string '1' was automatically coerced to the numerical value 1
      file: site-modules/profile/manifests/coercion.pp
      line: 3
      column: 13
      factsets: Ubuntu-20.04-64
```

### Refactor our code and rerun the test

The error is reasonably clear, we provided a string and it was used as an integer. Lets fix that code!

```
# site-modules/profile/manifests/coercion.pp
class profile::coercion {
  $result = 1 + 1
}
```

The result is a pass!

```
role::example: P
```

## Conclusion

Using [Onceover](https://github.com/voxpupuli/onceover) and the Puppet 8 Gem, we can quickly detect code that will not work on Puppet 8 and we can do this on a test machine or laptop, well away from our Puppet servers and production environment.

## A note on Continuous Delivery for Puppet Enterprise (CD4PE)

If you're using [Continuous Delivery](https://help.puppet.com/cdpe/) you're probably already using [Onceover](https://github.com/voxpupuli/onceover) on the [puppet-dev-tools:4.x](https://hub.docker.com/r/puppet/puppet-dev-tools/tags) container that ships with [Continuous Delivery](https://help.puppet.com/cdpe/). To start testing against Puppet 8, duplicate the [Onceover](https://github.com/voxpupuli/onceover) job you're already running, but change the Docker container it uses to [puppet-dev-tools:puppet8](https://hub.docker.com/layers/puppet/puppet-dev-tools/puppet8/images/sha256-6da1c6cdde55a3b174a6042b0e724ce0340d146f616e4ea1853aee78d4e87677?context=explore)
