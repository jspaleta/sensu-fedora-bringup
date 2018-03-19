# Sensu-Fedora-Bringup
Little glue bits necessary to bring up Sensu server/client services on Fedora... hopefully leading to official Fedora/EPEL packages.

## Quick Start
If you just want to try Sensu out really quickly, follow the Sensu 1.x [ 5 minute quick start guide for RHEL/Centos] (https://docs.sensu.io/sensu-core/1.2/quick-start/five-minute-install/) .The Sensu provided binary packages for RHEL/Centos 7 should install and run on currently supported Fedora installs.

### Motivation 1: Generate Official Fedora/EPEL Packaging
The provided binary rpms don't conform to the Fedora packaging guidelines, and there is no srpm or spec file, so the provided sensu packaging is a non-starter for inclusion in Fedora/EPEL as official packages.    

#### Why Bother With Official Fedora Packages?
If Sensu's RHEL/Centos rpms work on Fedora why bother?  

The provided Sensu core 1.x packages use an embedded ruby environment as a convenience. This potentially has some security concerns as some of the bundled libraries in the embedded ruby environment (ex: openssl) installed as part of the Sensu provided package (look in /opt/sensu/embedded/lib/).  There is an arguable benefit in being able to have a package of Sensu that is built against system libraries (maintained by the packaging system) to take advantage of vendor provided security updates for system libraries.  We can get there by building a reusable rpm specfile and associated srpm that can be rebuilt in a clean Fedora environment.  An install via ruby gems won't have this particular concern, but you lose some of the benefits of using the rpm packaging manager in non-containerized environment.

Additionally, the existing Sensu rpms make some tradeoffs in its rpm preinstall scripts with regard to trying to support multiple vendor targets (check rpm -q --scripts sensu) that result in false positive rpm verify (rpm -V sensu) warnings concerning user/group changes on the configuration files and directories.  It would be great from an admin perspective to have a set of Sensu rpm packages that installed and verified cleanly.  

#### Why not Debian Packages?
This could be done for Debian as well.  I'm just more familiar with Fedora and EPEL, so I'll start here and hopefully I can encourage a DD to work with me in the future to get sensu core packages into Debian repository as well.

### Motivation 2: Sensu Core Contributor On Ramp
Let's make it easy for potential contributors of Sensu Core to install and test from dev/feature repository branch in a workstation environment. 

As a first step, I'd like to provide a small bit of systemd magic to make it easy to use the Sensu git repository checkout as a regular user, outside the packaging system, as a way to easily run in-development Sensu feature branches and to test multiple client check configurations from a single unpriviledged workstation account.  

Going further, for Fedora development workstations specifically, I intend to build the the reference rpm spec file to make it easy to locally build a Sensu rpm built from a specific git repo or git commit with only minor modification. With that spec file, you will be able to locally rebuild and install a replacement rpm, re-using the same packaging scripting as provided in the official Fedora/EPEL package from Motivation 1. This will hopefully lower the bar for community contributors looking to help Sensu testing by dogfooding of weekly or nightly pre-release builds with their real world workloads by running Sensu pre-release in parallel with a production roll out. 


## Plan of Attack
### [Sensu Core 1.x](/sensu-core-1.x/FEDORA_ENABLEMENT.md)
Ruby gem based and ready for production installs. 

#### Primary benefit: Motivation 1 
Sensu core 1.x is a great target for end-user consumable Fedora/EPEL repository packages.

### Sensu Core 2.x
This is the new hotness. Go based and under heavy development.  

#### Primary Benefit: Motivation 2  
This will hopefully lower the bar for community to test Sensu 2.x including Sensu 1.x to Sensu 2.x migration paths.  
