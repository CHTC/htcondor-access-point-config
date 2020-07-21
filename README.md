# chtc-flocking

Flocking to the CHTC condor pool requires some setup and configuration on both ends, and this document should help walk you through it.

Information that needs to be submitted to CHTC
* hostname and IP address of submit host
* contact email for host admin

install development condor rpm
	https://research.cs.wisc.edu/htcondor/instructions/

Copy token from CHTC to 
	/etc/condor/tokens.d

place chtc-flock.conf in 
	/etc/condor/config.d

start condor service or run condor_reconfig
