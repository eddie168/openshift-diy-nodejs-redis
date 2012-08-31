openshift-diy-nodejs-redis
==========================

Thanks for the great work by [razorinc](https://github.com/razorinc/redis-openshift-example) and [creationix](https://github.com/creationix/nvm/), this repo let you test Node.js (v0.8 and above) with Redis in a OpenShift DIY application. For Node.js, it will first check for pre-compiled linux version, then compile from source if not found. Redis will be compiled from source code. Subsequent `push` will skip source code compile unless a new version is specified.

[node-supervisor](https://github.com/isaacs/node-supervisor) is used to automatically restart the node.js app if somehow crashed.

Usage
-----

Create an DIY app

    rhc app create -t diy-0.1 -a yourapp

Add this repository

    cd yourapp
    git remote add nodejsRedis -m master git://github.com/eddie168/openshift-diy-nodejs-redis.git
    git pull -s recursive -X theirs nodejsRedis master

Then push the repo to openshift

    git push

If pre-compiled node.js binary is not available, first push will take a while to finish.

You can specify the node.js script to start with in `package.json` as described [here](https://openshift.redhat.com/community/kb/kb-e1048-how-can-i-run-my-own-nodejs-script).

Check the end of the message for Node.js and Redis version:

    remote: Starting application...
    remote: Node Version:
    remote: { http_parser: '1.0',
    remote:   node: '0.8.8',
    remote:   v8: '3.11.10.19',
    remote:   ares: '1.7.5-DEV',
    remote:   uv: '0.8',
    remote:   zlib: '1.2.3',
    remote:   openssl: '1.0.0f' }
    remote: Redis Version:
    remote: redis-cli 2.4.17
    remote: nohup supervisor server.js >/var/lib/stickshift/xxxxxxxxxxxxxxxxxx/yourapp/logs/server.log 2>&1 &
    remote: Done

In this case it is node `v0.8.8` and redis `2.4.17`

You can find node.js app's log at `$OPENSHIFT_LOG_DIR/server.log`. Subsequent `push` will rename the log file with a time stamp before overwritten. The same goes to Redis log file and can be found at `$OPENSHIFT_LOG_DIR/redis.log`. 
You should be able to see these log files with `rhc app tail -a yourapp`.

Now open your openshift app in browser and you should see the standard openshift sample page. Enjoy!!

Settings
--------

Edit `config_diy.json`

    {
      "nodejs": {
        "version": "v0.8.8",
        "removeOld": false
      },
      "redis": {
        "version": "2.4.17",
        "port": 16379,
        "loglevel": "notice",
        "removeOld": false
      }
    }

- `nodejs.version`: change node.js version (keep the `v` letter in front)
- `nodejs.removeOld`: delete previous installed node.js binarys
- `redis.version`: change redis version
- `redis.port`: port used by redis (Refer to [here](https://openshift.redhat.com/community/kb/kb-e1038-i-cant-bind-to-a-port))
- `redis.loglevel`: `debug`, `verbose`, `notice`, or `warning`
- `redis.removeOld`: delete previous installed redis binarys

`commit` and then `push` to reflect the changes to the OpenShift app.

**Note that `v0.6.x` won't work with this method.**

Use Redis in Node.js
--------------------

An environment variable `REDIS_PORT` is defined. Simply connect to Redis server with `REDIS_PORT`. For example, using [node-redis](https://github.com/mranney/node_redis)

    var redis = require("redis");
    var redisClient = redis.createClient(process.env.REDIS_PORT);

