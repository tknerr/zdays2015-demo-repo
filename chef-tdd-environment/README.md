
## Chef TDD Environment

The environment we need for Test-Driven Development or ["Test-Driven Infrastructure with Chef"](http://www.amazon.com/Test-Driven-Infrastructure-Chef-Behavior-Driven-Development/dp/1449372201)
is basically a set of tools, which we want to briefly introduce here:

 * [rubocop](https://github.com/bbatsov/rubocop) is a Ruby-level linting tool
 * [foodcritic](https://acrmp.github.io/foodcritic/) is a linting tool on Chef level
 * [chefspec](https://github.com/sethvargo/chefspec) + [fauxhai](https://github.com/customink/fauxhai) is for unit testing Chef cookbooks / mocking platforms
 * [test-kitchen](https://github.com/test-kitchen/test-kitchen) + [serverspec](http://serverspec.org) is an integration testing framework / library for testing servers

All of these tools come bundled with [ChefDK](https://downloads.chef.io/chef-dk/),
the "Chef Development Kit", in a specific version:
```
$ chef -v
Chef Development Kit Version: 0.13.21
chef-client version: 12.9.41
berks version: 4.3.2
kitchen version: 1.7.3

$ rubocop -v
0.37.2

$ foodcritic -V
foodcritic 6.1.1

$  kitchen -v
Test Kitchen version 1.7.3
```

ChefSpec, Fauhai and Serverspec are only libraries, not commandline tools. They
are also bundled at a very specific version:
```
$ gem list chefspec fauxhai serverspec --no-details

*** LOCAL GEMS ***

chefspec (4.6.1)

*** LOCAL GEMS ***

fauxhai (3.2.0)

*** LOCAL GEMS ***

serverspec (2.31.1)
```

### Using a Gemfile to lock your Environment

Having all the tools bundled with ChefDK is comfortable for getting started quickly,
but also comes at a cost: you might have updated your ChefDK in the meanwhile and
once you work with an older cookbook nothing might work anymore!

In order to avoid that, you should create a `Gemfile` (for each cookbook structure / repository)
to specify all your development dependencies:
```ruby
source 'https://rubygems.org'

gem 'chef', '12.9.41'
gem 'berkshelf', '4.3.2'

group :test do
  gem 'foodcritic', '6.2.0'
  gem 'rubocop', '0.40.0'
  gem 'chefspec', '4.6.1'
end

group :integration do
  gem 'test-kitchen', '1.8.0'
  gem 'kitchen-docker', '2.4.0'
  gem 'kitchen-vagrant', '0.20.0'
  gem 'serverspec', '2.34.0'
end
```

Once you run `bundle install` [bundler](bundler.io) will create an `Gemfile.lock`
with all the dependencies (and their transitive dependencies) locked to the resolved
version. If the `Gemfile.lock` already exists, it will use exactly the versions specified therein.

Now whenever you run a command prefixed with `bundle exec <command>`, it will only
have the gems specified in the `Gemfile.lock` in the LOAD_PATH and thus see only the
specific versions, ignoring any other installed gems on your system:
```
$ bundle exec foodcritic -V
foodcritic 6.2.0

$ bundle exec rubocop -v
0.40.0
```

Essentially, this allows your cookbook projects to safely evolve independently from each other.

### Pro Tip: use a Rakefile

A `Rakefile` in Ruby is quite similar to a Makefile -- it let's you define the tasks
your are expected to run in a central place. It also integrates quite nicely with
bundler, so that you don't have to specify the `bundle exec` prefix anymore when you
run a rake task.

This is a `Rakefile` like I commonly use for cookbook projects:
```ruby
require 'bundler/setup'

desc 'check Ruby code style with rubocop'
task :rubocop do
  sh 'rubocop . --format progress --format offenses'
end

desc 'run foodcritic lint checks'
task :foodcritic do
  sh 'foodcritic -f any .'
end

desc 'run chefspec examples'
task :chefspec do
  sh 'rspec --format doc --color'
end

desc 'run test-kitchen integration tests'
task :integration do
  sh 'kitchen test --log-level info'
end

desc 'run all unit-level tests'
task :unit => [:rubocop, :foodcritic, :chefspec]
```

Whenever you see a `Rakefile` around, you can list the available tasks via `rake -T`:
```
$ rake -T
rake chefspec     # run chefspec examples
rake foodcritic   # run foodcritic lint checks
rake integration  # run test-kitchen integration tests
rake rubocop      # check code style with rubocop
rake unit         # run all unit-level tests
```

Running a task then becomes as simple as running `rake foodcritic` or `rake unit`.
