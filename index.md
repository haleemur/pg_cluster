# HackDay Project: Setting up PostgreSQL Replication

by Haleemur Ali


# PostgreSQL Replication

The goal is to set up a master - slave replication using the latest and greatest PostgreSQL (version 9.5).

The stretch goal is to also set up a connection pool that does load balancing and automatic failover.

## Good News

PostgreSQL comes built in with replication (since version 9.0). *streaming asynchronous
master - slave replication*

## Bad News

Requires fungling with the configuration. *I wish there was sample replication config file. No, but
there are plenty of blogs with their opinionated and slightly varying configuration options to set 
up postgres replication*


# How does Streaming Replication work?

PostgreSQL uses something called WAL (write ahead log).

Every time a transaction is committed, it is written to
the WAL. If the server crashes, it can be rebuilt by replaying the WAL from scratch, or since the last backup.

## Crazy Idea

What if we ship the logs to another computer running a PostgreSQL server and have it constantly restore
the backup.

__Us__: *'Sup Computer! Can you restore this PostgreSQL server for me. here's the backup I made last week, and here are some logs.*

__Backup Machine__: *There you go, backup restored, need anything else?*

__Us__: *Oh yeah, forgot these last bits WAL records. Be a peach and stack them on, will ya*

... and so on and so on.


# What do we need for replication?

>(At Least) Two machines running the same version of PostgreSQL.

>Make sure the computers can talk to each other (set up firewall, PostgreSQL Authentication, etc.)

>*Some* changes to the configuration files.

>Add a User with `Replication` privileges.

>*Some* work to set up the replica


# Config Changes (on all PostgreSQL servers):

In `pg_hba.conf` (Host Based Authentication)

>Allow the replication user to connect from within the cluster.

In `postgresql.conf`

>Set `listen_addresses` to `*`

>Set `wal_level` to 'hot_standby'

>Set `archive_mode` to `on`

>Set `archive_command` to `'cd .'` *for reasons I havent figured out yet*

>Set `max_wal_senders` to `1` *because in this example there is only 1 replica*

>Set `hot_standby` to `on`


# Finalize Replication

## Restart the Master Server

>*So that the configuration changes can take effect*

## Set up the Replica Server

>Stop the Database

>Delete the PostgreSQL database, *Why? because we'll be replicating another dB*

>Create or Load a backup of the PostgreSQL master database

>Fool the PostgreSQL Replica server into starting the recovery process.

>Start the Database.

*It should be replicating the master at this point*


# Live Demo

note to self: have iterm set up with all remote machine connections.


# Extra Credits - I

Now, with that out of the way! Lets set up a connection pool & load balancer.

Bad news! No officially supported solution. 

Good News! Googling reveals many 3rd party solutions. The two most popular are:

1. [repmgr](http://www.repmgr.org/) to take care of failover + [pgbouncer](https://pgbouncer.github.io/) 
    to do connection pooling of replicas. *To write to the master, connect to it directly*. Apparently, these tools 
    are simple to set up.

2. [Pgpool-II](http://pgpool.net/mediawiki/index.php/Main_Page) to take care of **everything**: *failover, pooling & load balancing*. But
    apparently its finicky to set up, has great Japanese documentation but not a lot in English.


# Extra Credits - II

What path did I chose?

### Pgpool

*Why? Because my Japanese is awesome? No. Because it seemed more reasonable to learn one thing than two,
even if the 1 thing is explained in a foreign language. Also, diagrams are sufficiently comprehensible in Japanese*

Set up Steps:

* Install the Pgpool2 extension on each PostgreSQL server
* Install Pgpool2 on a machine. *It could be one of the PostgreSQL servers, or a separate machine.
* Create a Pgpool2 user to talk to the database cluster
* Create client accounts on Pgpool2 & edit its configuration files `pgpool_hba.conf` and `pgpool.conf`
* Set up automatic failover handling


# Live Demo 2: Pgpool

note to self: start pgpool in debug mode and don't detach the process to show the server load balancing.


# END