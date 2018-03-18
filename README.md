# sensu-fedora-bringup
Little glue bits necessary to bring up sensu server/client services on fedora... hopefully leading to official Fedora/EPEL packages.

## Quick Start
The Sensu project's 5 minute quickstart guide for RHEL/Centos will work on fedora clients.  The Sensu provided binary packages will install and run on Fedora.  If you just want to try sensu out really quickly, follow the 5 minute quick start guide.

### Goal 1: Official Fedora Packaging
The provided binary rpms don't conform to the Fedora packaging guidelines, and there is no srpm or spec file, so the provided sensu packaging is a non-starter for inclusion in Fedora/EPEL as official packages and difficult to contribute a fix to.  

#### Why Bother with Fedora Packages?
If Sensu's RHEL/Centos rpms work on Fedora why bother?  The provided sensu core 1.x packages use an embedded ruby environment as a convience. This potentially has some security concerns as some of the gem dependencies bundled in the embedded ruby environment build and run against openssl (and other libraries) installed as part of the package (look in /opt/sensu/embedded/lib/).  There is an arguable benefit in being able to have a package of sensu that is built against system libraries (maintained by the packaging system) to take advantage of vendor provided security updates for system libraries.  We can get there by building a reusable rpm specfile and associated srpm that can be rebuilt in a clean Fedora environment. 

Additoinally, The existing sensu rpms make some tradeoffs in its rpm preinstall scripts with regard to trying to support multiple vendor targets (check rpm -q --scripts sensu) that result in false positive rpm verify (rpm -V sensu) warnings concerning user/group.  It would be great from an admin perspective to have sensu rpm packages that installed and verified cleanly.  

#### Why not Debian Packages?
This could be done for Debian as well.  I'm just more familiar with Fedora and EPEL, so I'll start here and hopefully I can encourage a DD to work with me in the future to get sensu core packages into Debian repository as well.

### Goal 2: Sensu Core Contributor On Ramp
Let's make it easy for potential contributors of sensu core to install and test from dev/feature repository branch. 

For Fedora specifically, I intend to build the the reference rpm spec file to make it easy to locally build a sensu rpm built from a specific git repo or git commit. With that spec file, you will be able to locally build and install a replacement rpm, reusing the same configs, the same package preinstall scripts and what not as provided in the official Fedora/EPEL package from Goal 1. This greatly lowers the bar to for community contributors looking to help sensu testing by dogfooding of weekly or nightly pre-release builds with their real world workloads by running sensu pre-release in parallel with a production roll out. 

And going further, I'd like to provide a small bit of reusable shell magic to make it easy to use the sensu git repository as a regular user, outside the packaging system, as a way to easily run in-development sensu feature branches so community actively working on an sensu core enhancement, can install and use a sensu core branch without having to disturb an installed sensu release package set.   Things like reusable userspace systemd service units, and drop-in bash stanzas so you can setup a little pocket in your workstation filesystem from which you can run and hack on the sensu repository branch without having to install anything sensu specific systemwide.

## Plan of Attack
### Sensu Core 1.x
Mature and ruby gem based. Primary benefit: Goal 1. sensu core 1.x is a great target for end-user consumable Fedora/EPEL repository packages.

### Sensu Core 2.x
This is the new hotness. Go based and under heavy development.  Primary Benefit: Goal 2.  This will hopefully lower the bar for community to test sensu 2.x including sensu 1.x to sensu 2.x migration paths.  
