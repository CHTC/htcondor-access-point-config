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

The first step is to install the HTCondor Software Suite (HTCSS). See installation instructions [here](https://htcondor.readthedocs.io/en/latest/getting-htcondor/admin-quick-start.html), if it has not already been installed. 
It is recommended to configure HTCondor on your EC2 instance as a submit node:

```
curl -fsSL https://get.htcondor.org | sudo GET_HTCONDOR_PASSWORD="dummy-password" /bin/bash -s -- --no-dry-run --submit cm.chtc.wisc.edu
```

`get.htcondor.org` configures HTCondor to use password auth by default. Flocking to CHTC requires IDToken auth, so the pool password created in `/etc/condor/passwords.d/` is unused.
Remove the created password after install:

```
sudo rm /etc/condor/passwords.d/POOL
```

After installation confirm that the following directories exist (the RPM/DEB should create these):
* `/etc/condor/tokens.d` (`chmod 700` and owned by the user account that condor is running as)
* `/etc/condor/config.d`

**Note:** If you are running your AP as an AWS EC2 instance, see section [AWS Prerequisites](#aws-prerequisites) before proceeding.

## Clone Configuration

The second step is to clone the "HTCondor Access Point" configuration from github.  You should `git clone` this repository into `/usr/local/etc/condor`, or some other appropriate directory **other than** `/etc/condor`:

**Note:** EL-based users may need to install `git` first:
```
sudo yum install -y git
```

Clone the configuration:
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

**Note:** AWS users must also set the `TCP_FORWARDING_HOST` to the Elastic IP of their access point.
```
# The AP must be made aware of its own Elastic IP. This IP is advertised
to the CHTC CM and then used by EPs to advertise back to the AP.
TCP_FORWARDING_HOST = <ELASTIC_IP>
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

Generally the `condor_token_request` method is preferred (the tool is included in the HTCSS package [installed previously](#install-the-htcondor-software-suite)).

A `condor_token_request` will timeout, so coordinate with a CHTC administrator before requesting one so that they will be ready to approve it in real time. An appropriate invocation might be:
```
sudo env _CONDOR_TOOL.SEC_CLIENT_AUTHENTICATION_METHODS=SSL condor_token_request -pool cm.chtc.wisc.edu -type collector -identity SCHEDD_$(hostname -f)@cm.chtc.wisc.edu -authz ADVERTISE_MASTER -authz ADVERTISE_SCHEDD -authz NEGOTIATOR -authz DAEMON -authz READ -token SCHEDD_$(hostname -f)@cm.chtc.wisc.edu
```

**Note:** AWS users should use their Elastic IP address instead of `$(hostname -f)`:
```
sudo env ELASTIC_IP="<ELASTIC_IP>" _CONDOR_TOOL.SEC_CLIENT_AUTHENTICATION_METHODS=SSL condor_token_request -pool cm.chtc.wisc.edu -type collector -identity SCHEDD_$ELASTIC_IP@cm.chtc.wisc.edu -authz ADVERTISE_MASTER -authz ADVERTISE_SCHEDD -authz NEGOTIATOR -authz DAEMON -authz READ -token SCHEDD_$ELASTIC_IP@cm.chtc.wisc.edu
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

To determine if the CHTC collector knows about your access point:
```
condor_status -pool cm.chtc.wisc.edu -schedd
```
Your access point's fully qualified domain name should show up in the list.  If so, the CHTC central manager will attempt to find matches for your access point's jobs.

## Firewall Configuration
Although your access point can now communicate with CHTC resources, CHTC resources may not be able to communicate with your access point.  This will lead to failures when the CHTC execution points reach out to your access point for file transfers.  Your access point will need to allow connections for destination port TCP 9618 coming from the following (CHTC) source IP address ranges:
* 128.104.55.0/24
* 128.104.58.0/23
* 128.104.100.0/22 
* 128.105.68.0/23
* 128.105.76.0/24
* 128.105.82.0/24 
* 128.105.244.0/23

And, if your access point has an IPv6 address, from the following IPv6 ranges:
* 2607:f388:107c:0501::/64 
* 2607:f388:1086::/64
* 2607:f388:2200:00c0::/64 
* 2607:f388:2200:0100::/60

## Troubleshooting

### condor_status permissions issues

The CHTC CM is configured to require IDTOKEN authentication for all incoming commands outside of CHTC's internal network.  If you see errors like the following when running `condor_status` against the CHTC CM, then your IDTOKEN is not being used correctly:
```
Error: communication error
SECMAN:2010:Received "DENIED" from server for user unauthenticated@unmapped using no authentication method, which may imply host-based security.  
Our address was '127.0.0.1', and server's address was '192.168.0.1'.  Check your ALLOW settings and IP protocols.
```

HTCondor CLI tools can be configured to use IDToken auth as follows (note that `sudo` is required to read the IDTOKEN from `/etc/condor/tokens.d`):
```
sudo env _CONDOR_TOOL.SEC_CLIENT_AUTHENTICATION=REQUIRED condor_status
```

## AWS Prerequisites

Running a CHTC access point on AWS EC2 requires additional config steps in the AWS console to ensure that the AP can be reached reliably by CHTC execution points.

### Elastic IP

By default, AWS EC2 instances get assigned a new public IP address each time they are started. This is not compatible with the CHTC pool, which requires a static IP address for each Access Point.  
You will need to configure an Elastic IP for your Access Point.

1. Navigate to the EC2 console, and select "Elastic IPs" from the left-hand menu.

1. Create a new Elastic IP, and associate it with your access point's EC2 instance.

1. Note the provisioned Elastic IP address. You must provide it to the Access Point in subsequent steps.
   1. Where required, the Elastic IP will be noted as `<ELASTIC_IP>`

### Security Group

Your access point's security group must allow incoming TCP traffic on port 9618 so that CHTC execution points can reach it (see [Firewall Configuration](#firewall-configuration) for the specific CHTC source IP ranges to allow).

1. Navigate to the EC2 console, and select "Instances" from the left-hand menu.

1. Select your access point's EC2 instance, then open the "Security" tab in the details pane.

1. Click the security group listed under "Security groups" to open it.

1. On the security group's page, select the "Inbound rules" tab, then click "Edit inbound rules".

1. Click "Add rule". For "Type", select "Custom TCP". For "Port range", enter `9618`.

1. For "Source", select "Custom", and enter the CHTC source IP ranges (in CIDR notation) that should be allowed to connect. Add one rule per CIDR range, clicking "Add rule" again for each additional range.

1. Click "Save rules".
