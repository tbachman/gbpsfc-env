#SETUP

This is a demonstration/development environment for show-casing OpenDaylight GroupBasedPolicy (GBP) with ServiceFunctionChaining (SFC)

The initial instalation may take some time, with vagrant and docker image downloads. 

After the first time it is very quick.

1. Set up Vagrant. 
  * Edit env.sh for NUM_NODES. (Keep all other vars the same for this version)
  * Each VM takes approximately 1G RAM, 2GB used HDD (40GB)
  * demo-gbp1: 3 VMs.
  * demo-symmetric-chain: 6 VMs.
  * demo-asymmetric-chain: 6 VMs.
2. From the directory you cloned into:
```
source ./env.sh
vagrant up
```
  * This takes quite some time initially. 

3. Start controller.
  * Currently it is expected that that controller runs on the host, hosting the VMs.
  * Tested using groupbasedpolicy stable/lithium.
  * Start controller and install following features:

```
 feature:install odl-groupbasedpolicy-ofoverlay odl-groupbasedpolicy-ui odl-restconf
```

#Demos:
* demo-gbp1: 
  * 8 docker containers in 2 x EPGs (web, client)
  * contract with ICMP and HTTP
* demo-symmetry:
  * 2 docker containers in 2 x EPGs (web, client)
  * contract with ICMP (ALLOW) and HTTP (CHAIN, where Client request is chained, Web reverse path is reverse path of chain)
* demo-asymmetry:
  * 2 docker containers in 2 x EPGs (web, client)
  * contract with ICMP (ALLOW) and HTTP (CHAIN, where Client request is chained, Web reverse path is ALLOW)

##demo-gbp1

###Setup

VMs:
* gbpsfc1: gbp
* gbpsfc2: gbp
* gbpsfc3: gbp

Containers:
* h35_{x} are in EPG:client
* h36_{x} are in EPG:web

To run, from host folder where Vagrantfile located do:

` ./startdemo.sh demo-gbp1`

After this, `infrastructure_config.py` will be copied from `/demo-gbp1`, and you are ready to start testing.
 
###To test:

SSH to test VM (may take some seconds):
```bash
vagrant ssh gbpsfc1
```

Get root rights:
```bash
sudo -E bash
```

Check docker containers running on your VM:
```bash
docker ps
```

Notice there are containers from two different endpoint groups, "h35" and "h36".
Enter into the shell on one of "h36" (web) container (on `gbpsfc1` it will be `h36_4`, its IP is `10.0.36.4`, 
you will need it later).
*(You need double ENTER after `docker attach`)*
```bash
docker attach h36_4
```

Start a HTTP server:
```bash
python -m SimpleHTTPServer 80
```

Press `Ctrl-P-Q` to return to your root shell on `gbpsfc1`

Enter into one of "h35" (client) container, 
ping the container where HTTP server runs, 
and connect to index page:

*We use eternal loop here to imitate web activity. 
After finishing your test, you might want to stop the loop with `Ctrl-C`*
```
docker attach h35_{x}
ping 10.0.36.4
while true; do curl 10.0.36.4; done
```

You may `ping` and `curl` to the web-server from any test VM.

`Ctrl-P-Q` to leave back to root shell on VM.

Now watch the packets flow:
```
ovs-dpctl dump-flows
```

Leave to main shell:
```bash
exit #leave root shell
exit #close ssh session
```
Repeat `vagrant ssh` etc. for each of gbpsfc2, gbpsfc3.

###After testing

When finished from host folder where Vagrantfile located do:

`./cleandemo.sh`

If you like `vagrant destroy` will remove all VMs.

##demo-symmetric-chain / demo-asymmetric-chain

VMs:
* gbpsfc1: gbp
* gbpsfc2: sff
* gbpsfc3: "sf"
* gbpsfc4: sff
* gbpsfc5: "sf"
* gbpsfc6: gbp

Containers:
* h35_2 is in EPG:client on gbpsfc1
* h36_4 is in EPG:web on gbpsfc6

To run, from host folder where Vagrantfile located do:

` ./startdemo.sh demo-symmetric-chain` | `demo-asymmetric-chain`

For now, go through each POSTMAN entry in the folder for the demo. This will be ported.

### To test:
*(don't) forget double ENTER after `docker attach`*
```bash
vagrant ssh gbpsfc6
sudo -E bash
docker ps
docker attach h36_4
python -m SimpleHTTPServer 80
```

Ctrl-P-Q

```bash
docker attach h35_2
ping 10.0.36.4
while true; do curl 10.0.36.4; done
```

Ctrl-P-Q

`ovs-dpctl dump-flows`

### When finished from host folder where Vagrantfile located do:

`./cleandemo.sh`

If you like `vagrant destroy` will remove all VMs
