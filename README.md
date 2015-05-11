# pdns-backend-pgsql-cidr-based-bulk-cpe
PowerDNS backend for CIDR based bulk CPE definitions (via pgsql)

Introduction
------------

This quasi-backend coerces PowerDNS into serving forward and reverse IP
mappings based on a printf format and cidr ranges. This is done via PostgreSQL,
which has nice ip address handing functions.

Rather than having millions of forward and reverse records, you merely define
two rows of authority, one or more template strings, then a few rows of CIDR's.

I thought about doing it via the lua backend or a new c++ backend, but I still
needed to provide a store for the data - so PostgreSQL seemed like a reasonable
vehicle for an experiment / proof of concept.

WARNING
-------

This is a quick proof of concept. Call it a "hack" if you like.

See the disclaimer in the GPLv2 license, then read it over again :)

Installation
------------

This is intended for PostgreSQL 9.1+. The ip4r extension is used
along with PostgreSQL's native ip addressiing functions. MySQL
doesnt have network adress functions, so don't ask :P

 su postgres
 createuser -W
 # u: pdns p: pdns, no other privileges
 createdb pdns
 psql pdns < ./schema

Test with:

 #cd to same directory as pdns.conf
 pdns_server --daemon=no --query-logging=yes --loglevel=10 --config-dir=.
 nslookup 100.68.123.250 127.0.0.1
 nslookup cnat-100-68-123-250.cnat.acme.com. 127.0.0.1

Table Structure
---------------

 * cpe_formats - this is where the format of domain names is defined. See notes
 * cpe_ranges - ip range cidrs are defined here, and reference cpe_formats.id
 * cpe_authorites - authorities here define universally for all cids and formats
 * cpe_comainmetadata - this keeps powerdns happy. just leave it empty


Notes
-----

A range of non-public addresses are included in the schema for reference.
These include TEST1, Carrier NAT, rfc1918 and Benchmarks networks.

These are mapped forward and reverse to names within acme.com namespace.

The cpe_formats table will let you define dns names as 1.2.3.4 or 4.3.2.1,
however the function that does forward lookups will fail with 4.3.2.1 -
but you can fix it and send a pull request!

Zone transfers don't work, nor are they designed to. Name servers should have
their own local postgresql database which is kept up to date outside of pdns

domain_id's are an integer which is crudely generated based on the domain name.
PowerDNS seems happy enough with this (even with negative integers). On the
downside, PowerDNS really wants to run 'update' queries, which for now are
neutered with "AND 1=0".

TODO
----

* Map all of the queries to something appropriate
* Ensure that everything PowerDNS might do is mapped to something sane
* Refine the PostgreSQL functions for performance and sanity
* For example, there may be needless/wasteful casting of data types
* Currently there are NO INDEXES
* Given the small number of rows required for millions of records, there may
  only be minor performance benefits. ip4r's 'iprange' type was used in case
  indexes are wanted.
