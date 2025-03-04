# CHTC HTCondor Access Point Configuration

Flocking to the CHTC condor pool requires some setup and configuration on both ends, and this document should help walk you through it.

Information that needs to be submitted to CHTC:
* hostname of access point(s)
* IPv4 address of access point(s), if available
* IPv6 address of access point(s), if available
* contact email for host admin

Information that needs to be received from CHTC:
* One or more project names [(see below)](#create-a-project-map-file)
* An IDTOKEN credential [(see below)](#obtain-and-place-the-idtoken)

## Install the HTCondor Software Suite

The first step is to install the HTCondor Software Suite (HTCSS). See installation instructions [here](https://research.cs.wisc.edu/htcondor/instructions/), if it has not already been installed.  If installing from scratch, it is recommended to use "Developement" branch rather than "Stable".

After installation confirm that the following directories exist (the RPM/DEB should create these):
* `/etc/condor/tokens.d` (`chmod 700` and owned by the user account that condor is running as)
* `/etc/condor/config.d`

## Clone Configuration

The second step is to clone the "HTCondor Access Point" configuration from github.  You should `git clone` this repository into `/usr/local/etc/condor`, or some other appropriate directory **other than** `/etc/condor`:
```
cd /usr/local/etc/
sudo git clone git@github.com:CHTC/htcondor-access-point-config.git ./condor
```

## Modify Local Configuration
The third step is to modify your HTCondor configuration to include the configuration cloned from github in the previous step.

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
The fourth step is to create a "project map file" that will help with identifying what project your jobs belong to.

Create a file `/etc/condor/chtc_user_to_project_map`, which can be either empty or have entries like so:
```
* ACCOUNT1 PROJECT1 PROJECT2 PROJECT3
* ACCOUNT2 PROJECT1 
* ACCOUNT2 PROJECT2 PROJECT3
```
Where the project names were provided by the CHTC.

## Obtain and Place the IDTOKEN
The fifth step is to obtain and place an IDTOKEN for the purpose of authenticating to the CHTC pool.

An IDTOKEN can be obtained in at least two ways, including:
* Using the [condor_token_request](https://htcondor.readthedocs.io/en/latest/man-pages/condor_token_request.html) tool
* Having a CHTC administrator provide one over a **secure** channel

Generally the `condor_token_request` method is preferred (the tool is included in the HTCSS package [installed previously[(#install-the-htcondor-software-suite)).

A `condor_token_request` will timeout, so coordinate with a CHTC administrator before requesting one so that they will be ready to approve it in real time. An appropriate invocation might be:
```
sudo condor_token_request -identity SCHEDD_$(hostname -f)@cm.chtc.wisc.edu -authz ADVERTISE_MASTER -authz ADVERTISE_SCHEDD -authz NEGOTIATOR -authz DAEMON -authz READ -token SCHEDD_$(hostname -f)@cm.chtc.wisc.edu`
```

The `condor_token_request` tool will place the IDTOKEN in the correct place with the correct ownership (`root`) and mode (`0400`).  If a CHTC administrator provided the IDTOKEN to you over a secure channel, then copy the IDTOKEN into the `/etc/condor/tokens.d` directory:
* With ownership (`chown`) that matches the user account that the condor service is running as
* With mode (`chmod`) `0600`

Invoking the [condor_token_list](https://htcondor.readthedocs.io/en/latest/man-pages/condor_token_list.html) tool as a privileged user should give you an idea if the IDTOKEN has been placed correctly.

## Tell HTCondor to Read Configuration
Finally, HTCSS services must read the new configuration.  If this is a new installation you can start services with:
```
sudo systemctl start condor.service
```

If HTCSS services were already running you can reconfigure them with:
```
sudo condor_reconfig
```
(The `sudo` may not be necessary depending on your existing HTCSS configuration).
