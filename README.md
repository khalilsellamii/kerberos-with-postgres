# Postgres_Authentification_with_Kerberos

This project is to demonstrate the establishment and arrangement of a Kerberos authentication system that links a client, a Postgres server, and a Key Distribution Center (KDC). Through this system, the client is able to authenticate securely to the Postgres server by obtaining Kerberos tickets from the KDC.

## Prerequisites

`+` You need to have 3 linux based machines successfully communicating with each other.  
`+` You need to install postgresql on the server and client machine.

```
  sudo apt update
  sudo apt-get install Postgresql postgresql-contrib
```
