# sensu-fedora-bringup
Little glue bits necessary to bring up sensu server/client services on fedora... hopefully leading to official fedora packages.

## Quick Start
The Sensu project's 5 minute quickstart guide for RHEL/Centos will work on fedora clients.  The Sensu provided binary packages will install and run on Fedora.  If you just want to try sensu out really quickly, follow the 5 minute quick start guide.

## So Why Bother?
If Sensu's rpms work on Fedora why bother?

### Goal 1: Official Fedora Packaging
The provided binary rpms don't conform to the Fedora packaging guidelines, and there is no srpm or spec file, so the provided sensu packaging is a non-starter for inclusion in Fedora/EPEL as official packages.  

#### Why Bother with Fedora Packages?
The sensu packages use an embedded ruby environment as a convience. This potential has some security concerns as some of the gem dependencies (ex: eventmachine) builds against openssl.  There's benefit in being able to have a package of sensu that built against vendor provided openssl to take advantage of vendor provided security updates for system libraries.

### Goal 2:
Be able to make it easy run sensu core dev/feature repository branch as non-priv user, wihtout having to install anything sensu specific as system admin to lower the bar for testing/development scenarios.
