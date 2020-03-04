---
title: "Using Ansible to Setup a Security Audit Infrastructure"
published: true
---

# Using Ansible to Setup a Security Audit Infrastructure

Last time I wrote about setting up OpenVAS in a Master/Slave architecture, but I
left all the work to the reader to complete. You can find the original article
[here](https://oparkins.github.io/2019/11/22/OpenVAS-Master-Slave/). 

Recently, I had to create an Ansible script to setup the architecture
automatically for security audits. The scripts can be found on my GitHub at
https://github.com/oparkins/ignition-key.security-audit

But the scripts don't only setup OpenVAS automatically, they also setup a SSH
Certificate system. SSH Certificates are beautiful and I wish more people used
them. They allow for auto-expiring credentials and more awesome features. 

In this post, I will describe the reasoning because the way we setup our
security audit infrastructure. A lot of the setup information is available to in
repository README. If there is any confusion or a section that needs to be
clarified/elaborated, file an issue and I will get around to it.


## Purpose for the Security Audit Infrastructure

I needed to create this infrastructure for New Mexico Institute of Mining and
Technology's CyberCorp Scholarship for Service group. As part of our
professional development course, we perform security audits around the State of
New Mexico. The purpose of these audits are twofold: have students gain real
world experience and make a positive impact in our community by helping securing
organizations. 

## Cool Features

An overview of the really cool features (in my opinion) are:
- Automatic setup of OpenVAS Master/Slave
	- The slaves are automatically added to the master
- SSH Certificate Setup
	- Sets up a CA and everything for you
	- Allows one file to grant access to server and remote machines with accountability
- All done in Ansible


## Basic System Layout

We created our infrastructure so that every remote machine calls back to the
Command and Control (C2) server. We assumed that our server would not be able to
connect directly to the laptops, but that any organization that we perform an
audit for does allow outbound connections. So the remote machines, often times
laptops for us, will connect to the C2 server and setup multiple tunnels for
communication back to their ssh servers and OpenVAS connections.

This also allows for the remote machines to only accept connections from their
local machine which protects them. We assume that the remote machines are
compromised because malicious actors could have physical access to them. So we
design the system with that assumption to protect ourselves and our clients.

The remote machines will call back to the server with an account that doesn't
allow for the user to login. We setup the audit user to have a login shell of
`/bin/false`. Our ssh connection to the server uses the `-N` flag which the man
pages says:

```
-N      Do not execute a remote command.  This is useful for just forwarding ports.
```

This means that if a malicious actor logged onto our machines (which lets be
honest, is trivial due to the lack of encryption), they could not access the
server. Yes, the could try to run the server out of resources by opening a ton
of ports, but our data would be out of their reach.

## Why Ubuntu?

I chose Ubuntu because of the packages available on the system. CentOS is great
for servers because of SELinux and the fact that it has primarily security
updates, but it was not appropriate in this case. I did not want to struggle
with installing many of the required packages manually. 

The major package that causes problems is OpenVAS. It seems that every
distribution has different assumptions with OpenVAS that get in the way of a
manual installation. These problems could be caused by myself while I was
learning the OpenVAS system architecture, but I have seen problems with OpenVAS
occur on Kali Linux even. Since the Ansible scripts setup a Master/Slave
architecture, they have to mess with OpenVAS to a certain extent. I have found
that Ubuntu provides me the best ability to change settings without destroying
everything that package maintainers worked hard to create.

## Why SSH Certs?

I chose to use SSH Certificates because they provided the best way to allow team
members to access all the machine they need while keeping accountability. SSH
Certificates allow all the team members to sign in to the same audit account,
but for each individual team member name to be logged in the server/remote
machine logs. It also allows the system administrator the ability to log in as
root on any machine in the infrastructure. This is very appealing because the
system administrator (me in this case), has a shortcut to fix all the machines
with a single file. In addition, since I have access to the CA, I can create
another SSH Cert that gives me access to the same resources as the team members
to ensure that the changes worked.

Another benefit of the certificates is the expiration property that I can set.
Typically our audits last a school semester, so I can set the certificate to be
valid for 16 weeks and not worry about it.

The eventual goal would be to use a single CA for all of the SFS infrastructure
so that students can access any server that I setup. Of course, when a new audit
is created, I would send out a new certificate file for each user, but that
reduces the burden of connecting to new services on students.

## Conclusion

I have very proud of the Ansible scripts that have been created. It was a
combination of OpenVAS, SSH Certificates, and more that made it worth while. If
there are any questions or concerns, start a GitHub issue. Feel free to add
other important tools to the remote machine configurations and create a pull
request. My hope is that other poeple can modify and use the infrastructure.