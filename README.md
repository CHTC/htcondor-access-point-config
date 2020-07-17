# chtc-flocking

Flocking to the CHTC condor pool requires some setup and configuration on both ends, and this document should help walk you through it.

From Mat's docker submit node build:

	# In the future, the condor RPMs will create these
	RUN \
 	install -m 0700 -o root -g root -d /etc/condor/passwords.d && \
 	install -m 0700 -o condor -g condor -d /etc/condor/tokens.d
 	useradd submituser

	copy chtc-flock.conf to /etc/condor/config.d

	generate token

Information that needs to be submitted to CHTC
* hostname and IP address of submit host
* contact email for host admin
* token

Copy token from CHTC in to /etc/condor/tokens.d
