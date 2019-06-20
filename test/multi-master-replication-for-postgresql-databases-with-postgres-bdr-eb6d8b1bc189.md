Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m124[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m131[39m, end: [33m139[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m145[39m, end: [33m259[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m201[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m401[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m182[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m164[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m245[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m63[39m }

# Multi-Master Replication for PostgreSQL databases with Postgres-BDR

One of the most challenging thing when scaling your backend to a new server cluster in a new region is replicating the databases. Most of the time, master-slave replication is not enough and you need to find a solution for master-master replication which can do the replicating job smoothly in realtime. And today we will learn how to setup a multi-master replication for PostgreSQL database with Postgres-BDR.

![Multi-master replication for Postgres-BDR](https://cdn-images-1.medium.com/max/2000/1*exc54EVr6jeERtPtvTWhyw.png)*Multi-master replication for Postgres-BDR*
> Bi-Directional Replication for PostgreSQL (Postgres-BDR, or BDR) is the first open source multi-master replication system for PostgreSQL to reach full production status.

## 0. What if I already have a running PostgreSQL database system?

That is a bad news but don’t worry, we can deal with it. In this case we need to dump all the data and restore it to the new PostgreSQL server with Postgres-BDR plugin. For dumping the database, you can take a look at: [https://www.postgresql.org/docs/9.4/static/backup-dump.html](https://www.postgresql.org/docs/9.4/static/backup-dump.html)

If your database is small, this process may be really fast, but if your database is really big, be carefully and choose a good time for doing this to prevent downtime to your application.

## 1. Building PostgreSQL with Postgres-BDR from source

In this example, I assume we are doing replication on 2 server, we must do all steps in part 1 and 2 on two servers

* server 1: 10.0.0.1

* server 2: 10.0.0.2

For replication, you need to install a special PostgreSQL with Postgres-BDR plugin. Although we can install with apt-get or yum or aptitude . I strongly recommend we should build everything from source. We do this to make sure every node we make in the future use the same version with currently thing we make. In this tutorial, I will use postgres version 9.4.12 and bdr version 1.0.2.

Ok, let’s download two files below

    $ cd ~

    $ wget [https://github.com/2ndQuadrant/bdr/archive/bdr-pg/REL9_4_12-1.tar.gz](https://github.com/2ndQuadrant/bdr/archive/bdr-pg/REL9_4_12-1.tar.gz)
    $ tar -xzvf [REL9_4_12-1.tar.gz](https://github.com/2ndQuadrant/bdr/archive/bdr-pg/REL9_4_12-1.tar.gz)

    $ wget [https://github.com/2ndQuadrant/bdr/archive/bdr-plugin/1.0.2.tar.gz](https://github.com/2ndQuadrant/bdr/archive/bdr-plugin/1.0.2.tar.gz)
    $ tar -xzvf [1.0.2.tar.gz](https://github.com/2ndQuadrant/bdr/archive/bdr-plugin/1.0.2.tar.gz)

Now we need to install some dependencies before building:

    $ sudo sh -c 'echo "deb-src [http://apt.postgresql.org/pub/repos/apt/](http://apt.postgresql.org/pub/repos/apt/) $(lsb_release -cs)-pgdg main 9.4" > /etc/apt/sources.list.d/pgdg.list'

    $ sudo apt-get install wget ca-certificates
    $ wget --quiet -O - [https://www.postgresql.org/media/keys/ACCC4CF8.asc](https://www.postgresql.org/media/keys/ACCC4CF8.asc) | sudo apt-key add -
    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt-get build-dep postgresql-9.4

Then we can build everything now:

    $ cd ~/bdr-bdr-pg-REL9_4_12-1
    $ ./configure --prefix=/usr/lib/postgresql/9.4 --enable-debug --with-openssl
    $ make -j4 -s install-world

    $ cd ~/bdr-bdr-plugin-1.0.2
    $ PATH=/usr/lib/postgresql/9.4/bin:"$PATH" ./configure
    $ make -j4 -s all
    $ make -s install

## 2. Setup for replication

First, we will create a new database with bdr turned on.

    $ createuser postgres
    $ mkdir -p /var/lib/postgresql
    $ chown postgres:postgres /var/lib/postgresql
    $ sudo usermod -d /var/lib/postgresql postgres
    $ su -l postgres
    $ export PATH=/usr/lib/postgresql/9.4/bin:$PATH
    $ mkdir ~/9.4-bdr
    $ initdb -D ~/9.4-bdr -A trust

Edit file ~/9.4-bdr/postgresql.conf , find those lines and change the value as below, those lines already availble in the file:

    listen_addresses = '*'
    shared_preload_libraries = 'bdr'
    wal_level = 'logical'
    track_commit_timestamp = on
    max_connections = 100
    max_wal_senders = 10
    max_replication_slots = 10
    max_worker_processes = 10

Edit file ~/9.4-bdr/pg_hba.conf , add those lines for enable communication between two server:

    local   replication     postgres                        trust
    host    replication     postgres        127.0.0.1/32    trust
    host    replication     postgres        ::1/128         trust
    
    host all all 0.0.0.0/0  password
    
    host replication postgres 10.0.0.1/32 trust
    host replication postgres 10.0.0.2/32 trust
    
    host replication bdrsync 10.0.0.1/32 password
    host replication bdrsync 10.0.0.2/32 password

Ok, now we have a new database at ~/9.4-bdr . Let’s repeat all step from part 1 and 2 all two server. Then we need to run the new PostgreSQL server:

    (in terminal of **postgres **user)

    $ export PATH=/usr/lib/postgresql/9.4/bin:$PATH
    $ pg_ctl -l ~/log -D ~/9.4-bdr start
    $ psql -c "CREATE USER bdrsync superuser;"
    $ psql -c "ALTER USER bdrsync WITH PASSWORD '**12345#**';"

Note that 12345# is our password for replication between server, you need to keep this password carefully as hackers may stole your data with some brute force technics (of course we can prevent this by a simple firewall for IPs and port)

### 3.1. Restore dump database (for ongoing database)

Now, if you have a database to restore, you can do it now. If not and you are going to create a fresh database, let’s go to step 3.2.

Note that if you are going to restore the database, only do this on the master server (server 1) as we will replicate it in server 2 later.

### 3.2. Create new database

Let’s create a new user test_user with a new database test_db for demo

    (in terminal of **postgres **user)

    $ createuser *test_user*
    $ createdb -O test_user *test_db*
    $ psql *test_db *-c 'CREATE EXTENSION btree_gist;'
    $ psql *test_db *-c 'CREATE EXTENSION bdr;'

### 3.3. Create a master node in server 1

    (in terminal of **postgres **user)

    psql
    \c *test_db*
    SELECT bdr.bdr_group_create(
        local_node_name := 'node1',
        node_external_dsn := 'host=*10.0.0.1* user=bdrsync dbname=*test_db *password=12345#'
    );

### 3.4. Join master node in server 2

    (in terminal of **postgres **user)

    psql
    \c *test_db*
    SELECT bdr.bdr_group_join(
        local_node_name := 'node2',
        node_external_dsn := 'host=*10.0.0.2* user=bdrsync dbname=*test_db *password=12345#',
        join_using_dsn := 'host=*10.0.0.1* user=bdrsync dbname=*test_db *password=12345#'
    );

Ok now you should see server 2 do the replication process the database in server 1 (if you have a database in server 1), this may take a while before two nodes become sync to each other and do realtime replication. It is based on the network between two server.

## * Some commands which may be useful *

View bdr nodes and connections:

    (in psql terminal of the database)

    select * from bdr.bdr_nodes;
    select * from bdr.bdr_connections;

To drop replication from a node, this will remove the node from replication with other servers

    (in psql terminal of the database)

    select bdr.remove_bdr_from_local_node(true)
