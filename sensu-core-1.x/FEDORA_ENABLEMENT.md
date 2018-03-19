# Fedora Enablement

Here are some straight forward recipe instructions for Fedora install of Sensu core 1.x from source so that you can run Sensu core 1.x as a normal user for testing and development purposes.

These instructions will also be used to inform rpm package spec files for submission into Fedora/EPEL repositories in the future.

Most of these instructions should apply to any linux distribution using systemd, except for steps involving the Fedora package repository.

---

## Pre-Reqs

### Install existing Fedora packaged dependancies
`sudo dnf install -y ruby-devel rubygems rubygem-parse-cron rubygem-ffi rubygem-em-worker redis curl jq`

##### Note 1
Once sensu core is packaged into the Fedora, this step will be simplified greatly.

##### Note 2
Gem deps that still need to be packaged as fedora packages.  These will be installed as part of the gem install step below.

1. em-http-request
2. oj
3. amq-protocol
4. amqp
5. eventmachine  -> Fedora has an old versioned packaged
6. sensu-* -> sensu related gems

---  

## Start local Redis service  

`sudo systemctl start redis`

The Fedora provided redis service comes pre-configured for local host only access, and should be fine for the scope of this tutorial. If you plan to test networked clients on any interface except localhost, you may be required to adjust the redis configuration to bind to additional interface addresses or disable protected mode.


##### Note 1
This should be the last step that requires sudo.  Remaining required steps should only require unprivileged user access.

---

## Download Sensu Source

#### Option 1: Release tarball
This guide will be using sensu 1.2.1 tarball release

`wget https://github.com/sensu/sensu/archive/v1.2.1.tar.gz`

`tar xvzf v1.2.1.tar.gz`

#### Option 2: Clone repository
Alternatively you can work directly in a local copy of the sensu source repository.

`git clone https://github.com/sensu/sensu.git`

`cd sensu`  

`git checkout v1.2.1`

---

## Locally build sensu gem
cd into the sensu source directory and build the sensu gem:

`gem build sensu.gemspec`

this will create a sensu-1.X.Y.gem 

---

## Test install locally built gem

`gem install --explain sensu-1.2.1.gem`

This gives you a brief summary of all the gems that might be needed.

---

## Install locally built gem

`gem install sensu-1.X.Y.gem`

This should install the missing gem dependencies into your current users directory, assuming you haven't changed gem environment to install to a different location. Check using:

`ls -1 ~/.gem/ruby/gems`

You should also have a set of sensu binaries available in `~/bin/` Check using:

`ls -1 ~/bin/`

##### Note 1
This tutorial can be extended to use bundler for gem installs to support parallel installs of different sensu releases in cleanly separated project spaces. Bundler is commonly used for rails application deployment as a best practise, but there's nothing rails specific about it.  For now this is left as an exercise for the reader.   

---

## Setup filesystem directory structure
As a convenience, the sensu-fedora-bringup repository includes a pre-configured filesystem directory that you can use when running Sensu from inside your home directory. The provided filesystem stub includes a basic Sensu configuration setup to use local Redis for transport and datastore.

#### Option 1: Copy filesystem stub into user home directory
Copy the provided filesystem stub directory.  This will allow you to run Sensu from source without disturbing system-wide installed Sensu packages.  This is not recommended for production systems, but it makes for a clean way to work with in-development feature branches or test without fear of disrupting system-wide configs maintained by the packaging system.

To make use of the systemd service files provided here without modification you should copy to `$HOME/sensu` using:

`cp -R filesystem $HOME/sensu`


#### Option 2: Reuse system-wide directories
If you have the sensu RHEL/Centos rpm installed already, then the system-wide directories already exist and can be used. Just make sure you add the development/testing user into the sensu group on the system. You'll need to slightly modify the systemd unit files provided here if you still want to run sensu services as user services instead of system services.

---

## Setup systemd user service for Sensu source install
Recent versions of systemd have the ability to run user services. These are a great way to test in-development services without having to reach for administrator access to adjust working systemwide service configuration.

### Copy provided systemd service files into user home directory

`cp systemd/* ~/.config/systemd/user/`

These systemd service files assume $HOME/sensu will hold your local copy  of the filesystem stub directory.


---  

## Run Sensu services as a systemd controlled user services

`systemctl --user daemon-reload`

`systemctl --user start sensu-server`

`systemctl --user status sensu-server`

`systemctl --user start sensu-api`

`systemctl --user status sensu-server`

`systemctl --user start sensu-client`

`systemctl --user status sensu-server`


Log files should now be populating in `$HOME/sensu/var/log/sensu/`

Sensu Api can now be used to inspect clients, events  and so on. 


---

## Multiple sensu-clients using systemd template unit
Using the flexibility of the systemd service mechanism its possible to smoke test multiple sensu client configurations from the same user account on a single development workstation without reaching for system admin access.    

With the provided systemd template `sensu-client@.service`, it will be easier to spin up multiple client services, running in parallel, each using a different SENSU_PREFIX directory and different configuration all from the same workstation account to test multi client patterns and check behavior.

### Bring up a 2nd client via systemd template service
First replicate filesystem stub into `$HOME/test-client` and edit the client name, socket port, and http_socket port in the `client.json` configuration located `$HOME/test-client/etc/sensu/conf.d/client.json`. The second client service will fail to start unless both socket ports are changed.

To see which tcp ports are in use on the system run:
`sudo netstat -t -l -p -n`

The running sensu client should show up as listening on port 3030 and 3031.

Once configured to use open tcp ports, start up the 2nd client service via systemd using the test-client SENSU_PREFIX directory:

`systemctl --user start sensu-client@test-client`

Sensu API should now list two clients
`curl -s http://127.0.0.1:4567/clients | jq`

---

## Install systemd check  

`gem install sensu-plugin-systemd`  

or if the sensu core package is not installed

`sensu-install sensu-plugin-systemd`    

sensu-plugins-systemd should now be installed in the user's gem collection
Check to make sure the plugin script is installed in the user's executable path:

`which check-systemd.rb`


##### Note 1
If the systemwide sensu RHEL/Centos rpm is installed, the systemwide sensu-install may be invoked and will attempt to install the plugin into the embedded environment installed as part of the sensu package.  This will result in an access denied error. If this happens try calling `$HOME/bin/sensu-install` explicitly or use the `gem install` mechanism. 


---

## Test systemd check on cmdline  


`check-systemd.rb -s redis.service,crond.service`

should result in a clean result:  

`CheckSystemd OK: All services are running`

Now disable crond and retest:
 
`sudo systemctl stop crond`

`check-systemd.rb -s redis.service,crond.service,atd.service`

should result in a critical result:  

`CheckSystemd CRITICAL: crond.service - Not Present`  


## Configure systemd check for fedora-hosts subscribers  

Add a new file called `$HOME/sensu/etc/sensu/conf.d/systemd-check.json`

```
{
  "checks": {
    "systemd": {
      "command": "check-systemd.rb -s redis.service,crond.service",
      "subscribers": [
        "fedora-hosts"
      ],
      "interval": 60,
      "handler": "debug"
    }
  }
}
```

With the debug handler, both the client and the server logs should now be logging messages concerning check request and check result.   And if the 2nd client was also configured to subscribe to "fedora-hosts", you should see both clients responding to check requests.  

