---
layout: post
title: "Docker Swarm with TLS authentication"
---
![docker-swarm-tls](/images/posts/docker-swarm-tls.jpg)

This article is part 2 of a series, you can read part 1 here – [Getting Started with Docker Swarm](/blog/2015/getting-started-with-docker-swarm/)

My last post discussed a quick-and-dirty guide to getting started with Docker Swarm. Today I’m covering how to set up TLS authentication between Docker, Swarm, and you.

The big thing to remember is that Docker Swarm requires all of the nodes to be running a Docker daemon bound to a TCP port. Since I was testing things using AWS that meant that things were avialable on the internet and that anyone who knew my IP:Port could send Docker commands to my machine. Not cool.

To get around this we’ll set up TLS/SSL and require all parties involved to use TLS authentication.

**Please note these instructions are still only for getting a dev environment for testing/playing with and definitely aren’t recommended for production.** TLS/SSL is a huge topic, one I’m mostly glossing over for the sake of just ‘making it work’ for this article. You’ve been warned.

Up until about a week ago Docker Swarm required you to use subjectAltName (SAN) IPs in your certificates which made things a bit more of a hassle than normal. Luckily, thanks to a recent update to the Swarm code that’s no longer the case as long as you use hostnames instead of IPs – which is what we’re going to be doing here.

### Hostname Resolution
**Swarm Host**  
Now that we’re using hostnames we need to add them to our /etc/hosts file. Open the /etc/hosts file on your Swarm host and add:  
`xx.xx.xx.xx node1`  
`xx.xx.xx.xx node2`  
Where xx.xx.xx.xx are replaced by the respective IPs of your Docker nodes.

Test connectivity* `ping host1 -c 3`

**Local**  
Add the hostname of your Swarm host to your local /etc/hosts file.  
`xx.xx.xx.xx swarm`  
Where xx.xx.xx.xx is replaced by the IP of your Swarm host.  

Test connectivity* `ping swarm -c 3`

***I’m not covering firewall settings here so I’m assuming that your machines can talk to each other and that ICMP isn’t blocked.**

### Basic Steps
Here’s a quick rundown of what we’re going to be doing:

- Create a minimal openssl.cnf file with required settings
- Create a Certificate Authority keypair
- Use the CA to create a keypair for your Swarm Host
- Use the CA to create a keypair for your Nodes
- Use the CA to create a keypair for your local machine
- Transfer the keys
- Configure Docker Nodes/Swarm/Local to use TLS

### Create an openssl.cnf file

My file is listed below:

```
[ req ]
default_bits = 4096
default_keyfile = privkey.pem
distinguished_name = req_distinguished_name
x509_extensions = v3_ca
default_md = sha1
string_mask = nombstr
req_extensions = v3_req
prompt = no

[req_distinguished_name]
countryName = US
stateOrProvinceName = NY
localityName = Transmetropolitan
organizationalUnitName = Filty Assistants

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth,serverAuth
subjectKeyIdentifier = hash

[ v3_ca ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints = CA:true

[ crl_ext ]
authorityKeyIdentifier=keyid:always
```

### Create a Certificate Authority keypair

1. Create our private CA Key
  `openssl genrsa -out CAkey.pem 2048`
2. Use the private key to sign our certificate
  `openssl req -config openssl.cnf -new -key CAkey.pem -x509 -days 3650 -out ca.pem`

### Use the CA to create a keypair for your Swarm Host

1. Create our private Swarm key
  `openssl genrsa -out swarmKEY.pem 2048`
2. Create our Certificate Signing Request (CSR)
  `openssl req -subj "/CN=swarm" -new -key swarmKEY.pem -out swarm.csr`
  `openssl x509 -req -days 3650 -in swarm.csr -CA ca.pem -CAkey CAkey.pem -CAcreateserial -out swarmCRT.pem -extensions v3_req -extfile openssl.cnf
    openssl rsa -in swarmKEY.pem -out swarmKEY.pem`

### Use the CA to create a keypair for your Nodes

**NODE1**

1. Create your private key
  `openssl genrsa -out node01KEY.pem 2048`
2. Create your Certificate Signing Request (CSR)
  `openssl req -subj "/CN=node1" -new -key node01KEY.pem -out node01.csr`
3. Sign your certificate
  `openssl x509 -req -days 3650 -in node01.csr -CA ca.pem -CAkey CAkey.pem -CAcreateserial -out node01CRT.pem -extensions v3_req -extfile openssl.cnf
    openssl rsa -in node01KEY.pem -out node01KEY.pem`

**NODE2**

1. Create your private key
  `openssl genrsa -out node02KEY.pem 2048`
2. Create your Certificate Signing Request (CSR)
  `openssl req -subj "/CN=node2" -new -key node02KEY.pem -out node02.csr`
3. Sign your certificate
  `openssl x509 -req -days 3650 -in node02.csr -CA ca.pem -CAkey CAkey.pem -CAcreateserial -out node02CRT.pem -extensions v3_req -extfile openssl.cnf
    openssl rsa -in node02KEY.pem -out node02KEY.pem`

### Use the CA to create a keypair for your local machine

1. Create your private key
  `openssl genrsa -out localKEY.pem 2048`
2. Create your CSR (Make sure to change ‘CN=HOSTNAME’ to the hostname of your local machine that will be sending commands to the Swarm host)
  `openssl req -subj "/CN=HOSTNAME" -new -key localKEY.pem -out local.csr`
3. Sign your certificate
  `openssl x509 -req -days 3650 -in local.csr -CA ca.pem -CAkey CAkey.pem -CAcreateserial -out localCRT.pem -extensions v3_req -extfile openssl.cnf
    openssl rsa -in localKEY.pem -out localKEY.pem`

### Transfer the keys

I used SCP to move all of the keys to their respective machines.

**Swarm Host**  
ca.pem, swarmCRT.pem, swarmKEY.pem, all moved to ~/.ssh on the Swarm Host  
`scp /path/to/ca.pem user@xx.xx.xxx.xxx:~/.ssh/ca.pem`  
etc.

**Node1**  
ca.pem, node01CRT.pem, node01KEY.pem, all moved to ~/.ssh on Node1  
`scp /path/to/ca.pem user@xx.xx.xxx.xxx:~/.ssh/ca.pem`  
etc.

**Node2**  
ca.pem, node02CRT.pem, node02KEY.pem, all moved to ~/.ssh on Node2  
`scp /path/to/ca.pem user@xx.xx.xxx.xxx:~/.ssh/ca.pem`  
etc.

**Local**  
ca.pem, localCRT.pem, localKEY.pem, all moved to ~/.docker on my laptop (where I generated everything to begin with)  
`cp /path/to/ca.pem ~/.docker/ca.pem`  
etc.

### Configure Docker Nodes to use TLS

At this point you’ll need to make a few adjustments to make sure your Docker nodes are using TLS authentication.

**Nodes**  
Update your Docker daemon settings to use TLS. I’m using Project Atomic Fedora AMIs so my file is at /etc/sysconfig/docker.  
`# vi /etc/sysconfig/docker`  
Add the following to your options:  
`OPTIONS='--tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/node0xCRT.pem --tlskey=/path/to/node0xKEY.pem -H 0.0.0.0:2376 -H fd://'`

Make sure you point to the correct pathname for each of the certs on your box. The other thing to note is that we’re no longer using port 2375 since Docker uses port 2376 for SSL.

Reload and restart Docker or just restart your machines.

## Putting it all together

**Docker Nodes**  
Make sure your Docker nodes are up and running with TLS authentication.

**Swarm Host**  
Create your swarm (make sure to note the Swarm token):  
`~/gocode/bin/swarm create`

Add your nodes to the swarm – making sure to use the hostname ‘node1′ or ‘node2′ instead of the IP in the –addr variable:  
`~/gocode/bin/swarm join --addr=node1:2376 token://YOUR-SWARM-TOKEN`  
`~/gocode/bin/swarm join --addr=node2:2376 token://YOUR-SWARM-TOKEN`  

Start the Swarm daemon like this:  
`~/gocode/bin/swarm -debug manage --tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/Swarm-cert.pem --tlskey=/path/to/Swarm-key.pem --host=0.0.0.0:2376 token://YOUR-SWARM-TOKEN`

Make sure you point to the correct pathname for each of the certs on your box, also note that we’re using port 2376 here as well. You should probably replace ‘YOUR-SWARM-TOKEN’ with the actual Swarm token you’re using.

**Local**  
Send a command to the Swarm host, making sure you point to the correct pathname for each of the certs on your box.

`docker --tlsverify --tlscacert=/path/to/ca.pem --tlscert=/path/to/local-cert.pem --tlskey=/path/to/local-key.pem -H swarm:2376 run blackfinsecurity/tha-kali`

The above command should launch a container on one of your hosts. Pretty cool.

Now to test it one last time, try to send the command without the TLS stuff and watch it get rejected:
`docker -H swarm:2376 run blackfinsecurity/tha-kali`

Which should give you the following error:

`FATA[0000] Post http://xx.xx.xxx.xxx:2376/v1.16/containers/create: malformed HTTP response "\x15\x03\x01\x00\x02\x02\x16". Are you trying to connect to a TLS-enabled daemon without TLS?`
