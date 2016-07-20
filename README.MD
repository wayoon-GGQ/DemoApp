# Overview
---

This is a peer-to-peer permissioned network built on the hyperledger/fabric code from the Linux Foundation's Hyperledger Project.

The Certificate Authority (CA) provides a number of certificate services to users of a blockchain. More specifically, these services relate to user enrollment, transactions invoked on the blockchain, and TLS-secured connections between users or components of the blockchain. 

The read more about CA, please refer to the [hyperledger documentation](http://hyperledger-fabric.readthedocs.io/en/latest/) 


# Prerequisites
---

Make sure to have the following installed/configured on local machine. 
Get [Vagrant (1.7.4 or later)](https://www.vagrantup.com/)
Get [Virtualbox (5.0, not 5.1)](https://www.virtualbox.org/)
Get [Go](https://golang.org/)
Setup [$GOPATH environmental variables](https://github.com/golang/go/wiki/GOPATH)


### Note to Windows users 
If you are running Windows, before running any git clone commands, run the following command.

git config --get core.autocrlf
If core.autocrlf is set to true, you must set it to false by running

git config --global core.autocrlf false
If you continue with core.autocrlf set to true, the vagrant up command will fail with the error ./setup.sh: /bin/bash^M: bad interpreter: No such file or directory



# Set up chaincode
---

First clone chaincode from [Github]() into a folder under $GOPATH/src/github.com

Once this is done, run the following commands
```
$ cd <to your chain-code path>
$ mkdir vendor/github.com/hyperledger
$ cd vendor/github.com/hyperledger
$ git clone https://github.com/hyperledger/fabric.git
```


# Next set up local hyperledger
---

Run the following commands
```
$ cd <gopath>
$ cd src
$ mkdir github.com\hyperledger
$ cd github.com\hyperledger
$ git clone https://github.com/hyperledger/fabric.git
```

Open membersrvc.yaml under hyperledger/fabric/membersrvc
Uncomment lines 115 and 116 so that the following lines are uncommented
```
	vp0: 4 vp0_secret
    vp1: 4 vp1_secret
```

# Set up development environment
---

## Start Vagrant
```
$ cd $GOPATH/src/github.com/hyperledger/fabric/devenv
$ vagrant up
```
Note: vagrant up could take a while (10+ minutes) if it's your first time 


Once Vagrant is up and running in Terminal 1, run the following commands:

```
$ vagrant ssh
```

Once inside vagrant
```
$ cd $GOPATH/src/github.com/hyperledger/fabric
```

## Start up the CA
```
$ make membersrvc && membersrvc
```

## Start up validating peers

Open a new terminal (Terminal 2):
```
$ cd $GOPATH/src/github.com/hyperledger/fabric/devenv
$ vagrant ssh
```

Once inside vagrant
```
$ ip add
```

Your results might look like this:
```
	vagrant@hyperledger-devenv:v0.0.10-dab545d:/opt/gopath/src/github.com/hyperledger/fabric$ ip add
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host
	       valid_lft forever preferred_lft forever
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 08:00:27:b7:5b:a8 brd ff:ff:ff:ff:ff:ff
	    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::a00:27ff:feb7:5ba8/64 scope link
	       valid_lft forever preferred_lft forever
	3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
	    link/ether 02:42:d1:bc:4d:9c brd ff:ff:ff:ff:ff:ff
	    inet 172.17.0.1/16 scope global docker0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::42:d1ff:febc:4d9c/64 scope link
	       valid_lft forever preferred_lft forever
```

See your inet address for eth0, in this case it's 10.0.2.15

Next, make a peer. Run the following commands:
```
$ cd $GOPATH/src/github.com/hyperledger/fabric
$ make peer-image
```

### Start the first peer
```
$ docker run --rm -it -p 10.0.2.15:5000:5000 -p 10.0.2.15:30303:30303 -e CORE_VM_ENDPOINT=http://172.17.0.1:2375 -e CORE_PEER_ID=vp0 -e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_SECURITY_ENABLED=false -e CORE_SECURITY_PRIVACY=false -e CORE_PEER_PKI_ECA_PADDR=172.17.0.1:50051 -e CORE_PEER_PKI_TCA_PADDR=172.17.0.1:50051 -e CORE_PEER_PKI_TLSCA_PADDR=172.17.0.1:50051 -e CORE_SECURITY_ENROLLID=vp0 -e CORE_SECURITY_ENROLLSECRET=vp0_secret hyperledger/fabric-peer peer node start
```

### Start the second peer
Open a new terminal (Terminal 3) and run
```
$ cd $GOPATH/src/github.com/hyperledger/fabric/devenv
$ vagrant ssh
```

Once inside vagrant, run the following commands
```
$ cd $GOPATH/src/github.com/hyperledger/fabric
$ docker run --rm -it -e CORE_VM_ENDPOINT=http://172.17.0.1:2375 -e CORE_PEER_ID=vp1 -e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_SECURITY_ENABLED=false -e CORE_SECURITY_PRIVACY=false -e CORE_PEER_PKI_ECA_PADDR=172.17.0.1:50051 -e CORE_PEER_PKI_TCA_PADDR=172.17.0.1:50051 -e CORE_PEER_PKI_TLSCA_PADDR=172.17.0.1:50051 -e CORE_SECURITY_ENROLLID=vp1 -e CORE_SECURITY_ENROLLSECRET=vp1_secret -e CORE_PEER_DISCOVERY_ROOTNODE=172.17.0.2:30303 hyperledger/fabric-peer peer node start
```



# Test API using Postman
---

Try a post method using the address: localhost:5000/registrar
Click on "body", click on "raw", type in the following JSON
```json
{
    "enrollId": "WebAppAdmin",
    "enrollSecret": "DJY27pEnl16d"
}
```

# Download demo app
---

Open a new terminal (Terminal 4) and clone this demo app from [Github]() into a folder under $GOPATH
```
$ cd <your folder>
$ npm install
$ sudo npm install gulp -g
$ gulp
```


# Interact with the demo
---
Wait until you see "Websocket Up" in the terminal
Go to localhost:3000 to test the app! :+1:

