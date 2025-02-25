# CHTC HTCondor Access Point Configuration

Flocking to the CHTC condor pool requires some setup and configuration on both ends, and this document should help walk you through it.

Information that needs to be submitted to CHTC:
* hostname and IP address of submit host
* contact email for host admin

Information that needs to be received from CHTC:
* IDTOKEN file
* project name(s)

## Install the HTCondor development branch

See installation instructions [here](https://research.cs.wisc.edu/htcondor/instructions/).

Confirm that the following directories exist (the RPM/DEB should create these):
* `/etc/condor/tokens.d` (`chmod 700` and owned by the user account that condor is running as)
* `/etc/condor/config.d`

## Place the IDTOKEN

Copy the IDTOKEN received from CHTC into `/etc/condor/tokens.d` (`chmod 600` and owned by the user account that condor is running as).

## Clone Configuration
Git clone this repository into `/usr/local/etc/condor`, or some other appropriate directory other than `/etc/condor`:
```
cd /usr/local/etc/
sudo git clone git@github.com:CHTC/htcondor-access-point-config.git ./condor
```

## Modify Local Configuration
Edit `/etc/condor/condor_config.local`, and add the lines below; Edit `LOCAL_CONFIG_DIR` to point to where you cloned the configuration, and substitute  the project name given to you by the CHTC for `DEFAULT_PROJECT_NAME` (replace "`FooLab`"):
```
# We don't append to LOCAL_CONFIG_DIR below because doing so would
# result in HTCondor reading the config files in the existing
# LOCAL_CONFIG_DIR a second time; we overwrite instead
LOCAL_CONFIG_DIR = /usr/local/etc/condor/config.d

# The default project name to give to accounts that don't have a
# mapping in /etc/condor/condor_config.local;  SURROUNDING DOUBLE
# QUOTES ARE IMPORTANT.
DEFAULT_PROJECT_NAME = "FooLab"
```

## Create a Project Map File
Create a file `/etc/condor/chtc_user_to_project_map`, which can be either empty or have entries like so:
```
* ACCOUNT1 PROJECT1 PROJECT2 PROJECT3
* ACCOUNT2 PROJECT1 
* ACCOUNT2 PROJECT2 PROJECT3
```
Where the project names were provided by the CHTC.

## Tell HTCondor to Read Configuration
Start condor service or run `condor_reconfig`.

## Check Connecivity
```
condor_status -pool cm.chtc.wisc.edu
```

you should see a large list of execution points and their slots.
