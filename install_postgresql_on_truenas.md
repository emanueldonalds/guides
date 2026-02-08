# Installing PostgreSQL on TrueNAS

This guide shows a way to install PostgreSQL in a Jail on TrueNAS using CLI commands.

When I made this guide I was running TrueNAS version `13.0-U6.1`.

## Create dataset

```
zfs create main-storage-pool/postgresql
zfs set atime=off main-storage-pool/postgresql
zfs set compression=lz4 main-storage-pool/postgresql
zfs set recordsize=8K main-storage-pool/postgresql
zfs set primarycache=metadata main-storage-pool/postgresql
```

## Create a jail

```
iocage create --name postgresql --release 13.5-RELEASE
```

## Configure networking

Set the IP you want to use. I'm using 192.168.0.15.

```
iocage set ip4_addr="re0|192.168.0.15/24" postgresql
iocage set defaultrouter=192.168.0.1 postgresql
```

## Start the container and open a console

```
iocage start postgresql
iocage console postgresql
```

## Install PostgreSQL

Update packages:

```
pkg update
```

Find the latest postgresql version:

```
pkg search postgresql
```

Install the server and contrib packages

```
pkg install postgresql18-server-18.1_1
pkg install postgresql18-contrib-18.1_1
```

Create the postgresql dir in the jail:

```
mkdir /mnt/postgresql
chown -R postgres:postgres /mnt/postgresql
```

While in the jail console, get the ID of the postgres user:

```
id -u postgres
```

In my case it's 770.

Exit the console session:

```
exit
```

## Mount the dataset

```
iocage fstab -a postgresql /mnt/main-storage-pool/postgresql /mnt/postgresql nullfs rw 0 0
```

Set ownership to the postgres user:

```
chown -R 770:770 /mnt/main-storage-pool/postgresql
```

## Setup PostgreSQL

Enter jail again:

```
iocage console postgresql
```

Init PostgreSQL:

```
sysrc postgresql_enable=yes
sysrc postgresql_data=/mnt/postgresql
service postgresql initdb
```

Setup listen address (with nano or vim):

```
vim /mnt/postgresql/postgresql.conf
```

Set listen addresses to any IP:

```
listen_addresses = '*'
```

Save and exit.

Allow connections from local network to connect with password auth:

```
vim /mnt/postgresql/pg_hba.conf
```

At the end of the file, add:

```
host    all             all             192.168.0.1/24             md5
```

Start postgresql

```
service postgresql start
```

## Setup user password

Right now the postgres user has no password, let's create one.

Switch to postgres user and run psql:

```
su postgres
psql
```

Set the password:

```sql
ALTER USER postgres WITH PASSWORD 'the_password';
```

Done
