# Postgres_Authentification_with_Kerberos

<p align="center">
<img src="https://github.com/khalilsellamii/kerberos-with-postgres/blob/main/kerberos_icon.gif" alt="Alt text" width="300" height="200">
</p>


This project is to demonstrate the establishment and arrangement of a Kerberos authentication system that links a client, a Postgres server, and a Key Distribution Center (KDC). Through this system, the client is able to authenticate securely to the Postgres server by obtaining Kerberos tickets from the KDC.

## Prerequisites

`+` You need to have 3 linux based machines successfully communicating with each other.  
`+` You need to install postgresql on the server and client machine.  
`+` You need to install kerberos packages (krb5-kdc krb5-admin-server krb5-user)

## Kerberos Configuration

### 1. Architecture:


### 2. Working Environment:
First of all, we need to define :  
  `+` Domain name : "kdc.sec.com"  
  `+` Realm : "SEC.COM"  
  `+` Hostnames :   
  
        KDC --> kdc.sec.com  
        Postgres_sever --> postgres.sec.com  
        Client --> client.sec.com  
       
### 3. Domain Name System:
`+` In each machine, we need to change the /etc/hosts file and add the other machines to it

```
sudo nano /etc/hosts

IP address  kdc.sec.com         kdc
IP address  postgres.sec.com    server
IP address  client.sec.com      client
```
> use ctrl+x, ctrl+y, Enter to save the changes

`+` Now, we need to change the hostnames
```
hostnamectl --static set-hostname <HOSTNAME_DESIRED>
```

### 4. Configure the KDC:

`*` Install the kerberos packages
```
sudo apt-get install krb5-kdc krb5-admin-server
```
> when installing, you will be asked to provide some initial configuration like realm name and servers  
> we can later modify and view these configurations with ``` cat /etc/krb5kdc/kdc.conf```

`*` Add the databse in which the principals lately created will be stored
```
sudo krb5_newrealm
```
> You have to provide a password, Don't forget it !!

`*` Create an admin principal , a host principal and generate its keytab
```
$ sudo kadmin.local 
add_principal root/admin
```
`*` Create a principal for the client and a principal for the service server
```
$ sudo kadmin.local
addprinc client
addprinc postgres/postgres.sec.com
```

### 5. Configure Postgres Server

`*` Install the packages of kerberos that allow you to use kerberos's authentification 
```
sudo apt install krb5-user libpam-krb5 libpam-ccreds
```

> MAKE SURE TO ENTER THE SAME CONFIGURATION YOU ENTERED IN THE KDC MACHINE (realm name and servers)

`*` initialize the keytab file
```
$ sudo ktutil
ktutil: addent -password -p <PRINCIPAL> -k <key_Number> -e <encryption_method>
wkt postgres.keytab
```

### Configuration of PostgreSQL

`*` Install the required packages
```
sudo apt-get install postgresql postgresql-contrib
```
Now that the service is installed, let's actually seed our postgres databse,
```
create user khalil with encrypted password 'khalil';  
create database khalil;  
grant all privileges on databse khalil to khalil;  
```

> By default, Postgres Server only allows connections from localhost. Since the client will connect to the Postgres server remotely, we will need to modify postgresql.conf so that Postgres Server allows connection from all the reached network and also specify the keytab file
```
listen_address = '*'
krb_server_keyfile = '/home/postgres/postgres.keytab'
``` 



























