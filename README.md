[简体中文](README_cn.md)

Name
====

* nginx-tfs

Description
===========

* This project is forked from **alibaba/nginx-tfs** which was discontinued itself and integrated in **[alibaba/tengine](https://github.com/alibaba/tengine)**.
* Changes from **alibaba/nginx-tfs**:
  * All changes of TFS module in **alibaba/tengine** were merged back with original author & commit date kept.
  * Support OpenResty / official Nginx 1.0.x-1.11.x and Tengine 1.2.1-2.2.0 .
* This module implements an asynchronous client of TFS(Taobao File System), providing [RESTful API](TFS_RESTful_API.md) to it. [TFS](http://tfs.taobao.org) is a distributed file system developed by Taobao Inc.

Install
=======

1. TFS use an open source JSON library for JSON support. Please install [yajl](http://lloyd.github.com/yajl/)-2.0.1 or later first.

2. Download [Nginx](http://www.nginx.org/) or [Tengine](http://tengine.taobao.org/).

3. ./configure --add-module=/path/to/nginx-tfs

4. make && make install

Example
====

    http {
        #tfs_upstream tfs_rc {
        #    server 127.0.0.1:6100;
        #    type rcs;
        #    rcs_zone name=tfs1 size=128M;
        #    rcs_interface eth0;
        #    rcs_heartbeat lock_file=/logs/lk.file interval=10s;
        #}

        tfs_upstream tfs_ns {
            server 127.0.0.1:8100;
            type ns;
        }

        server {
              listen       7500;
              server_name  localhost;

              tfs_keepalive max_cached=100 bucket_count=10;
              tfs_log "pipe:/usr/sbin/cronolog -p 30min /path/to/nginx/logs/cronolog/%Y/%m/%Y-%m-%d-%H-%M-tfs_access.log";

              location / {
                  tfs_pass tfs://tfs_ns;
              }
        }
    }

Directives
==========

tfs\_upstream
----------------

**Syntax**： *tfs\_upstream name {...}*

**Default**： *none*

**Context**： *http*

Defines information of upstream TFS server.

Example:

    tfs_upstream tfs_rc {
        server 127.0.0.1:6100;
        type rcs;
        rcs_zone name=tfs1 size=128M;
        rcs_interface eth0;
        rcs_heartbeat lock_file=/logs/lk.file interval=10s;
    }

    tfs_upstream tfs_ns {
        server 127.0.0.1:8100;
        type ns;
    }

server
------------

**Syntax**： *server address*

**Default**： *none*

**Context**： *tfs_upstream*

Defines the address of upstream TFS server. An address can be specified as a domain name or IP address. This directive must be defined.

Example :

	server 10.0.0.1:8108;

type
----------------

**Syntax**： *type [ns | rcs]*

**Default**： *ns*

**Context**： *tfs_upstream*

Specify the type of upstream TFS server. It could be ns(NameServer) or rcs(RcServer), default is ns.

It must be rcs if user-defined name is used. User-defined name also require MetaServer & RootServer.

rcs\_zone
--------------

**Syntax**： *rcs_zone name=n size=num*

**Default**： *none*

**Context**： *tfs_upstream*

Defines the shared memory zone used to store application configuration information registerd in RcServer. This directive is mandatory when upstream is RcServer. Application configuration information can be updated through heartbeat with RcServer(directive <i>rcs_heartbeat</i>).

Example:

	rcs_zone name=tfs1 size=128M;

rcs\_heartbeat
--------------

**Syntax**： *rcs_heartbeat lock_file=/path/to/file interval=time*

**Default**： *none*

**Context**： *tfs_upstream*

Enable heartbeat with RcServer so that application configuration information can be updated in time. It is mandatory when upstream is RcServer.

The following parameters must be defined:

lock_file=<i>/path/to/file</i>
    use to create a mutex so that only one Nginx worker can do heartbeat at one time.
interval=<i>time</i>
    set heartbeat interval.

Example:

	rcs_heartbeat lock_file=/path/to/nginx/logs/lk.file interval=10s;

rcs\_interface
----------------

**Syntax**： *rcs\_interface interface*

**Default**： *none*

**Context**： *tfs_upstream*

Specify net interface used by TFS module. It is used to get local IP address. It is only mandatory when upstream is RCServer.

Example:

	rcs_interface eth0;

tfs_pass
--------

**Syntax**： *tfs_pass name*

**Default**： *none*

**Context**： *location*

Specify TFS upstream. It must be defined.

Remember that protocol used here must be "tfs".

Example:

	tfs_upstream tfs_rc {
    };

	location / {
		tfs_pass tfs://tfs_rc;
	}

tfs_keepalive
-------------

**Syntax**： *tfs_keepalive max_cached=num bucket_count=num*

**Default**： *none*

**Context**： *http, server*

Defines connection pool used by TFS module. This connection pool caches upstream connections. This directive must be defined.


The connection pool can be regarded as hash table with many buckets.

The following parameters must be defined:

max_cached=<i>num</i>
    set the capacity of one hash bucket.
bucket_count=<i>num</i>
    set the count of hash buckets.

Note: The size of physical memory should be considered to define the capacity.

Example:

	tfs_keepalive max_cached=100 bucket_count=15;

tfs\_block\_cache\_zone
-----------------------

**Syntax**： *tfs_block_cache_zone size=num*

**Default**： *none*

**Context**： *http*

Defines the shared memory zone used for BlockCache.

Note: The size of physical memory should be considered to define the capacity.

Example:

	tfs_block_cache_zone size=256M;

tfs\_log
----------------

**Syntax**： *tfs_log path*

**Default**： *none*

**Context**： *http, server*

Sets the TFS access log.

If this directive was defined, the TFS access log will recorded to specified log file in certain format. See source code for the format.
Example:

	tfs_log "pipe:/usr/sbin/cronolog -p 30min /path/to/nginx/logs/cronolog/%Y/%m/%Y-%m-%d-%H-%M-tfs_access.log";

Note: cronolog support depends on log extension of tengine.

tfs\_body\_buffer\_size
-----------------------

**Syntax**： *tfs_body_buffer_size size*

**Default**： *2m*

**Context**： *http, server, location*

Sets the buffer size for reading upstream response. By default, buffer size is 2m.

Example:

	tfs_body_buffer_size 2m;

tfs\_connect\_timeout
---------------------

**Syntax**： *tfs_connect_timeout time*

**Default**： *3s*

**Context**： *http, server, location*

Sets a timeout for connecting upstream servers.

tfs\_send\_timeout
------------------

**Syntax**： *tfs_send_timeout time*

**Default**： *3s*

**Context**： *http, server, location*

Sets a timeout for transmitting data to upstream servers.

tfs\_read\_timeout
------------------

**Syntax**： *tfs_read_timeout time*

**Default**： *3s*

**Context**： *http, server, location*

Sets a timeout for reading data from upstream servers.

Others
------
Uploading file size supported depends on the directive <i>client_max_body_size</i>.
