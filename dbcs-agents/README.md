# Installing an OEM Agent on a DBCS Host in OCI - Including Individual PDBs 

## Introduction
You can monitor cloud resources by installing OEM agents on cloud hosts such as an Oracle DB System. This way you can take advantage of the chargeback plug-in feature in OEM and generate cost reports down to the PDB level.

## Objectives
* Install an OEM agent on an Oracle cloud database for monitoring purposes
* Discover targets within OEM console once agent is installed on DB System

### Outline
1. Adding Hostnames & Creating Auth. Keys
2. Configuring the OEM Instance
3. Creating a Named Credential (In OEM Console)
4. Discovering Host Targets (In OEM Console)
5. Open Firewall in DBCS instance to Allow Traffic Through Port 3872

# Walkthrough

## 1. Add Hostnames
Add Hostnames (in both the OEM & DBCS instances) **as root user**

* Run "hostname" to get name of the instance you are in
* Add to /etc/hosts in the other instance
  * DBCS hostname in OEM instance and vice versa

## 1.1 Create Auth. Keys

Create a new ssh key **as oracle user**

* ssh into DBCS Instance
* sudo su - oracle
* ssh-keygen -t rsa -b 2048
* "/home/oracle/.ssh/chooseyourkeyname"
* choose whether you want a passphrase or not
* navigate to /home/oracle/.ssh
  * cd to .ssh once you are in the oracle folder (you might not see the .ssh folder when you use the ls command)
* vim authorized_keys
  * add the public ssh key that you just created
  * make sure ssh key has oracle user at end
  * copy private key to own computer
  
## 1.2 Create Folder for Agent

**As Oracle user**

* ssh into DBCS Instance
* go to /home/oracle
* Create a new folder for the agent


## 2. Configure your OEM Instance

**As root user**
* Go to etc/ssh/sshd_config and open the file for editing

Make sure the comments in that file appear as they do below
* Uncomment pubkeyauthentication yes 
* Uncomment password authentication yes
* Comment password authentication no

* To lock in sshd_config changes write and quit the sshd_config file
    * run "sudo systemctl restart sshd" (for Linux)

## 3. Create a Named Credential (In the OEM Console)

* Navigate to Setup —> Security —> Named Credentials
* Click Create
* Credential Details
 * Name: yourchoice (I use "oracle")
 * Credential type: SSH Key Credentials
 * Scope: global
 * Username: sysman
 * SSH private key: your key created in DBCS instance

## 4. Discover Host Targets from OEM Console

* Navigate to Setup -> Add Target -> Add Target Manually
* Click Install Agent on Host
  * Add
  * Host text box: Database hostname
  * Platform: Linux x86-64
  * Next

* Installation Base Directory: /home/oracle/newfoldername
* Instance Directory: automatically created for you
* Named Credential: Named credential you creted earlier
* Next
* Deploy Agent


## 5. Open Firewall in DBCS Instance to Allow Traffic Through Port 3872

**As root user**

* iptables -L | grep -i 3872
* iptables-save > /tmp/iptables.orig
* iptables -I INPUT 8 -p tcp -m state --state NEW -m tcp --dport 3872 -j ACCEPT -m comment --comment "Required for EM OMS to talk to the Agent."
 
*Type these commands*
* service iptables status 
  * this applies the new firewall rule
* /sbin/service iptables save

**Change to Oracle user**
* ./emctl status agent

Then if you do the listtargets again, your output should look like:
* [mms:3872, oracle_emd]




## 6. Other 

* When adding target manually
  * /home/oracle



-----------------------------------------------------

*Enable SQL Developer to connect to OMR w/o ssh host*
* opc@emcc ~]$ sudo firewall-cmd --zone=public --add-port=1521/tcp

* [opc@emcc ~]$ sudo firewall-cmd --zone=public --list-ports
7803/tcp 4903/tcp 7301/tcp 9851/tcp 1521/tcp

*Run sqlplus on OMR*
* sudo su - oracle
* vim .bash_profile

*Edit so looks like:*
* cat /home/oracle/README.FIRST
* ORACLE_HOME=/u01/app/database/product
* PATH=$ORACLE_HOME/bin:$PATH
* LD_LIBRARY_PATH=$ORACLE_HOME/lib
* export ORACLE_HOME
* export LD_LIBRARY_PATH
* export PATH
* export TNS_ADMIN=$ORACLE_HOME/network/admin
* export ORACLE_SID=emrep
