# Pigsty - PostgreSQL Deploy Template

> PIGSTY: Postgres Installation Graphic Standard Template.Yaml

This repo provides a PostgreSQL testing environment based on [vagrant](https://vagrantup.com/) consist of 4 vm nodes.

It is a template for illustrating some practice about running PostgreSQL in real world. A minimal HA cluster consist of 3 nodes (primary, standby, offline) and a battery included control center (monitoring, alerting, DCS, HA, yum..)



## Architecture

This cluster contains 4 nodes running following services:

| Node  | IP          | Default Role       | Service                                                      |
| ----- | ----------- | ------------------ | ------------------------------------------------------------ |
| node0 | 10.10.10.10 | control node       | Nginx, Grafana, Prometheus, Altermanger, Dnsmasq, PgAdmin4, Consul |
| node1 | 10.10.10.11 | 1.testdb (primary) | consul, postgres, pgbouncer, patroni, node_exporter, pg_exporter, pgbouncer_exporter |
| node2 | 10.10.10.12 | 2.testdb (standby) | same as above                                                |
| node3 | 10.10.10.13 | 3.testdb (standby) | same as above                                                |



![](doc/img/architecture.png)

**Control Node Services** 

* Nginx (10.10.10.10:80)：Local yum server, and proxy for grafana, prometheus, alertmanager, consul, etc...
* Yum (10.10.10.10:80): http://yum.pigsty/pigsty/
* Consul (10.10.10.10:8500): http://c.pigsty/
* Consul DNS Service (10.10.10.10:8600)
* Grafana (10.10.10.10:3000): http://g.pigsty/
* Prometheus (10.10.10.10:9090): http://p.pigsty/
* AlertManager (10.10.10.10:9093): http://am.pigsty/

* DNSMASQ (10.10.10.10:53)

**DB Node Services**

* consul (:8500) agent mode
* postgres (:5432) major service
*  pgbouncer (:6432) connection pool
* patroni (:8008) HA agent
* node_exporter (:9100) prometheus machine metrics collector 
* pg_exporter (:9630) prometheus postgres metrics collector
* pgbouncer_exporter (:9630) prometheus pgbouncer metrics collector

**Domain Names**

Default database cluster is named 'testdb'

`testdb` is a virtual service registered to consul, which have optional tags: `primary`, `standby` ,`1`,`2`,`3`. You can discover these services via common domain names. 

* TestDB primary service domain name: `primary.testdb.service.consul`
* TestDB standby service domain name: `standby.testdb.service.consul` 

**DSN**

Default database name: `testdb`, default business user is `dbuser_test` and password is `dbuser_test` too

```bash
# connect to primary
psql postgres://dbuser_test:dbuser_test@primary.testdb.service.consul:5432/testdb

# conntect to standby (random choose one)
psql postgres://dbuser_test:dbuser_test@standby.testdb.service.consul:5432/testdb
```







## Quick Start

1. Install [vagrant](https://vagrantup.com/) and [virtualbox](https://www.virtualbox.org/)
2. `git clone https://github.com/vonng/pigsty && cd pigsty`
3. `make dns` (one-time job, write DNS records to `/etc/hosts`)
4. `./build.sh` (pull up vm cluster, setup control node and init postgres cluster)
5. Open http://pigsty  (pigsty is a local dns record created in step 3 )

Then you will have a running PostgreSQL cluster with a battery included monitoring/managing system.

> **Tips**: bootstrap pigsty may takes a long time to download yum packages. Once you have a functional pigsty instance. Run `make cache` to copy rpm packages to `pigsty/control/yum`, so next time it would be much faster (15min to 1min)



## What's Next?

* Explore the monitoring system
* Add some load to cluster
* HA scenario
* Managing postgres cluster with ansible
* blah blah...



## File structure

* [control](control/) contains resource for control node: yum rpms, scripts, ansible playbooks, monitoring system,...
* [ansible](ansible/) contains ansible playbooks to manage this database cluster





## About

Author：Vonng ([fengruohang@outlook.com](mailto:fengruohang@outlook.com))
