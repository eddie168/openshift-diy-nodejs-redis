openshift-diy-nodejs-redis
==========================

Thanks for the great work by [razorinc](https://github.com/razorinc/redis-openshift-example) and [creationix](https://github.com/creationix/nvm/), this repo let you test Node.js (v0.8.6 and above) or io.js (v1.0.0 or above) with Redis (as found in [here](http://download.redis.io/releases/) ) in a OpenShift DIY application. For Node.js, it will first check for pre-compiled linux version, then compile from source if not found. For io.js, this script only support version that have pre-compiled binary. Redis will be compiled from source code. Subsequent `push` will skip source code compile unless a new version is specified.

[node-supervisor](https://github.com/isaacs/node-supervisor) is used to automatically restart the node.js app if somehow crashed.

**@Feb 05, 2015: Note that hiredis-node `0.1.17` doesn't support io.js yet!**

Usage
-----

Create an DIY app

    rhc app create -t diy-0.1 -a yourapp

Add this repository

    cd yourapp
    git remote add nodejsRedis -m master git://github.com/eddie168/openshift-diy-nodejs-redis.git
    git pull -s recursive -X theirs nodejsRedis master

Edit `config_diy.json` then push the repo to openshift

    git push

If pre-compiled node.js binary is not available, first push will take a while to finish.

You can specify the node.js script to start with in `package.json` as described [here](https://openshift.redhat.com/community/kb/kb-e1048-how-can-i-run-my-own-nodejs-script).

Check the end of the message for io.js/node.js and Redis version:

    remote: Starting DIY cartridge
    remote: Node Version:
    remote: { http_parser: '2.3.0',
    remote:   node: '1.1.0',
    remote:   v8: '4.1.0.14',
    remote:   uv: '1.3.0',
    remote:   zlib: '1.2.8',
    remote:   ares: '1.10.0-DEV',
    remote:   modules: '43',
    remote:   openssl: '1.0.1k' }
    remote: Redis Version:
    remote: redis-cli 2.8.19
    remote: nohup supervisor server.js >/var/lib/openshift/xxxxxxxxxxxxxxxxxx/app-root/logs/server.log 2>&1 &

In this case it is io.js `v1.1.0` and redis `2.8.19`

To check is redis working

    rhc tail -a yourapp

You should see something like this in `app-root/logs/server.log`

    Reply: OK
    Reply: 0
    Reply: 0
    2 replies:
        0: hashtest 1
        1: hashtest 2

You can find io.js/node.js app's log at `$OPENSHIFT_DIY_LOG_DIR/server.log`. Subsequent `push` will rename the log file with a time stamp before overwritten. The same goes to Redis log file and can be found at `$OPENSHIFT_DIY_LOG_DIR/redis.log`. 

Now open your openshift app in browser and you should see the standard openshift sample page. Enjoy!!

Settings
--------

Edit `config_diy.json`

    {
      "nodejs": {
        "use_iojs": true,
        "version": "v1.1.0",
        "removeOld": true
      },
      "redis": {
        "version": "2.8.19",
        "port": 16379,
        "loglevel": "notice",
        "removeOld": true
      }
    }

- `nodejs.use_iojs`: use io.js instead of node.js
- `nodejs.version`: change io.js/node.js version (keep the `v` letter in front)
- `nodejs.removeOld`: delete previous installed io.js/node.js binarys
- `redis.version`: change redis version
- `redis.port`: port used by redis (Refer to [here](https://openshift.redhat.com/community/kb/kb-e1038-i-cant-bind-to-a-port))
- `redis.loglevel`: `debug`, `verbose`, `notice`, or `warning`
- `redis.removeOld`: delete previous installed redis binarys

`commit` and then `push` to reflect the changes to the OpenShift app.

**Note that node.js `v0.6.x` won't work with this method.**

Use Redis in Node.js
--------------------

Environment variables `REDIS_IP` (which is based on `$OPENSHIFT_DIY_IP` when app is started) and `REDIS_PORT` are defined. Simply connect to Redis server with `REDIS_IP` and `REDIS_PORT`. For example, using [node-redis](https://github.com/mranney/node_redis)

    var redis = require("redis");
    var redisClient = redis.createClient(process.env.REDIS_PORT, process.env.REDIS_IP);

