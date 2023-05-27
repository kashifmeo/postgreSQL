*************************************************************************************************
************Set Up a Highly Available PostgreSQL Release 15 Cluster on Oracle Linux 8************
*************************************************************************************************
Virtual box setting. Should have two network adaptors. 1st for host on for host connections. 2nd NAT for internet.
cpu = 4 core
RAM = 4 GB
Disk = 50
Patroni high availability cluster is comprised of the following components:

PostgreSQL database is a powerful, open source object-relational database system with over 35 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance.

Patroni is a cluster manager used to customize and automate deployment and maintenance of PostgreSQL HA (High Availability) clusters.

PgBouncer is an open-source, lightweight, single-binary connection pooler for PostgreSQL. PgBouncer maintains a pool of connections for each unique user, database pair. It’s typically configured to hand out one of these connections to a new incoming client connection, and return it back in to the pool when the client disconnects.

etcd is a strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or cluster of machines. We will use etcd to store the state of the PostgreSQL cluster.

HAProxy is a free and open source software that provides a high availability load balancer and reverse proxy for TCP and HTTP-based applications that spreads requests across multiple servers.

Keepalived implements a set of health checkers to dynamically and adaptively maintain and manage load balanced server pools according to their health. When designing load balanced topologies, it is important to account for the availability of the load balancer itself as well as the real servers behind it. 

All of these patroni cluster components need to be installed on Linux servers. Patroni software shares the server with the PostgreSQL database.

Patroni high availability cluster is comprised of the following components:
Basic environment for patroni cluster
For the demonstration purpose, we will start with the basic environment to set up a 3-node patroni cluster on three separate virtual machines:
will be installed on all nodes (Patroni, PostgreSQL, PgBouncer, Etcd, HAProxy, Keepalived)
HOSTNAME	IP ADDRESS	SERVICES	
pgsql01	192.168.56.231  (Patroni, PostgreSQL, PgBouncer, Etcd, HAProxy, Keepalived)
pgsql02	192.168.56.232	(Patroni, PostgreSQL, PgBouncer, Etcd, HAProxy, Keepalived)
pgsql03	192.168.56.233	(Patroni, PostgreSQL, PgBouncer, Etcd, HAProxy, Keepalived)
SHARED IP(192.168.56.200)
We will install all the patroni cluster components on these three virtual machines.

Now that you have understood patroni cluster components and its requirements, 
so lets begin with the following steps to build a highly available patroni cluster with postgres database version 15 on Oracle Linux release 8.
Prepare your Linux server to run patroni cluster
Log in to your Linux server using a non-root user with sudo privileges and perform the following steps.
 
Set correct timezone on each node:timedatectl set-timezone Asia/Riyadh
nano /etc/hosts
192.168.56.231 pgsql01.localdomain pgsql01
192.168.56.232 pgsql02.localdomain pgsql02
192.168.56.233 pgsql03.localdomain pgsql03
Save and close the editor when you are finished.
Make sure you repeat the same on each node before proceeding to next.
Disable SELINUX
nano /etc/selinux/config
Change SELINUX=enforcing to SELINUX=disabled
Save and close the editor when you are finished.

Make sure you repeat the same on each node before proceeding to next. When you are finished, reboot each node to make the changes effect:
shutdown -r now (or run "setenforce 0" to avoid restart)

Configure Firewalld
The ports required for operating PostgreSQL HA cluster using (patroni, pgbouncer, etcd, haproxy, keepalived) are the following:

5432 Postgres database standard port.
6432 PgBouncer standard port.
8008 patroni rest api port required by HAProxy to check the nodes status.
2379 etcd client port required by any client including patroni to communicate with etcd cluster.
2380 etcd peer urls port required by the etcd cluster members communication.
5000 HAProxy front-end listening port, required to establish connection to the back-end masater database server via pgbouncer port 6432.
5001 HAProxy front-end listening port, required to establish connection to the back-end replica database servers via pgbouncer port 6432
7000 HAProxy stats dashboard, required to access HAProxy web interface using HTTP.
You can allow these required ports from firewalld using the following command:

sudo firewall-cmd --zone=public --add-port=5432/tcp --permanent
sudo firewall-cmd --zone=public --add-port=6432/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8008/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2379/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2380/tcp --permanent
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --zone=public --add-port=5000/tcp --permanent
sudo firewall-cmd --zone=public --add-port=5001/tcp --permanent
sudo firewall-cmd --zone=public --add-port=7000/tcp --permanent
sudo firewall-cmd --add-rich-rule='rule protocol value="vrrp" accept' --permanent
sudo firewall-cmd --reload

or disable firewall by "systemctl stop firewalld" then "systemctl disable firewalld"
Make sure you repeat the same on each node before proceeding to next.   
Install Required Repository
Type below command to install EPEL repo on your oracle Linux servers:
 
sudo dnf install -y epel-release

sudo dnf install -y yum-utils
Make sure you repeat the same on each node before proceeding to next.

Install PostgreSQL
Type below command to install PostgreSQL database release 15 on oracle Linux Servers: 
 
sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

sudo dnf config-manager --enable pgdg15

sudo dnf module disable -y postgresql

sudo dnf -y install postgresql15-server postgresql15 postgresql15-devel

sudo ln -s /usr/pgsql-15/bin/* /usr/sbin/

Make sure you repeat the same on each node before proceeding to next.
--vid 01
Install etcd
The etcd package is not available in oracle Linux default repositories. To install etcd using dnf package manager, first you need to create a repo like below:
sudo nano /etc/yum.repos.d/etcd.repo

Add following:
 
[etcd]
name=PostgreSQL common RPMs for RHEL / oracle $releasever - $basearch
baseurl=http://ftp.postgresql.org/pub/repos/yum/common/pgdg-rhel8-extras/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
repo_gpgcheck = 1

Save and close the editor when you are finished.
 
Type following command to install etcd on your oracle Linux servers:
 
sudo dnf makecache

sudo dnf install -y etcd
Make sure you repeat the same on each node before proceeding to next.
 
 Configure etcd Cluster
Edit /etc/etcd/etcd.conf file on your first node (pgsql01) in our case, to make the required changes:

sudo mv /etc/etcd/etcd.conf /etc/etcd/etcd.conf.orig

sudo nano /etc/etcd/etcd.conf
Add following configuration: 

ETCD_NAME=pgsql01
ETCD_DATA_DIR="/var/lib/etcd/pgsql01"
ETCD_LISTEN_PEER_URLS="http://192.168.56.231:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.231:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.231:2380"
ETCD_INITIAL_CLUSTER="pgsql01=http://192.168.56.231:2380,pgsql02=http://192.168.56.232:2380,pgsql03=http://192.168.56.233:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.231:2379"
ETCD_ENABLE_V2="true"

Save and close the editor when you are finished.

Edit /etc/etcd/etcd.conf file on your second node (pgsql02) in our case, to make the required changes:

sudo mv /etc/etcd/etcd.conf /etc/etcd/etcd.conf.orig

sudo nano /etc/etcd/etcd.conf
Add following configuration:

ETCD_NAME=pgsql02
ETCD_DATA_DIR="/var/lib/etcd/pgsql02"
ETCD_LISTEN_PEER_URLS="http://192.168.56.232:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.232:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.232:2380"
ETCD_INITIAL_CLUSTER="pgsql01=http://192.168.56.231:2380,pgsql02=http://192.168.56.232:2380,pgsql03=http://192.168.56.233:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.232:2379"
ETCD_ENABLE_V2="true"

Save and close the editor when you are finished.
 
Edit /etc/etcd/etcd.conf file on your third node (pgsql03) in our case, to make the required changes:

sudo mv /etc/etcd/etcd.conf /etc/etcd/etcd.conf.orig

sudo nano /etc/etcd/etcd.conf
Add following configuration:

ETCD_NAME=pgsql03
ETCD_DATA_DIR="/var/lib/etcd/pgsql03"
ETCD_LISTEN_PEER_URLS="http://192.168.56.233:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.233:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.233:2380"
ETCD_INITIAL_CLUSTER="pgsql01=http://192.168.56.231:2380,pgsql02=http://192.168.56.232:2380,pgsql03=http://192.168.56.233:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.233:2379"
ETCD_ENABLE_V2="true"

Save and close the editor when you are finished. 
 
Edit .bash_profile on each node:
 
cd ~
nano .bash_profile
Add etcd environment variables at the end of the file:

export PGDATA="/var/lib/pgsql/15/data"
export ETCDCTL_API="3"
export pgsql0_ETCD_URL="http://127.0.0.1:2379"
export pgsql0_SCOPE="pg_cluster"
pgsql01=192.168.56.231
pgsql02=192.168.56.232
pgsql03=192.168.56.233
ENDPOINTS=$pgsql01:2379,$pgsql02:2379,$pgsql03:2379

Save and close the editor when you are finished.

Make sure you repeat the same on each node before proceeding to next.

Start etcd Cluster
Type below command simultaneously on each node (pgsql01, pgsql02, pgsql03) to start etcd cluster:

sudo systemctl start etcd
sudo systemctl status etcd

Verify etcd cluster from any of your nodes using the following command:
 
source ~/.bash_profile

[root@pgsql01 ~]# etcdctl --write-out=table --endpoints=pgsql01:2379 member list
You will see the output similar to like as shown below:

+------------------+---------+---------+----------------------------+----------------------------+------------+
|        ID        | STATUS  |  NAME   |         PEER ADDRS         |        CLIENT ADDRS        | IS LEARNER |
+------------------+---------+---------+----------------------------+----------------------------+------------+
| 73e0d9f48ef262e8 | started | pgsql01 | http://192.168.56.231:2380 | http://192.168.56.231:2379 |      false |
| a2ad653c009b5ada | started | pgsql03 | http://192.168.56.233:2380 | http://192.168.56.233:2379 |      false |
| e41aa538ff796078 | started | pgsql02 | http://192.168.56.232:2380 | http://192.168.56.232:2379 |      false |
+------------------+---------+---------+----------------------------+----------------------------+------------+

Make sure you etcd cluster is up and running as described before proceeding to next step.



Install Patroni
Type below command to install patroni on your first node (pgsql01) in our case:

sudo dnf -y install python3 python3-devel python3-pip gcc libpq-devel

sudo pip3 install --upgrade testresources --upgrade setuptools psycopg2 python-etcd

sudo dnf -y install patroni patroni-etcd watchdog
Make sure you repeat the same on each node before proceeding to next.

 
 Configure Patroni Cluster
By default Patroni configures PostgreSQL database for asynchronous replication using PostgreSQL streaming replication method. Choosing your replication schema is dependent on your production environment.

Synchronous vs Asynchronous replication:
PostgreSQL supports both synchronous and asynchronous replication for high availability and disaster recovery.
Synchronous replication means that a write operation is only considered complete once it has been confirmed as written to the master server as well as one or more synchronous standby servers. This provides the highest level of data durability and consistency, as clients are guaranteed that their writes have been replicated to at least one other server before the write is acknowledged as successful. However, the tradeoff is that the performance of the master server is impacted by the time it takes to confirm the write to the synchronous standby servers, which can be a bottleneck in high-write environments. 
Asynchronous replication, on the other hand, means that a write operation is acknowledged as successful as soon as it is written to the master server, without waiting for any confirmations from standby servers. This provides better performance as the master server is not blocked by the replication process, but the tradeoff is that there is a risk of data loss if the master server fails before the write is replicated to the standby servers.
PostgreSQL allows you to configure one or more synchronous replicas and any number of asynchronous replicas, so you can balance the performance and data durability trade-offs according to your needs.
For more information about async, and sync replications, see the Postgres documentation as well as Patroni documentation to determine which replication solution is best for your production need.

For this guide, we will use default asynchronous replication mode.

To configure patroni cluster, you need to create a patroni.yml file in /etc/patroni location on your first node (pgsql01) in our case:

sudo mkdir -p /etc/patroni

sudo nano /etc/patroni/patroni.yml
Add following configuration:

scope: pg_cluster
namespace: /service/
name: pgsql01

restapi:
    listen: 192.168.56.231:8008
    connect_address: 192.168.56.231:8008

etcd:
    hosts: 192.168.56.231:2379,192.168.56.232:2379,192.168.56.233:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:

  initdb:
  - encoding: UTF8
  - data-checksums

  pg_hba:
  - host replication replicator 127.0.0.1/32 md5
  - host replication replicator 192.168.56.231/32 md5
  - host replication replicator 192.168.56.232/32 md5
  - host replication replicator 192.168.56.233/32 md5
  - host all all 0.0.0.0/0 md5

  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb

postgresql:
  listen: 192.168.56.231:5432
  connect_address: 192.168.56.231:5432
  data_dir: /var/lib/pgsql/15/data
  bin_dir: /usr/pgsql-15/bin
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: replicator
    superuser:
      username: postgres
      password: postgres

watchdog:
  mode: required
  device: /dev/watchdog
  safety_margin: 5

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false
	

Save and close the editor when you are finished.
Next, create a patroni.yml file on your second node (pgsql02) in our case:

sudo mkdir -p /etc/patroni

sudo nano /etc/patroni/patroni.yml
 Add following configuration:

scope: pg_cluster
namespace: /service/
name: pgsql02

restapi:
    listen: 192.168.56.232:8008
    connect_address: 192.168.56.232:8008

etcd:
    hosts: 192.168.56.231:2379,192.168.56.232:2379,192.168.56.233:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:

  initdb:
  - encoding: UTF8
  - data-checksums

  pg_hba:
  - host replication replicator 127.0.0.1/32 md5
  - host replication replicator 192.168.56.231/32 md5
  - host replication replicator 192.168.56.232/32 md5
  - host replication replicator 192.168.56.233/32 md5
  - host all all 0.0.0.0/0 md5

  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb

postgresql:
  listen: 192.168.56.232:5432
  connect_address: 192.168.56.232:5432
  data_dir: /var/lib/pgsql/15/data
  bin_dir: /usr/pgsql-15/bin
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: replicator
    superuser:
      username: postgres
      password: postgres

watchdog:
  mode: required
  device: /dev/watchdog
  safety_margin: 5

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false


Save and close the editor when you are finished.
Next, create a patroni.yml file on your third node (pgsql03) in our case:

sudo mkdir -p /etc/patroni

sudo nano /etc/patroni/patroni.yml

Add following configuration:


scope: pg_cluster
namespace: /service/
name: pgsql03

restapi:
    listen: 192.168.56.233:8008
    connect_address: 192.168.56.233:8008

etcd:
    hosts: 192.168.56.231:2379,192.168.56.232:2379,192.168.56.233:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:

  initdb:
  - encoding: UTF8
  - data-checksums

  pg_hba:
  - host replication replicator 127.0.0.1/32 md5
  - host replication replicator 192.168.56.231/32 md5
  - host replication replicator 192.168.56.232/32 md5
  - host replication replicator 192.168.56.233/32 md5
  - host all all 0.0.0.0/0 md5

  users:
    admin:
      password: admin
      options:
        - createrole
        - createdb

postgresql:
  listen: 192.168.56.233:5432
  connect_address: 192.168.56.233:5432
  data_dir: /var/lib/pgsql/15/data
  bin_dir: /usr/pgsql-15/bin
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: replicator
    superuser:
      username: postgres
      password: postgres

watchdog:
  mode: required
  device: /dev/watchdog
  safety_margin: 5

tags:
    nofailover: false
    noloadbalance: false
    clonefrom: false
    nosync: false

Save and close the editor when you are finished.

 Enable Software Watchdog
Watchdog devices are software or hardware mechanisms that will reset the whole system when they do not get a heartbeat within a specified timeframe. This adds an additional layer of fail safe in case usual Patroni split-brain protection mechanisms fail.

Patroni will try to activate the watchdog before promoting PostgreSQL to primary. Currently watchdogs are only supported using Linux watchdog device interface. For more information about watchdog, see the Patroni documentation.

Default Patroni configuration will try to use /dev/watchdog on Linux if it is accessible to Patroni. For most use cases using software watchdog built into the Linux kernel is secure enough.
 
Edit /etc/watchdog.conf to enable software watchdog:
 
sudo nano /etc/watchdog.conf
Locate, and uncomment following line:
 
watchdog-device = /dev/watchdog
Save and close the editor when you are finished.

Execute following commands to activate software watchdog:
 
sudo mknod /dev/watchdog c 10 130

sudo modprobe softdog

sudo chown postgres /dev/watchdog
Make sure you repeat the same on each node before proceeding to next.

Start Patroni Cluster
Type below command on your (pgsql01) to start your first patroni cluster node:
 
sudo systemctl start patroni
Verify patroni status using the following command:
 
sudo systemctl status patroni
If you look carefully at the bottom of the patroni status output, you will see that the (pgsql01) is acting as leader node in the cluster:
Next, start patroni on subsequent nodes, (pgsql02) for example, you will see (pgsql02) is acting as secondary node in the cluster:
Start patroni on (pgsql03), and it will also act as the secondary node in the cluster:

[root@pgsql01 ~]# systemctl status patroni
● patroni.service - Runners to orchestrate a high-availability PostgreSQL
   Loaded: loaded (/usr/lib/systemd/system/patroni.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-05-17 22:57:41 +03; 2h 33min ago
 Main PID: 40980 (patroni)
    Tasks: 15 (limit: 22938)
   Memory: 163.2M
   CGroup: /system.slice/patroni.service
           ├─40980 /usr/bin/python3 /usr/bin/patroni /etc/patroni/patroni.yml
           ├─50046 /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data --config-file=/var/lib/pgsql/15/data/postgresql.conf --listen_addresses=192.168.56.231 --po>
           ├─50047 postgres: pg_cluster: logger
           ├─50048 postgres: pg_cluster: checkpointer
           ├─50049 postgres: pg_cluster: background writer
           ├─50076 postgres: pg_cluster: postgres postgres 192.168.56.231(37600) idle
           ├─84365 postgres: pg_cluster: walwriter
           ├─84366 postgres: pg_cluster: autovacuum launcher
           ├─84367 postgres: pg_cluster: logical replication launcher
           ├─84401 postgres: pg_cluster: walsender replicator 192.168.56.233(46144) streaming 0/449CC18
           └─84425 postgres: pg_cluster: walsender replicator 192.168.56.232(39144) streaming 0/449CC18

May 18 01:29:17 pgsql01.localdomain patroni[40980]: 2023-05-18 01:29:17,074 INFO: no action. I am (pgsql01), the leader with the lock

Next, start patroni on subsequent nodes, (pgsql02) for example, you will see (pgsql02) is acting as secondary node in the cluster:	

[root@pgsql02 ~]# systemctl status patroni
● patroni.service - Runners to orchestrate a high-availability PostgreSQL
   Loaded: loaded (/usr/lib/systemd/system/patroni.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-05-17 01:34:37 +03; 2h 26min ago
 Main PID: 21349 (patroni)
    Tasks: 12 (limit: 22938)
   Memory: 139.1M
   CGroup: /system.slice/patroni.service
           ├─21349 /usr/bin/python3 /usr/bin/patroni /etc/patroni/patroni.yml
           ├─79141 /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data --config-file=/var/lib/pgsql/15/data/postgresql.conf --listen_addresses=192.168.56.232 --po>
           ├─79142 postgres: pg_cluster: logger
           ├─79143 postgres: pg_cluster: checkpointer
           ├─79144 postgres: pg_cluster: background writer
           ├─79145 postgres: pg_cluster: startup recovering 000000070000000000000004
           ├─79151 postgres: pg_cluster: walreceiver streaming 0/449CC18
           └─79153 postgres: pg_cluster: postgres postgres 192.168.56.232(35990) idle

May 17 03:59:36 pgsql02.localdomain patroni[21349]: 2023-05-17 03:59:36,947 INFO: no action. I am (pgsql02), a secondary, and following a leader (pgsql01)


Start patroni on (pgsql03), and it will also act as the secondary node in the cluster:

[root@pgsql03 ~]# systemctl status patroni
● patroni.service - Runners to orchestrate a high-availability PostgreSQL
   Loaded: loaded (/usr/lib/systemd/system/patroni.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2023-05-17 01:50:39 +03; 2h 10min ago
 Main PID: 25716 (patroni)
    Tasks: 12 (limit: 22938)
   Memory: 88.9M
   CGroup: /system.slice/patroni.service
           ├─25716 /usr/bin/python3 /usr/bin/patroni /etc/patroni/patroni.yml
           ├─25779 /usr/pgsql-15/bin/postgres -D /var/lib/pgsql/15/data --config-file=/var/lib/pgsql/15/data/postgresql.conf --listen_addresses=192.168.56.233 --po>
           ├─25780 postgres: pg_cluster: logger
           ├─25781 postgres: pg_cluster: checkpointer
           ├─25782 postgres: pg_cluster: background writer
           ├─25783 postgres: pg_cluster: startup recovering 000000070000000000000004
           ├─25790 postgres: pg_cluster: postgres postgres 192.168.56.233(51808) idle
           └─46797 postgres: pg_cluster: walreceiver streaming 0/449CC18

May 17 03:59:40 pgsql03.localdomain patroni[25716]: 2023-05-17 03:59:40,649 INFO: no action. I am (pgsql03), a secondary, and following a leader (pgsql01)

Install PgBouncer
Type following command on your oracle Linux servers to install PgBouncer:

sudo dnf install -y pgbouncer
Make sure you repeat the same on each node before proceeding to next.

 

Configure PgBouncer Authentication
PgBouncer uses /etc/pgbouncer/userlist.txt file to authenticate database clients. You can write database credentials in userlist.txt file manually using the information from the pg_shadow catalog table, or you can create a function in database to allow a specific user to query for the current password of the database users.

Direct access to pg_shadow requires admin rights. It’s preferable to use a non-superuser that calls a SECURITY DEFINER function instead.

From your (pgsql01), connect to the Postgres database as superuser, and create a security definer function:

psql -h pgsql01 -p 5432 -U postgres
Execute following at postgres=# prompt:
CREATE ROLE pgbouncer with LOGIN encrypted password 'Oracle_123';
CREATE FUNCTION public.lookup (
	INOUT p_user     name,
	OUT   p_password text
) RETURNS record
LANGUAGE sql SECURITY DEFINER SET search_path = pg_catalog AS
$$SELECT usename, passwd FROM pg_shadow WHERE usename = p_user$$;

Next, copy encrypted password of pgbouncer from pg_shadow catalog table:

select * from pg_shadow;
Type \q to exit from postgres=# prompt:

\q
Edit /etc/pgbouncer/userlist.txt file:

sudo nano /etc/pgbouncer/userlist.txt
Add pgbouncer credential like below:

"pgbouncer" "SCRAM-SHA-256$4096:WU1Z+WfQeCMsbNq+NgzPrA==$jgfcENxmGpjsKTpJW5CcC37FsXEVcRnBOMx+zKfiVOA=:/BnrKBTAwGqvjDH/x3MuMONX4Kl/V4sCy0vgM6InqZY="

Save and close the editor when you are finished. 

Edit /etc/pgbouncer/pgbouncer.ini file, to make the required changes:
 
sudo cp -p /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.orig

sudo nano /etc/pgbouncer/pgbouncer.ini
Add your database in [databases] section like below:
 
* = host=192.168.56.231 port=5432 dbname=postgres

and change listen_addr=localhost to listen_addr=*
 
listen_addr = *
In the [pgbouncer] section, add following, but below to auth_file = /etc/pgbouncer/userlist.txt line:

auth_user = pgbouncer
auth_query = SELECT p_user, p_password FROM public.lookup($1)
Save and close the editor when you are finished.

From (pgsql02), edit /etc/pgbouncer/userlist.txt file:

sudo nano /etc/pgbouncer/userlist.txt
Add pgbouncer credential like below
"pgbouncer" "SCRAM-SHA-256$4096:WU1Z+WfQeCMsbNq+NgzPrA==$jgfcENxmGpjsKTpJW5CcC37FsXEVcRnBOMx+zKfiVOA=:/BnrKBTAwGqvjDH/x3MuMONX4Kl/V4sCy0vgM6InqZY="
Save and close the editor when you are finished.

From (pgsql02), edit /etc/pgbouncer/pgbouncer.ini file, to make the required changes:
 
sudo cp -p /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.orig

sudo nano /etc/pgbouncer/pgbouncer.ini
Add your database in [databases] section like below:
 
* = host=192.168.56.232 port=5432 dbname=postgres

and change listen_addr=localhost to listen_addr=*
 
listen_addr = *
In the [pgbouncer] section, add following, but below to auth_file = /etc/pgbouncer/userlist.txt line:

auth_user = pgbouncer
auth_query = SELECT p_user, p_password FROM public.lookup($1)
Save and close the editor when you are finished.

From (pgsql03), edit /etc/pgbouncer/userlist.txt file:

sudo nano /etc/pgbouncer/userlist.txt
Add pgbouncer credential like below:
"pgbouncer" "SCRAM-SHA-256$4096:WU1Z+WfQeCMsbNq+NgzPrA==$jgfcENxmGpjsKTpJW5CcC37FsXEVcRnBOMx+zKfiVOA=:/BnrKBTAwGqvjDH/x3MuMONX4Kl/V4sCy0vgM6InqZY="

Save and close the editor when you are finished.

From (pgsql03), edit /etc/pgbouncer/pgbouncer.ini file, to make the required changes:
 
sudo cp -p /etc/pgbouncer/pgbouncer.ini /etc/pgbouncer/pgbouncer.ini.orig

sudo nano /etc/pgbouncer/pgbouncer.ini
Add your database in [databases] section like below:
 
* = host=192.168.56.233 port=5432 dbname=postgres

and change listen_addr=localhost to listen_addr=*
 
listen_addr = *
In the [pgbouncer] section, add following, but below to auth_file = /etc/pgbouncer/userlist.txt line:

auth_user = pgbouncer
auth_query = SELECT p_user, p_password FROM public.lookup($1)
Save and close the editor when you are finished.

When you are finished with pgbouncer configuration, execute below command to start pgbouncer on each node:

sudo systemctl start pgbouncer

Test PgBouncer Authentication
Make a connection to database using psql command via pgbouncer on port 6432:
 
psql -h pgsql01 -p 6432 -U pgbouncer -d postgres
This should connect you to your Postgres database if everything was configured correctly as described above.

Install HAProxy
With patroni, you need a method to connect to the leader node regardless of which of the node in the cluster is the leader. HAProxy forwards the connection to whichever node is currently the leader. It does this using a REST endpoint that Patroni provides. 
 
Patroni ensures that, at any given time, only the leader node will appear as online, forcing HAProxy to connect to the correct node. Users or applications, (psql) for example, will connect to haproxy, and haproxy will make sure connecting to the back-end database leader node in the cluster.
 
Type following command to install haproxy on your oracle Linux servers:
 
sudo dnf install -y haproxy
Make sure you repeat the same on each node before proceeding to next.
 
Configure HAProxy
Edit haproxy.cfg file on your first node (pgsql01) in our case:
 
sudo mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.orig

sudo nano /etc/haproxy/haproxy.cfg
Add following configuration:

global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     1000
    user        haproxy
    group       haproxy
    daemon
    stats socket /var/lib/haproxy/stats

defaults
    mode                    tcp
    log                     global
    option                  tcplog
    retries                 3
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout check           10s
    maxconn                 900

listen stats
    mode http
    bind *:7000
    stats enable
    stats uri /

listen primary
    bind 192.168.56.200:5000
    option httpchk OPTIONS /master
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server pgsql01 192.168.56.231:6432 maxconn 100 check port 8008
    server pgsql02 192.168.56.232:6432 maxconn 100 check port 8008
    server pgsql03 192.168.56.233:6432 maxconn 100 check port 8008

listen standby
    bind 192.168.56.200:5001
    balance roundrobin
    option httpchk OPTIONS /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server pgsql01 192.168.56.231:6432 maxconn 100 check port 8008
    server pgsql02 192.168.56.232:6432 maxconn 100 check port 8008
    server pgsql03 192.168.56.233:6432 maxconn 100 check port 8008
	
Save and close the editor when you are finished.
 
There are two important listen sections in haproxy configuration: 
primary using port 5000 for reads/writes requests to back-end database leader node.
standby using port 5001 only for reads-requests to back-end database replica nodes.
All three nodes are included in both sections: that is because all the database servers are potential candidates to be either primary or replica. Patroni provides a built-in REST API support for health check monitoring that works perfectly with HAProxy. HAProxy will send an HTTP request to port 8008 of patroni to know which role each node currently has.
The haproxy configuration will remain same on each node, 
so make sure you repeat the same configuration on your remaining nodes before proceeding to next.	

Install Keepalived
Type following command to install keepalived on your oracle Linux servers:

sudo dnf install -y keepalived
Make sure you repeat the same on each node before proceeding to next.
 
Configure Keepalived
Edit the /etc/sysctl.conf file on your (pgsql01) in our case, to allow the server to bind to the virtual IP Address.
 
sudo nano /etc/sysctl.conf
Add following at the end of the file:
 
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
Save and close the editor when you are finished.
Type following command on your Linux server to reload settings from config file without rebooting:

sudo sysctl --system
sudo sysctl -p
Make sure you repeat the same on each node before proceeding to next.
Create /etc/keepalived/keepalived.conf file on your first node (pgsql01) in our case, to add the required configuration:

cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.orig

sudo nano /etc/keepalived/keepalived.conf 
Add following configuration:
 
vrrp_script chk_haproxy {
        script "pkill -0 haproxy"
        interval 5
        weight -4
        fall 2
        rise 1
}

vrrp_script chk_lb {
        script "pkill -0 keepalived"
        interval 1
        weight 6
        fall 2
        rise 1
}

vrrp_script chk_servers {
        script "echo 'GET /are-you-ok' | nc 127.0.0.1 7000 | grep -q '200 OK'"
        interval 2
        weight 2
        fall 2
        rise 2
}

vrrp_instance vrrp_1 {
        interface enp0s3
        state MASTER
        virtual_router_id 51
        priority 101
        virtual_ipaddress_excluded {
                192.168.56.200
        }
        track_interface {
                enp0s3 weight -2
        }
        track_script {
                chk_haproxy
                chk_lb
        }
}

Do not forget to replace the highlighted text with yours. Save and close the editor when you are finished. 

Create /etc/keepalived/keepalived.conf file on your second node (pgsql02) in our case:
 
sudo nano /etc/keepalived/keepalived.conf 
Add following configuration:
 
vrrp_script chk_haproxy {
        script "pkill -0 haproxy"
        interval 5
        weight -4
        fall 2
        rise 1
}

vrrp_script chk_lb {
        script "pkill -0 keepalived"
        interval 1
        weight 6
        fall 2
        rise 1
}

vrrp_script chk_servers {
        script "echo 'GET /are-you-ok' | nc 127.0.0.1 7000 | grep -q '200 OK'"
        interval 2
        weight 2
        fall 2
        rise 2
}

vrrp_instance vrrp_1 {
        interface enp0s3
        state BACKUP
        virtual_router_id 51
        priority 100
        virtual_ipaddress_excluded {
                192.168.56.200
        }
        track_interface {
                enp0s3 weight -2
        }
        track_script {
                chk_haproxy
                chk_lb
        }
}

Do not forget to replace the highlighted text with yours. Save and close the editor when you are finished.
 
Create /etc/keepalived/keepalived.conf file on your third node (pgsql03) in our case:
 
sudo nano /etc/keepalived/keepalived.conf 
Add following configuration:
 
vrrp_script chk_haproxy {
        script "pkill -0 haproxy"
        interval 5
        weight -4
        fall 2
        rise 1
}

vrrp_script chk_lb {
        script "pkill -0 keepalived"
        interval 1
        weight 6
        fall 2
        rise 1
}

vrrp_script chk_servers {
        script "echo 'GET /are-you-ok' | nc 127.0.0.1 7000 | grep -q '200 OK'"
        interval 2
        weight 2
        fall 2
        rise 2
}

vrrp_instance vrrp_1 {
        interface enp0s3
        state BACKUP
        virtual_router_id 51
        priority 99
        virtual_ipaddress_excluded {
                192.168.56.200
        }
        track_interface {
                enp0s3 weight -2
        }
        track_script {
                chk_haproxy
                chk_lb
        }
}

Do not forget to replace the highlighted text with yours. Save and close the editor when you are finished.
 
Type below command on your first node (pgsql01) to start keepalived:
 
sudo systemctl start keepalived
Check on your (MASTER) node to see if your (enp0s3) network interface has configured with an additional shared IP (192.168.56.200):
 
ip addr show enp0s3
You will see the output similar to like below:

[root@pgsql01 ~]# ip addr show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ae:bb:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.231/24 brd 192.168.56.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.56.200/32 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::245d:20e3:21f9:4f07/64 scope link noprefixroute
       valid_lft forever preferred_lft forever


With this keepalived configuration, if MASTER haproxy node goes down for any reason, shared IP Address will automatically float to a BACKUP haproxy node, and connectivity to database clients will remain available.

Type following command to start HAProxy on each node: 

sudo systemctl start haproxy
Verify HAProxy status on each node:
 
sudo systemctl status haproxy
We have already taken care of HAProxy high availability with a shared IP Address using keepalived configuration earlier.
You can manually test HAProxy failover scenario by killing haproxy on your MASTER  node with sudo systemctl stop haproxy command, you will see, within few seconds of delay, shared IP Address (192.168.56.200) will automatically float to your BACKUP haproxy node.

 

Test Patroni Cluster
You can test your Patroni cluster by initiating a database connection request from any of your applications (psql) for example using your (shared_ip:port):
 
psql -h 192.168.56.200 -p 5000 -U postgres
If everything was configured properly as described in the tutorial, this will connect you to your master database as you can see in the image below.
[root@pgsql01 ~]# psql -h 192.168.56.200 -p 5000 -U postgres
Password for user postgres:
psql (15.3)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "192.168.56.200" at port "5000".
postgres=# \l
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =Tc/postgres         +
           |          |          |             |             |            |                 | postgres=CTc/postgres+
           |          |          |             |             |            |                 | testuser=CTc/postgres
Next, execute two reads-requests to verify HAProxy round-robin load balancing mechanism is working as intended:
 
psql -h 192.168.56.200 -p 5001 -U postgres -t -c "select inet_server_addr()"
This should return your replica node IP as shown in image below:
 [root@pgsql01 ~]# psql -h 192.168.56.200 -p 5001 -U postgres -t -c "select inet_server_addr()"
Password for user postgres:
 192.168.56.232

You can access HAProxy dashboard by typing http://shared-ip:7000/ in your browser address bar.

As you can see, in the primary section (pgsql01) row is highlighted in green. This indicates that 192.168.56.231 is currently a leader node in the cluster.
 
In the standby section, the (pgsql02, pgsql03) row is highlighted as green. This indicates that both nodes are replica node in the cluster.

If you kill the leader node using (sudo systemctl stop patroni) or by completely shutting down the server, the dashboard will look similar to like below:

As you can see, in the primary section (pgsql02) row is now highlighted in green. This indicates that 192.168.56.232 is currently a leader node in the cluster.

Please note that, in this particular scenario, it just so happens that the second node in the cluster is promoted to leader. This might not always be the case and it is equally likely that the 3rd node may be promoted to leader.
 
Test Postgres Database Replication
We will create a test database to see if it is replicated to other nodes in the cluster. For this guide, we will use (psql) to connect to database via haproxy like below: 

psql -h 192.168.56.200 -p 5000 -U postgres
From the Postgres prompt, create a test database like below:
 
create database testdb;
create user testuser with encrypted password 'mystrongpass';
grant all privileges on database testdb to testuser;

\q
Update your userlist.txt file on each node for testuser as explained in PgBouncer section.
 
Stop patroni on leader node (pgsql01) in our case with below command:
 
sudo systemctl stop patroni
Connect to database using psql, and this time haproxy will automatically make connection to  whichever node is currently leader in the cluster: 

psql -h 192.168.56.200 -p 5000 testuser -d testdb
As you can see in the output below, connection to testdb was successful via haproxy:

Now bring up your first node with (sudo systemctl start patroni), and it will automatically rejoin the cluster as secondary and automatically synchronize with the leader.
 
Patroni Cluster Failover
With patronictl command you can administer, manage and troubleshoot your patroni cluster.
 
Type following command to list the options and commands you can use with patronictl: 

patronictl --help
This will show you the options and commands you can use with patronictl.

Options:
  -c, --config-file TEXT  Configuration file
  -d, --dcs TEXT          Use this DCS
  -k, --insecure          Allow connections to SSL sites without certs
  --help                  Show this message and exit.

Commands:
  configure    Create configuration file
  dsn          Generate a dsn for the provided member, defaults to a dsn of...
  edit-config  Edit cluster configuration
  failover     Failover to a replica
  flush        Discard scheduled events (restarts only currently)
  history      Show the history of failovers/switchovers
  list         List the Patroni members for a given Patroni
  pause        Disable auto failover
  query        Query a Patroni PostgreSQL member
  reinit       Reinitialize cluster member
  reload       Reload cluster member configuration
  remove       Remove cluster from DCS
  restart      Restart cluster member
  resume       Resume auto failover
  scaffold     Create a structure for the cluster in DCS
  show-config  Show cluster configuration
  switchover   Switchover to a replica
  version      Output version of patronictl command or a running Patroni 
You can check patroni cluster member nodes using the following command:
 
patronictl -c /etc/patroni/patroni.yml list

[root@pgsql01 ~]# patronictl -c /etc/patroni/patroni.yml list
2023-05-18 04:44:17,030 - WARNING - /usr/lib/python3.6/site-packages/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.26.15) or chardet (3.0.4) doesn't match a supported version!
  RequestsDependencyWarning)

+ Cluster: pg_cluster -----+---------+---------+----+-----------+
| Member  | Host           | Role    | State   | TL | Lag in MB |
+---------+----------------+---------+---------+----+-----------+
| pgsql01 | 192.168.56.231 | Leader  | running |  7 |           |
| pgsql02 | 192.168.56.232 | Replica | running |  7 |         0 |
| pgsql03 | 192.168.56.233 | Replica | running |  7 |         0 |
+---------+----------------+---------+---------+----+-----------+


[root@pgsql01 ~]# patronictl -c /etc/patroni/patroni.yml history
2023-05-18 04:44:57,499 - WARNING - /usr/lib/python3.6/site-packages/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.26.15) or chardet (3.0.4) doesn't match a supported version!
  RequestsDependencyWarning)

+----+----------+------------------------------+----------------------------------+------------+
| TL |      LSN | Reason                       | Timestamp                        | New Leader |
+----+----------+------------------------------+----------------------------------+------------+
|  1 | 67381856 | no recovery target specified | 2023-05-17T01:25:57.682882+03:00 | pgsql02    |
|  2 | 71936464 | no recovery target specified | 2023-05-17T01:29:07.191895+03:00 | pgsql03    |
|  3 | 71940536 | no recovery target specified | 2023-05-17T23:08:48.404954+03:00 | pgsql01    |
|  4 | 71941864 | no recovery target specified | 2023-05-17T01:44:25.443959+03:00 | pgsql03    |
|  5 | 71942288 | no recovery target specified | 2023-05-17T01:46:24.560966+03:00 | pgsql02    |
|  6 | 71943912 | no recovery target specified | 2023-05-18T01:29:15.956906+03:00 | pgsql01    |
+----+----------+------------------------------+----------------------------------+------------+
In patroni cluster, failover is executed automatically when a leader node getting unavailable for unplanned reason. If you want to test your patroni cluster failover manually, you can initiate failover to a replica node using the following command:

patronictl -c /etc/patroni/patroni.yml failover

[root@pgsql01 ~]# patronictl -c /etc/patroni/patroni.yml failover
2023-05-18 01:29:04,924 - WARNING - /usr/lib/python3.6/site-packages/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.26.15) or chardet (3.0.4) doesn't match a supported version!
  RequestsDependencyWarning)

Current cluster topology
+ Cluster: pg_cluster -----+---------+---------+----+-----------+
| Member  | Host           | Role    | State   | TL | Lag in MB |
+---------+----------------+---------+---------+----+-----------+
| pgsql01 | 192.168.56.231 | Replica | running |  6 |         0 |
| pgsql02 | 192.168.56.232 | Leader  | running |  6 |           |
| pgsql03 | 192.168.56.233 | Replica | running |  6 |         0 |
+---------+----------------+---------+---------+----+-----------+
Candidate ['pgsql01', 'pgsql03'] []: pgsql01
Are you sure you want to failover cluster pg_cluster, demoting current leader pgsql02? [y/N]: y
2023-05-18 01:29:16.19920 Successfully failed over to "pgsql01"
+ Cluster: pg_cluster -----+---------+---------+----+-----------+
| Member  | Host           | Role    | State   | TL | Lag in MB |
+---------+----------------+---------+---------+----+-----------+
| pgsql01 | 192.168.56.231 | Leader  | running |  6 |           |
| pgsql02 | 192.168.56.232 | Replica | stopped |    |   unknown |
| pgsql03 | 192.168.56.233 | Replica | running |  6 |         0 |
+---------+----------------+---------+---------+----+-----------+

Disable Patroni Cluster Auto Failover
In some cases, it is necessary to perform maintenance task on a single node, such as applying patches or release updates. When you disable auto failover, patroni won’t change the state of the cluster.

You can disable patroni cluster auto failover using the following command:

patronictl -c /etc/patroni/patroni.yml pause
 

Patroni Cluster Switchover
There are two possibilities to run a switchover, either in scheduled mode or immediately. At the given time, the switchover will take place, and you will see an entry of switchover activity in the log file.

You can test patroni cluster switchover using the following command:
patronictl -c /etc/patroni/patroni.yml switchover --master your_leader_node --candidate your_replica_node
If you go with [now] option, switchover will take place immediately.

To keep running services after reboot must enable as below
systemctl enable etcd
systemctl enable patroni
systemctl enable pgbouncer
systemctl enable keepalived
systemctl enable haproxy
