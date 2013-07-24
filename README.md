# POD - git push deploy for Node.js [![Build Status](https://travis-ci.org/yyx990803/pod.png?branch=master)](https://travis-ci.org/yyx990803/pod)

Core API JSCoverage: **94.57%**

Pod simplifies the workflow of setting up, updating and managing multiple Node.js apps on a single Linux server. Perfect for experimenting with Node stuff on a VPS. It is built upon git hooks and [pm2](https://github.com/Unitech/pm2).

It doesn't manage DNS routing for you (personally I'm doing that in Nginx) but you can use pod to run a [node-http-proxy](https://github.com/nodejitsu/node-http-proxy) server on port 80 that routes incoming requests to other apps.

## Prerequisites

- Node >= 0.8.x
- git
- properly set up ssh so you can push to a repo on the VPS via ssh

## Example Workflow

**On the server:**

``` bash
$ pod create myapp
# will print path to repo and working tree
```

**On your local machine:**

``` bash
$ git clone ssh://your-server/pod_dir/myapp.git
# hack hack hack, commit
$ git push
```

**Or use an existing local repo:**

``` bash
$ git remote add deploy ssh://your-server/pod_dir/myapp.git
$ git push deploy master
```

That's it! App should be automatically running after the push. For later pushes, app process will be restarted.  

## Installation

``` bash
$ [sudo] npm install -g pod
```

To make pod auto start all managed apps on system startup, you might also want to write a simple [upstart](http://upstart.ubuntu.com) script that contains something like this:

``` bash
# /etc/init/pod.conf
start on startup
exec sudo -u <username> /path/to/node /path/to/pod startall
```

The first time you run `pod` it will ask you where you want to put your stuff. The structure of the given directory will look like this:

``` bash
.
├── repos # holds the bare .git repos
│   └── example.git
└── apps # holds the working copies
    └── example
        ├──app.js
        └──.podhook
```

## CLI Usage

```

  Usage: pod [command]

  Commands:

    create <app>            Create a new app
    rm <app>                Delete an app
    start <app>             Start an app monitored by pm2
    stop <app>              Stop an app
    restart <app>           Restart an app, start if not already running
    list                    List apps and status
    startall                Start all apps not already running
    stopall                 Stop all apps
    restartall              Restart all running apps
    edit <app>              Edit the app's post-receive hook
    config                  Edit config file
    help                    You are reading it right now

```

## Config

Example Config:

``` js
{
    "root": "/srv",
    "nodeEnv": "development",
    "defaultScript": "app.js",
    "editor": "vi",
    "apps": {
        "example": {
            "nodeEnv": "production", // passed to the app as process.env.NODE_ENV
            "port": 8080, // passed to the app as process.env.PORT
            "instances": 2 // any valid pm2 config here gets passed to pm2
        }
    }
}
```

## Logging & Monitoring

Since pod uses pm2 under the hood, logging is delegated to `pm2`. It's recommended to link `path/to/pod/node_modules/pm2/bin/pm2` to your `/usr/local/bin` so that you can use `pm2 monit` and `pm2 logs` to analyze more detailed app info. Note that all pod commands only concerns apps present in pod's config file, so it's fine if you use pm2 separately to run additional processes.

## Custom post-receive hook

You can edit the post-receive script of an app using `pod edit <appname>` to customize the actions after a git push.

Or, if you prefer to include the hook with the repo, just place a `.podhook` file in your app, which can contain shell scripts that will be executed after push.

## Important changes in 0.4.0

- switched the underlying monitor library from forever to pm2. pm2 is included as a dependency so you can simply link its executable for logging and monitoring. (see below)

- the config is now a single json file `.podrc` instead of a folder. Also the config fields have changed a bit, see *Config* section below for details. You might need to manually migrate the old app configs over. After that you can delete the old `.podconfig`.

- included unit tests for core functionalities (`npm test`)