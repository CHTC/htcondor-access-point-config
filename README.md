# chtc-flocking

Flocking to the CHTC condor pool requires some setup and configuration on both ends, and this document should help walk you through it.

Information that needs to be submitted to CHTC
* hostname and IP address of submit host
* contact email for host admin

Install development condor branch
	https://research.cs.wisc.edu/htcondor/instructions/

Confirm that the following exist (the RPM should create these)
	/etc/condor/tokens.d   (chmod 700 and owned by the user condor is running as)
	/etc/condor/config.d

Copy token from CHTC to 
	/etc/condor/tokens.d
	(token with chmod 700 and owned by the user condor is running as)

Place chtc-flock.conf in 
	/etc/condor/config.d

Start condor service or run condor_reconfig

Check connectivity
	condor_status -pool cm.chtc.wisc.edu
you should see a large list of machines

