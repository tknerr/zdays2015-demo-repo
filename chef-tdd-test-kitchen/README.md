## Chef TDD: Integration Testing with Test-Kitchen

Test-kitchen is an integration testing tool for Chef cookbooks. It is basically
a test runner which converges a matrix of platforms and test suites for you. It
is commonly used with Serverspec which provides rspec matchers for testing servers:

 * [test-kitchen](https://github.com/test-kitchen/test-kitchen)
 * [serverspec](http://serverspec.org/)

### Running Test-Kitchen

Our "myapp" example cookbook already contains a test suite and configuration for
test-kitchen.

The suite is located in `myapp/test/integration/default/serverspec/default_spec.rb`,
and in fact does nothing spectacular yet:
```ruby
require 'spec_helper'

describe 'myapp::default' do
  # Serverspec examples can be found at
  # http://serverspec.org/resource_types.html
  it 'does something' do
    skip 'Replace this with meaningful tests'
  end
end
```

The configuration in `.kitchen.yml` is quite self-explanatory, but I usually
add some stuff to make it play nice with vagrant-cachier and use the correct
basebox when using the vagrant docker provider (see
[here](https://github.com/tknerr/sample-toplevel-cookbook/blob/master/.kitchen.yml)
for a more complete example):
```yaml
---
driver:
  name: vagrant

provisioner:
  name: chef_zero
  require_chef_omnibus: 12.9.41
  chef_omnibus_install_options: -d /tmp/vagrant-cache/vagrant_omnibus
  client_rb:
    add_formatter: min  

platforms:
  - name: ubuntu-14.04
    driver_config:
      box: tknerr/baseimage-ubuntu-14.04

suites:
  - name: default
    run_list:
      - recipe[myapp::default]
```

That's enough for running it, as usual via one of our Rake tasks:
```
$ rake integration
kitchen test --log-level info
-----> Starting Kitchen (v1.8.0)
-----> Cleaning up any prior instances of <default-ubuntu-1404>
-----> Destroying <default-ubuntu-1404>...
       Finished destroying <default-ubuntu-1404> (0m0.00s).
-----> Testing <default-ubuntu-1404>
-----> Creating <default-ubuntu-1404>...
       Bringing machine 'default' up with 'docker' provider...
       ==> default: Creating the container...
           default:   Name: kitchen-myapp-default-ubuntu-1404_default_1463015135
           default:  Image: tknerr/baseimage-ubuntu:14.04
           default: Volume: /home/vagrant/.vagrant.d/cache/tknerr/baseimage-ubuntu-14.04:/tmp/vagrant-cache
           default:   Port: 127.0.0.1:2222:22
           default:  
           default: Container created: e10d74ed6499ce4d
       ==> default: Starting container...
       ==> default: Waiting for machine to boot. This may take a few minutes...
           default: SSH address: 172.17.0.3:22
           default: SSH username: vagrant
           default: SSH auth method: private key
           default: 
           default: Vagrant insecure key detected. Vagrant will automatically replace
           default: this with a newly generated keypair for better security.
           default: 
           default: Inserting generated public key within guest...
           default: Removing insecure key from the guest if it's present...
           default: Key inserted! Disconnecting and reconnecting using new SSH key...
       ==> default: Machine booted and ready!
       ==> default: Configuring cache buckets...
       ==> default: Running provisioner: shell...
           default: Running: inline script
       ==> default: stdin: is not a tty
       ==> default: Configuring cache buckets...
       [SSH] Established
       Vagrant instance <default-ubuntu-1404> created.
       Finished creating <default-ubuntu-1404> (0m8.20s).
-----> Converging <default-ubuntu-1404>...
       Preparing files for transfer
       Preparing dna.json
       Resolving cookbook dependencies with Berkshelf 4.3.2...
       Removing non-cookbook files before transfer
       Preparing validation.pem
       Preparing client.rb
-----> Installing Chef Omnibus (12.9.41)
       Downloading https://www.chef.io/chef/install.sh to file /tmp/install.sh
       Trying wget...
       Download complete.
       ubuntu 14.04 x86_64
       Getting information for chef stable 12.9.41 for ubuntu...
       downloading https://omnitruck-direct.chef.io/stable/chef/metadata?v=12.9.41&p=ubuntu&pv=14.04&m=x86_64
         to file /tmp/install.sh.471/metadata.txt
       trying wget...
       sha1	7fa9fec0ea5d3b6de8bfaed8798a77f3fab2f2a3
       sha256	4ce410534c1d967e5919ac759eb1f31603c063b3a682ac8b7e0ca08657d356a9
       url	https://packages.chef.io/stable/ubuntu/14.04/chef_12.9.41-1_amd64.deb
       version	12.9.41
       downloaded metadata file looks valid...
       /tmp/vagrant-cache/vagrant_omnibus/chef_12.9.41-1_amd64.deb already exists, verifiying checksum...
       Comparing checksum with sha256sum...
       checksum compare succeeded, using existing file!
       Installing chef 12.9.41
       installing with dpkg...
       Selecting previously unselected package chef.
(Reading database ... 16007 files and directories currently installed.)
       Preparing to unpack .../chef_12.9.41-1_amd64.deb ...
       Unpacking chef (12.9.41-1) ...
       Setting up chef (12.9.41-1) ...
       Thank you for installing Chef!
       Transferring files to <default-ubuntu-1404>
       Starting Chef Client, version 12.9.41
       resolving cookbooks for run list: ["myapp::default"]
       Synchronizing cookbooks
       .done.
       Compiling cookbooks
       done.
       Converging 3 resources
       UU.U
       System converged.
       
       resources updated this run:
       * apt_package[apache2]
         - install version 2.4.7-1ubuntu4.9 of package apache2
         - start service service[apache2]
       
       * service[apache2]
         - install version 2.4.7-1ubuntu4.9 of package apache2
         - start service service[apache2]
       
       * template[/var/www/html/index.html]
         - update content in file /var/www/html/index.html from 538f31 to 522276
       
       chef client finished, 3 resources updated
       Finished converging <default-ubuntu-1404> (0m20.89s).
-----> Setting up <default-ubuntu-1404>...
       Finished setting up <default-ubuntu-1404> (0m0.00s).
-----> Verifying <default-ubuntu-1404>...
       Preparing files for transfer
-----> Installing Busser (busser)
       Successfully installed thor-0.19.0
       Successfully installed busser-0.7.1
       2 gems installed
       Installing Busser plugins: busser-serverspec
       Plugin serverspec installed (version 0.5.9)
-----> Running postinstall for serverspec plugin
       Suite path directory /tmp/verifier/suites does not exist, skipping.
       Transferring files to <default-ubuntu-1404>
-----> Running serverspec test suite
-----> Installing Serverspec..
-----> serverspec installed (version 2.34.0)
       /opt/chef/embedded/bin/ruby -I/tmp/verifier/suites/serverspec -I/tmp/verifier/gems/gems/rspec-support-3.4.1/lib:/tmp/verifier/gems/gems/rspec-core-3.4.4/lib /opt/chef/embedded/bin/rspec --pattern /tmp/verifier/suites/serverspec/\*\*/\*_spec.rb --color --format documentation --default-path /tmp/verifier/suites/serverspec
       /opt/chef/embedded/bin/ruby -I/tmp/verifier/suites/serverspec -I/tmp/verifier/gems/gems/rspec-support-3.3.0/lib:/tmp/verifier/gems/gems/rspec-core-3.3.2/lib /opt/chef/embedded/bin/rspec --pattern /tmp/verifier/suites/serverspec/\*\*/\*_spec.rb --color --format documentation --default-path /tmp/verifier/suites/serverspec
       
       myapp::default
         does something (PENDING: Replace this with meaningful tests)

       Pending: (Failures listed here are expected and do not affect your suite's status)

         1) myapp::default does something
            # Replace this with meaningful tests
            # /tmp/verifier/suites/serverspec/default_spec.rb:6


       Finished in 0.00089 seconds (files took 0.35188 seconds to load)
       1 example, 0 failures, 1 pending

       Finished verifying <default-ubuntu-1404> (0m18.83s).
-----> Destroying <default-ubuntu-1404>...
       ==> default: Stopping container...
       ==> default: Deleting the container...
       Vagrant instance <default-ubuntu-1404> destroyed.
       Finished destroying <default-ubuntu-1404> (0m4.98s).
       Finished testing <default-ubuntu-1404> (0m54.00s).
-----> Kitchen is finished. (0m49.66s)
```

As you can see, this already:

 * spun up a fresh ubuntu 14.04 docker container (via Vagrant)
 * downloaded and installed chef-client 12.9.41
 * started a chef run and converged the system
 * installed serverspec and ran the tests

### Interacting with Test-Kitchen

Before we dig deeper into TDD'ing, we should learn some basic commands for
interacting and debugging with test-kitchen first:

* `kitchen list` - show instances(= platform + suite) and their status (running, converged, verified, etc)
* `kitchen destroy` - destroy instances
* `kitchen converge` - converge instances, i.e. apply the chef recipes
* `kitchen verify` - run tests against a converged instance, e.g. using serverspec
* `kitchen test` - shortcut for destroy + converge + verify
* `kitchen login` - login to an instance via SSH
* `kitchen diagnose` - show debugging / diagnostic information

*Note:* when you have multiple platforms / suites to test against defined in your
`.kitchen.yml` you can pass the name of the instance as the second parameter
(e.g. `kitchen verify <instance>`). If you don't pass the name the command will
be executed for *all* instances in the `.kitchen.yml` config.

The first thing we probably want to see now is that the apache web server is
actually running and serves the default content:
```
$ bundle exec kitchen list
Instance             Driver   Provisioner  Verifier  Transport  Last Action
default-ubuntu-1404  Vagrant  ChefZero     Busser    Ssh        <Not Created>

$ bundle exec kitchen converge
-----> Starting Kitchen (v1.8.0)
-----> Creating <default-ubuntu-1404>...
       Bringing machine 'default' up with 'docker' provider...

(...snip)

       chef client finished, 3 resources updated
       Finished converging <default-ubuntu-1404> (0m17.53s).
-----> Kitchen is finished. (0m26.45s)

$ bundle exec kitchen list
Instance             Driver   Provisioner  Verifier  Transport  Last Action
default-ubuntu-1404  Vagrant  ChefZero     Busser    Ssh        Converged
```

Now we see that the system is converged, but we didn't see it running yet. As
we don't know the IP address of the docker container yet, we could check from the
inside whether apache is running via `kitchen login`:
```
$ bundle exec kitchen login
Last login: Tue Sep 22 23:01:17 2015 from 172.17.42.1
vagrant@default-ubuntu-1404:~$ wget -qO- localhost
<html><body>some stuff!</body></html>
vagrant@default-ubuntu-1404:~$ exit
logout
Connection to 172.17.0.19 closed.
```

...or even "tunnel in" via `kitchen exec`:
```
$ bundle exec kitchen exec -c "wget -qO- localhost"
-----> Execute command on default-ubuntu-1404.
       <html><body>some stuff!</body></html>
```

### Networking: Accessing Test-Kitchen VMs from the Outside

Actually, you might have spotted the IP address of the docker container in the
log when the container was brought up. If not, we can still find out via `kitchen diagnose`:
```
$ bundle exec kitchen diagnose | grep hostname
      hostname: 172.17.0.19
      vm_hostname: default-ubuntu-1404
```

So opening the browser at http://172.17.0.19 in this case should serve you
"some stuff!" already :-)

If you want it more predictable, you can set up port forwarding via localhost
as well. In that case you need to add that to the [kitchen-vagrant](https://github.com/test-kitchen/kitchen-vagrant)
driver configuration in `.kitchen.yml`:
```yaml
...
platforms:
  - name: ubuntu-14.04
    driver_config:
      box: tknerr/baseimage-ubuntu-14.04
      network:
        - ["forwarded_port", {guest: 80, host: 8080}]
...
```

After a `kitchen destroy` and `kitchen reload` you should finally be able to see
the same result via http://localhost:8080 too!

*Note:* usually we could also pass a "private_network" here, but that is currently
not supported by the underlying vagrant docker provider.

### Let's add some Tests!

Enough played, let's get back to serious work and add some tests now :-)

 * specs can be added in `test/integration/default/serverspec/default_spec.rb`
 * the serverspec matchers are described here: http://serverspec.org/resource_types.html

First, we probably want to check similar things as we did with ChefSpec before,
but *against a real converged system*. In addition, we also want to check if
the expected content is actually being served:
```ruby
require 'spec_helper'

describe 'myapp::default' do
  it 'installs apache2' do
    expect(package('apache2')).to be_installed
  end
  it 'starts the apache2 service' do
    expect(service('apache2')).to be_running
  end
  it 'enables the apache2 service at startup' do
    expect(service('apache2')).to be_enabled
  end

  context 'when no attributes are set' do
    it 'serves just some stuff' do
      expect(command('wget -qO- localhost').stdout).to match('some stuff!')
    end
  end
end
```

Since the docker container is still running from the previous step, we only
want to trigger the verification:
```
$ bundle exec kitchen verify
-----> Starting Kitchen (v1.8.0)
-----> Verifying <default-ubuntu-1404>...
       Preparing files for transfer
-----> Busser installation detected (busser)
       Installing Busser plugins: busser-serverspec
       Plugin serverspec already installed
       Removing /tmp/verifier/suites/serverspec
       Transferring files to <default-ubuntu-1404>
-----> Running serverspec test suite
       /opt/chef/embedded/bin/ruby -I/tmp/verifier/suites/serverspec -I/tmp/verifier/gems/gems/rspec-support-3.3.0/lib:/tmp/verifier/gems/gems/rspec-core-3.3.2/lib /opt/chef/embedded/bin/rspec --pattern /tmp/verifier/suites/serverspec/\*\*/\*_spec.rb --color --format documentation --default-path /tmp/verifier/suites/serverspec

       myapp::default
         installs apache2
         starts the apache2 service
         enables the apache2 service at startup
         when no attributes are set
          serves just some stuff

       Finished in 0.16236 seconds (files took 0.36505 seconds to load)
       4 examples, 0 failures

       Finished verifying <default-ubuntu-1404> (0m4.60s).
-----> Kitchen is finished. (0m5.08s)
```

### Adding more Platforms and Suites

First, lets add another suite to our `.kitchen.yml`:
```yaml
...
suites:
  - name: default
    run_list:
      - recipe[myapp::default]

  - name: with-content
    run_list:
      - recipe[myapp::default]
    attributes:
      myapp:
        page_content: omg we have suites!
```

We also have to add specific tests for that suite in `test/integration/with-content/serverspec/default_spec.rb`
(note the suite name "with-content" is encoded in that path):
```ruby
require 'spec_helper'

describe 'myapp::default' do
  context 'when the myapp/page_content attribute is set' do
    it 'serves the specified content' do
      expect(command('wget -qO- localhost').stdout).to match('omg we have suites!')
    end
  end
end
```

If we want to run only that suite now, we can do so by specifying a regex that
matches "with-content-ubuntu-1404":
```
$ bundle exec kitchen verify content
-----> Starting Kitchen (v1.8.0)
-----> Creating <with-content-ubuntu-1404>...
       Bringing machine 'default' up with 'docker' provider...

(...snip)

-----> Running serverspec test suite
       /opt/chef/embedded/bin/ruby -I/tmp/verifier/suites/serverspec -I/tmp/verifier/gems/gems/rspec-support-3.3.0/lib:/tmp/verifier/gems/gems/rspec-core-3.3.2/lib /opt/chef/embedded/bin/rspec --pattern /tmp/verifier/suites/serverspec/\*\*/\*_spec.rb --color --format documentation --default-path /tmp/verifier/suites/serverspec

       myapp::default
         when the myapp/page_content attribute is set
           serves the specified content

       Finished in 0.11231 seconds (files took 0.36913 seconds to load)
       1 example, 0 failures

       Finished verifying <with-content-ubuntu-1404> (0m18.92s).
-----> Kitchen is finished. (0m54.68s)
```

You can see that it only ran the one specific test we defined for that suite,
as I did not want to duplicate the "basic tests" from the default suite here.
These should be extracted into a separate file so they can be included from
all test suites (and that is left as an exercise for the reader ;-))

Finally, we could even add another platform to our `.kitchen.yml`:
```yaml
...
platforms:
  - name: ubuntu-12.04
    driver_config:
      box: tknerr/baseimage-ubuntu-12.04
  - name: ubuntu-14.04
    driver_config:
      box: tknerr/baseimage-ubuntu-14.04
...
```

That now gives us 4 combinations in total:
```
$ bundle exec kitchen list
Instance                  Driver   Provisioner  Verifier  Transport  Last Action
default-ubuntu-1204       Vagrant  ChefZero     Busser    Ssh        <Not Created>
default-ubuntu-1404       Vagrant  ChefZero     Busser    Ssh        <Not Created>
with-content-ubuntu-1204  Vagrant  ChefZero     Busser    Ssh        <Not Created>
with-content-ubuntu-1404  Vagrant  ChefZero     Busser    Ssh        <Not Created>
```

You can also run them in parallel using the `--concurrency` flag:
```
$ bundle exec kitchen test --concurrency=4
...
```

(Caveat: you will run into trouble with parallel testing when vagrant-cachier
is enabled, unless you set the caching to :machine scope instead of :box scope)
