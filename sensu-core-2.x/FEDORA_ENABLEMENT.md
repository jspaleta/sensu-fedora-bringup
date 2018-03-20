# Fedora Enablement

Here are some straight forward recipe instructions for Fedora install of Sensu core 2.x from source so that you can run Sensu core 2.x as a normal user for testing and development purposes.

These instructions will also be used to inform rpm package spec files for submission into Fedora/EPEL repositories in the future.

Most of these instructions should apply to any linux distribution using systemd, except for steps involving the Fedora package repository.

---

## Pre-Reqs
### Install the sensu 2.x nightly el/7 yum repository

The RHEL/Centos nightly build should work on any modern Fedora instance as the packaged binaries are statically linked. 
The the repository install shell script provided in the Sensu 2.x install instructions build and install a repository definitions based on your version of fedora, which is the responsible thing to do. Unfortunately the fedora repository definition doesn't actually exist. So I'm going to a little irresponsible and use the el/7 repository instead.  So after you run the repository install script you'll need to edit the repository definition slightly.

Here is what `/etc/yum.repos.d/sensu-nightly.repo` should look like:

```
[sensu_nightly]
name=sensu_nightly
baseurl=https://packagecloud.io/sensu/nightly/el/7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/sensu/nightly/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[sensu_nightly-source]
name=sensu_nightly-source
baseurl=https://packagecloud.io/sensu/nightly/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/sensu/nightly/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```

Once this file is editted you can follow the rest of the Sensu 2.x RHEL/Centos install instructions to get everything installed and configured.  
Most importantly setting up the repository will help you consume the nightly binaries for testing.


## Setup a password for agent user in sensuctl
`sensuctl user change-password agent --interactive`

## Configure agent.yml
Here's my agent.yml that I'm using.  You'll need to replace `<PASSWORD>` with the password you set for agent authentication in the step above.

```
---
##
# agent configuration
##
agent-id: "Fedora"
#organization: "default"
#environment: "default"
subscriptions: "linux"
backend-url:
  - "ws://127.0.0.1:8081"

##
# authentication configuration
##
user: "agent"
password: <PASSWORD> 

##
# other
##
#cache-dir: "/var/cache/sensu/sensu-agent"
#deregister: false
#deregistration-handler: ""
#keepalive-timeout: 120
#keepalive-internal: 20
```

## Install a check
`sudo gem install sensu-plugins-cpu-checks`

`which check-cpu.rb`

`sensuctl check create test-cpu-check --command 'check-cpu.rb -w 75 -c 90' --interval 60 --subscriptions linux`

## Watch backend and agent log activity
`journalctl -f -u sensu-backend`

`journalctl -f -u sensu-agent`

You should see check-requests being issued by the backend and recieved by the agent every minute or so.


## Note:

Sensu 2.x is currently under heavy development as it pushes towards its official .0 release.  Tracking the nightly builds and testing should be interesting.


