# Postgres_Authentication_with_Kerberos

<p align="center">
<img src="https://github.com/khalilsellamii/kerberos-with-postgres/blob/main/kerberos_icon.gif" alt="Alt text" width="300" height="200">
</p>


This project is to demonstrate the establishment and arrangement of a Kerberos authentication system that links a client, a Postgres server, and a Key Distribution Center (KDC). Through this system, the client is able to authenticate securely to the Postgres server by obtaining Kerberos tickets from the KDC.

## What do we need ?

`+` You need to have at least 2 linux based machines successfully communicating with each other (3 machines for optimal usage).  
NB: All these machines need to be time synchronised as kerberos provide tickets with expiration delay.  
`+` You need to install postgresql on the server and client machine.  
`+` You need to install kerberos packages (krb5-kdc krb5-admin-server krb5-user)

# Introduction

## What is kerberos ?
> Kerberos is a computer network security protocol that authenticates service requests between two or more trusted hosts across an untrusted network

## Why is kerberos authentication secured ?
> Multiple secret keys, third-party authorization, and cryptography make Kerberos a secure verification protocol. Passwords are not sent over the networks, and secret  keys are encrypted, making it difficult for attackers to impersonate users or services.
 
## Architecture of the kerberos protocol:

<img src="https://github.com/khalilsellamii/kerberos-with-postgres/blob/main/kerberos_architecture.png" alt="Alt text" width="300" height="200">

## Authenticating process description
>After the client sends a request for authentication to the Kerberos Authentication Server (AS), the AS responds with a ticket-granting ticket (TGT). The TGT is encrypted with a secret key shared between the AS and the Kerberos Ticket Granting Server (TGS). The client then sends the TGT to the TGS, along with a request for a service ticket for a specific service. The TGS verifies the TGT and issues a service ticket, which is also encrypted with a secret key shared between the TGS and the service being requested. The client then sends the encrypted service ticket to the service, which decrypts it using its secret key to verify the client's identity and grant access to the requested service.

# Kerberos configuration

### 2. Environment:
First of all, we need to define :  
  `+` Domain name : "kdc.sec.com"  
  `+` Realm : "SEC.COM"  
  `+` Hostnames :   
  
        KDC --> kdc.sec.com  
        Postgres_sever --> postgres.sec.com  
        Client --> client.sec.com  
       
### 3. Setting the Domain(hosts):
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
> In this specific step, as I worked with only 2 machines (KDC and Postgres Server in the same machine), I had to add a principal postgres/kdc.sec.com for whom I attribute an entry in the keytab file, otherwise, when installing kerberos packages on the client machine, I will need to specify the kerberos servers as postgres.sec.com not kdc.sec.com. (2 solutions leading to the same result)
> ```
> $ sudo kadmin.local
> kadmin: addprinc postgres/kdc.sec.com@SEC.COM 
> $ sudo ktutil
> ktutil: addent -password -p postgres/kdc.sec.com@SEC.COM -k 1 -e aes256-cts-hmas=c-sha1-96
> ```

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
create database khalil_database;  
grant all privileges on databse khalil to khalil;  
```

> By default, Postgres Server only allows connections from localhost. Since the client will connect to the Postgres server remotely, we will need to modify postgresql.conf so that Postgres Server allows connection from all the reached network and also specify the keytab file
```
listen_address = '*'
krb_server_keyfile = '/home/postgres/postgres.keytab'
``` 
### Client machine Configuration

`*` As always, first install the required packages
```
sudo apt-get install krb5-user libpam-krb5 libpam-ccreds
```
> Same as before, re-eneter the same configuration specified earlier

## User authentication:
`+` Now, we've completed the needed setup, let's try and get that client authenticated !
```
$ psql -d khalil_database -h postgres.sec.com -U khalil 
```

In the client machine, get a ticket from the tgt (ticket granting ticket) service of kerberos
```
kinit
```
You can check the cached credentials with the command ```$ klist ```


By now, you should be able to authenticate your user to the postgresql without providing a user password, meaning, use ```$ kinit``` to get a ticket from the tgt service of kerebros, and then try connecting remotely to the service:
```
psql -d khalil_database -h postgres.sec.com -U khalil
```
You will be directly prompted to the psql command prompt without being asked to supply your password.




































