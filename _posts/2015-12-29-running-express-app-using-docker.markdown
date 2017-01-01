---
layout: post
title:  "Running An Express App Using Docker"
date:   2015-12-29 16:35:15
categories: docker javascript express
---

I have spent much of the past month building a simple photo-sharing application
using [Express][express] and running in [Docker][docker].

In my initial version of the application, I had a

# Main Application Code

{% highlight javascript %}
'use strict';

var express = require('express');
...
var database = require('./models/database');
var app = express();

/* database setup */
database.createTable(function () {
  console.log('table created');
});
{% endhighlight %}

# Database Code

{% highlight javascript %}
var Sequelize = require('sequelize');
var pgHost = process.env.DB_PORT_5432_TCP_ADDR || 'localhost';

/* sequelize setup */
var sequelize = new Sequelize('production', 'postgres', null, {
  host: pgHost,
  dialect: 'postgres',
});

var User = sequelize.define('user', {
  ...
});

var createTable = function(cb) {
  User.sync().then(function() {
    cb();
  });
}
{% endhighlight %}

# Initialization Difficulties
After the initial build

Docker Compose setup:

{% highlight yaml %}
db:
  build: 'dockerfiles/db'

web:
  build: 'dockerfiles/web'
  volumes:
    - "./app:/app"
  ports:
    - "3000:3000"
  links:
    - db
{% endhighlight %}

{% highlight bash %}
$ docker-compose up
db_1  | The files belonging to this database system will be owned by user "postgres".
db_1  | This user must also own the server process.
db_1  | 
db_1  | The database cluster will be initialized with locale "en_US.utf8".
db_1  | The default database encoding has accordingly been set to "UTF8".
db_1  | The default text search configuration will be set to "english".
db_1  | 
db_1  | Data page checksums are disabled.
db_1  | 
db_1  | fixing permissions on existing directory /var/lib/postgresql/data ... ok
db_1  | creating subdirectories ... ok
db_1  | selecting default max_connections ... 100
db_1  | selecting default shared_buffers ... 128MB
db_1  | selecting dynamic shared memory implementation ... posix
db_1  | creating configuration files ... ok
web_1 | npm info it worked if it ends with ok
web_1 | npm info using npm@3.3.12
web_1 | npm info using node@v5.3.0
web_1 | npm info lifecycle signup@0.0.0~prestart: signup@0.0.0
web_1 | npm info lifecycle signup@0.0.0~start: signup@0.0.0
web_1 | 
web_1 | > signup@0.0.0 start /app
web_1 | > nodemon -L bin/www
web_1 | 
db_1  | creating template1 database in /var/lib/postgresql/data/base/1 ... ok
db_1  | initializing pg_authid ... ok
db_1  | initializing dependencies ... ok
db_1  | creating system views ... ok
web_1 | [nodemon] 1.8.1
web_1 | [nodemon] to restart at any time, enter `rs`
web_1 | [nodemon] watching: *.*
web_1 | [nodemon] starting `node bin/www`
db_1  | loading system objects' descriptions ... ok
db_1  | creating collations ... ok
db_1  | creating conversions ... ok
db_1  | creating dictionaries ... ok
db_1  | setting privileges on built-in objects ... ok
db_1  | creating information schema ... ok
db_1  | loading PL/pgSQL server-side language ... ok
db_1  | vacuuming database template1 ... ok
db_1  | copying template1 to template0 ... ok
db_1  | copying template1 to postgres ... ok
db_1  | syncing data to disk ... ok
db_1  | 
db_1  | WARNING: enabling "trust" authentication for local connections
db_1  | You can change this by editing pg_hba.conf or using the option -A, or
db_1  | --auth-local and --auth-host, the next time you run initdb.
db_1  | 
db_1  | Success. You can now start the database server using:
db_1  | 
db_1  |     postgres -D /var/lib/postgresql/data
db_1  | or
db_1  |     pg_ctl -D /var/lib/postgresql/data -l logfile start
db_1  | 
db_1  | ****************************************************
db_1  | WARNING: No password has been set for the database.
db_1  |          This will allow anyone with access to the
db_1  |          Postgres port to access your database. In
db_1  |          Docker's default configuration, this is
db_1  |          effectively any other container on the same
db_1  |          system.
db_1  | 
db_1  |          Use "-e POSTGRES_PASSWORD=password" to set
db_1  |          it in "docker run".
db_1  | ****************************************************
db_1  | waiting for server to start....LOG:  database system was shut down at 2015-12-31 20:21:49 UTC
db_1  | LOG:  MultiXact member wraparound protections are now enabled
db_1  | LOG:  database system is ready to accept connections
db_1  | LOG:  autovacuum launcher started
web_1 | Unhandled rejection SequelizeConnectionRefusedError: connect ECONNREFUSED 172.17.0.2:5432
web_1 |     at /app/node_modules/sequelize/lib/dialects/postgres/connection-manager.js:89:20
web_1 |     at null.<anonymous> (/app/node_modules/pg/lib/client.js:176:5)
  web_1 |     at emitOne (events.js:77:13)
  web_1 |     at emit (events.js:169:7)
  web_1 |     at Socket.<anonymous> (/app/node_modules/pg/lib/connection.js:59:10)
  web_1 |     at emitOne (events.js:77:13)
  web_1 |     at Socket.emit (events.js:169:7)
  web_1 |     at emitErrorNT (net.js:1256:8)
  web_1 |     at nextTickCallbackWith2Args (node.js:455:9)
  web_1 |     at process._tickCallback (node.js:369:17)
  web_1 | From previous event:
  web_1 |     at ConnectionManager.connect (/app/node_modules/sequelize/lib/dialects/postgres/connection-manager.js:80:10)
  web_1 |     at ConnectionManager.$connect (/app/node_modules/sequelize/lib/dialects/abstract/connection-manager.js:260:41)
  web_1 |     at ConnectionManager.getConnection (/app/node_modules/sequelize/lib/dialects/abstract/connection-manager.js:217:44)
  web_1 |     at Sequelize.query (/app/node_modules/sequelize/lib/sequelize.js:796:83)
  web_1 |     at /app/node_modules/sequelize/lib/query-interface.js:157:31
  web_1 | From previous event:
  web_1 |     at Promise.then (/app/node_modules/sequelize/lib/promise.js:21:17)
  web_1 |     at Model.sync (/app/node_modules/sequelize/lib/model.js:980:6)
  web_1 |     at Object.createTable (/app/models/database.js:39:8)
  web_1 |     at Object.<anonymous> (/app/app.js:24:10)
  web_1 |     at Module._compile (module.js:398:26)
  web_1 |     at Object.Module._extensions..js (module.js:405:10)
  web_1 |     at Module.load (module.js:344:32)
  web_1 |     at Function.Module._load (module.js:301:12)
  web_1 |     at Module.require (module.js:354:17)
  web_1 |     at require (internal/module.js:12:17)
  web_1 |     at Object.<anonymous> (/app/bin/www:7:11)
  web_1 |     at Module._compile (module.js:398:26)
  web_1 |     at Object.Module._extensions..js (module.js:405:10)
  web_1 |     at Module.load (module.js:344:32)
  web_1 |     at Function.Module._load (module.js:301:12)
  web_1 |     at Function.Module.runMain (module.js:430:10)
  web_1 |     at startup (node.js:141:18)
  web_1 |     at node.js:980:3
  db_1  |  done
  db_1  | server started
  db_1  | ALTER ROLE
  db_1  | 
  db_1  | 
  db_1  | /docker-entrypoint.sh: running /docker-entrypoint-initdb.d/bootstrap.sh
  db_1  | ++ createdb -U postgres test
  db_1  | ++ createdb -U postgres production
  db_1  | ++ set +x
  db_1  | 
  db_1  | LOG:  received fast shutdown request
  db_1  | LOG:  aborting any active transactions
  db_1  | LOG:  autovacuum launcher shutting down
  db_1  | LOG:  shutting down
  db_1  | waiting for server to shut down...LOG:  database system is shut down
  db_1  | . done
  db_1  | server stopped
  db_1  | 
  db_1  | PostgreSQL init process complete; ready for start up.
  db_1  | 
  db_1  | LOG:  database system was shut down at 2015-12-31 20:21:51 UTC
  db_1  | LOG:  MultiXact member wraparound protections are now enabled
  db_1  | LOG:  database system is ready to accept connections
  db_1  | LOG:  autovacuum launcher started
{% endhighlight %}

[express]: http://expressjs.com/ 
[docker]:  https://www.docker.com/
