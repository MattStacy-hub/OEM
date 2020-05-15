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
4. Open Firewall in DBCS instance to Allow Traffic Through Port 3872
5. Discovering Host Targets in the OEM Console

# Walkthrough

## 1. Add Hostnames & Create Auth. Keys
*Add Hostnames (in both the OEM & DBCS instances)*

*Ssh into DBCS Instance*
Create a new ssh key as oracle user

* ssh-keygen -t rsa -b 2048
* "/home/opc/newKey/newKeyName"
* "/home/oracle/.ssh"
* create directory, auth_keys file, add ssh key there
* sudo su - oracle

Add public key to authorized_keys for the oracle user
Save private key to your computer

------------------------------------------------------
Create Auth. Keys

* Add to /etc/hosts
* IP of OEM
 * Run hostname if on OEM instance to get name
<IP address>    emcc.marketplace.com
 
Sudo -su oracle
Create ssh key, add public to authorized keys for oracle user
  Make sure ssh key has oracle user at end
  Copy private key to own computer
Change to root user
Then go to /etc/ssh/sshd_config
  Open that file
  Search PubkeyAuthentication
    Uncomment pubkeyauthentication yes
  Search password authentication
    Uncomment yes line
  To lock in sshd_config changes
    Sudo service sshd restart
Create a new named credential on OEM console
  Paste private key
  Name doesn’t matter
When adding target manually
  /home/oracle

## 2. Configure the OEM Instance

(As root user)
cd to /etc/ssh/sshd_config and open the file for editing
Make sure the comments in that file appear as they do below
 Uncomment pubkeyauthentication yes 
 Uncomment password authentication yes
 Comment out password authentication no
Lock in sshd_config changes
 sudo service sshd restart

## 3. Create a Named Credential (In OEM Console)

Navigate to Setup —> Security —> Named Credentials
Click Create
Credential Details
 Name: oracle
 Credential type: SSH Key Credentials
 Scope: global
 Username: sysman
 SSH private key: your key created in DBCS instance
Name doesn’t matter


## 4. Open Firewall in DBCS Instance to Allow Traffic Through Port 3872

*As root*
 iptables -L | grep -i 3872
 iptables-save > /tmp/iptables.orig
 iptables -I INPUT 8 -p tcp -m state --state NEW -m tcp --dport 3872 -j ACCEPT -m comment - -comment “REQUIRED for EM OMS to talk to the agent.” 
 
*Type command*
 Service iptables status (This applies the new firewall rule)
 
 "/sbin/service iptables save"
*Change to oracle user*
./emctl status agent
then if you do the listtargets again, your output should look like:
[mms:3872, oracle_emd]

## 5. Discover Host Targets from OEM Console

*Enable SQL Developer to connect to OMR w/o ssh host*
opc@emcc ~]$ sudo firewall-cmd --zone=public --add-port=1521/tcp
Success
[opc@emcc ~]$ sudo firewall-cmd --zone=public --list-ports
7803/tcp 4903/tcp 7301/tcp 9851/tcp 1521/tcp

*Run sqlplus on OMR*
sudo su - oracle
vim .bash_profile

*Edit so looks like:*
cat /home/oracle/README.FIRST
ORACLE_HOME=/u01/app/database/product
PATH=$ORACLE_HOME/bin:$PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib
export ORACLE_HOME
export LD_LIBRARY_PATH
export PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin
export ORACLE_SID=emrep







