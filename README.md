# Ryu integration
Integration of Ryu Controller and Snort Intrusion Prevention System

## Pre-installed Virtual Machines

This repository includes two pre-build VM's with Ubuntu Servers 14.04. With these two machines you are able to test integration of Ryu controller and Snort intrusion prevention system. The application which is run on Ryu VM is still under development and contains plenty of bugs.

Snort VM includes installation of Snort, Snorby (MySQL database), Barnyard2, Pigrelay and Mininet.

You can download Snort machine from [Here](https://www.idrive.com/idrive/sh/sh?k=k1l4u8h9u1) (.rar-file)

Ryu VM includes only installation of Ryu. With application of ryu/ryu/app/snort.py you can demonstrate integration of Ryu controller and Snort.

You can download Ryu machine from [Here](https://www.idrive.com/idrive/sh/sh?k=n8q9v4n7v9) (.rar-file)

## Credentials

Snort machine's main user with root privileges

username: maria  
passwd: ubuntusnort


Snorby database (two users, on Ryu controller use Snorby user!!)

username: root  
passwd: mysqlroot


username: snorby  
passwed: mysqlsnorby

Ryu machine's main user with root privileges

username: ryu  
passwd: ubunturyu


## Installation

Virtual machines are ready to run. You can create test network with Mininet tool installed on Snort machine. 

**NOTE!!** 
If you have trouble with network connections make sure that interfaces names are correct (May have changed during convertion).

Exmaple command which creates simple topology with two OF-switches and two Hosts:

```
sudo mn --topo linear,2 --switch ovsk,protocols=OpenFlow13 --controller remote,ip="Controller IP-address",port=6633
```

You need to modify the pigrelay.py application (/etc/snort/pigrelay/pigrelay.py) with correct IP-address of Ryu-controller. Also modify the IP-addresses used in ryu/app/snort.py application (Snorby database and FTP server, use Snort machines IP-address)

## Test run

Start snort.py applicaion on Ryu VM

```
cd ryu/ryu/app
sudo ryu-manager snort.py
```
Start two instances of Snort in seperated terminals (On Snort machine)

```
sudo snort -i s1-eth2 -A unsock -l /tmp -c /etc/snort/snort.conf
sudo snort -i s1-eth2 -c /etc/snort/snort.conf
```
Start Barnyard2n application with saves Unified2 data from Snort to Snorby database

```
sudo barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort/ -f snort.u2 -w /var/log/snort/barnyard2.waldo
```

Start pigrelay.py which transmits alert messages to Ryu

```
cd /etc/snort/pigrelay
sudo python pigrelay.py
```

Send ICMO ECHO message from Host1 to Host2. 

```
h1 ping h2
```
Snort makes an alert which is transmited to Ryu. Ryu creates new Flow to Switch1 to prevent ICMP-packet next time. Rules which determines how alerts are generated is located on Snort machine in /etc/snort/rules/ryu.rules -folder
