<!--
master: title-page
custom:
  0:
    width: 50%
    margin-left: -227px
    display: inline
    zoom: 1.4
    margin-top: 20px
    flex: 0 0 55%
  1:  
    margin-right: -510px
    zoom: 0.75
    margin-top: -435px
    flex: 0 0 70%
  2:
    margin-top: 60px
  3:
    margin-bottom: -30px
-->

# Ten Tips for Triumphant Noding</h1>
![](images/brian.jpg)

@davidmarkclem

<span suffix="@nearform.com">david.clements</span>

---
<!--
master: bullets
-->

# The 10 Tips

1. Develop Debugging Techniques
2. Avail and Beware of the Ecosystem
3. Know when (not) to Throw
4. Reproduce Core Callback Signatures
5. Use Streams
6. Break Out Blockers
7. Deprioritize Synchronous Code Optimizations
8. Use and Create Small Single-Purpose Modules
9. Prepare for Scale with Micro-Services
10. Expect to Fail, Recover Quickly

---
<!--
master: section-title
-->

# 1. Develop Debugging Techniques

---
<!--
master: command-bullets
custom:
  2:
    font-size: 1.25em
-->

# node debug

```sh
$ node debug myApp.js
```

* Node has a built in debugger, similar to LLDB
* It's not very fun

---
<!--
master: command-x2-bullets
-->

# node inspector

```sh
$ npm -g i node-inspector
```

```sh
$ node-debug myApp.js
```

* Hooks up Node debugging with devtools
* Essentially we can debug a node process like we debug a web app
* Doesn't include advanced dev tools features like profiling
  * However webkit-devtols-agents covers that (but not debug)

---
<!--
master: command-bullets
custom:
  1:
    font-size: 0.7em
  2:
    li ul li:
      margin-top: 6px
      margin-bottom: 6px

-->

# NODE_DEBUG

```sh
$ NODE_DEBUG=http,net node server.js
```

* Core API debug information can be enabled with the `NODE_DEBUG` environment variable
* Supported values are 
  * fs
  * http
  * net
  * tls
  * module
  * timers

---
<!--
master: code
-->

# debug module
```javascript
var debug = require('debug');
var log = debug('myThing:log');
var info = debug('myThing:info');
var error = debug('myThing:error');

log('Squadron 40, DIVE!!!');
info('Gordon\'s ALIVE!');
error('Impetuous boy!');

```


---
<!--
master: command-x2-bullets
custom: 
  3: 
    font-size: 0.95em
-->

# debug module

```
$ npm install debug
```
```
$ DEBUG=myThing node app.js
```


* The `debug` module creates log output on a per namespace basis
* Log functions are no-ops unless the `DEBUG` environment variable enables a log
* The DEBUG variable can contain wildcards
  * eg. `DEBUG=*` will turn all debug on
  * `DEBUG=myThing:*` will turn on any debug namespaces prefixed with `myThing:`

---
<!--
master: command-bullets
custom:
  1:
    font-size: 0.8em
-->

# Deeper Stack Traces

```sh
$ node --stack_trace_limit=200 -e "function x()  { --x.c ? x() : u } x.c=100; x()"
```

* Default stack trace limit is ten
  * Presumably for performance reasons
* Often times, there is more than 10 levels to a stack trace
  * Particularly when relying on modules that rely on modules etc.
* The `--stack_trace_limit` flag can be set for deeper stack trace output

---

<!--
master: command-bullets
custom:
  1:
    font-size: 0.75em
-->

# Deeper Stack Traces

```
$ node -e "Error.stackTraceLimit=200;  function x()  { --x.c ? x() : u } x.c=100; x();"
```

* Stack trace limit can also be set in-process, by setting the `stackTraceLimit` property on the error object
* This can be preferred in some cases, for instance when executing a file directly, flags don't get passed to the node executable
---
<!--
master: command-bullets
custom:
  1:
    font-size: 0.8em
  2: 
    font-size: 0.9em
-->

# Asynchronous Tracing

```
$ node -e "require('longjohn'); function x()  { --x.c ? setImmediate(x) : u } x.c=10; x()"
```

* A stack trace is relevant to a particular event queue
* Cause and effect of asynchronous operations happen in different iterations of the event loop, and thus exists in seperate event queues
* The `longjohn` module provides asynchronous traces, so the relationship between calls can be seen accross stacks
* Definitley remove `longjohn` before production, async traces require
a lot of logic monitoring and state management

---
<!--
master: bullets
custom:
  1:
    font-size: 0.9em
-->

# Function Deanonymizing

* Anonymous functions lead to puzzling stack traces
* However, a lot of the time a function is used as a lambda (a small item of work, or an iteratable function), or a continuation (a callback)
* Lambda's and continuations are hard to name, 
  * `setTimeout(function whatDoICallThis() {}, 1000);`
* Also, people are lazy and have bad habits
* So we're generally stuck at some point or another with stack traces that say
&lt;anonymous function&gt;
  * ... or are we?
  
---
<!--
master: command-bullets
-->

# Function Deanonymizing

```
npm install decofun 
```

* The decofun module parses code and names any anonymous functions according to their context
* There's a demo server, pass it the address of a JavaScript file and it'll name the anonymous functions
* http://decofun.herokuapp.com/?addr=http://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.js
* http://decofun.herokuapp.com/?addr=http://ajax.googleapis.com/ajax/libs/angularjs/1.3.10/angular.js

---

<!--
master: section-title
-->

# 2. Avail and Beware of the Ecosystem

---

<!--
master: section-title
-->

![](/images/brian-roaring.jpg)
# Avail

---
<!--
master: command-bullets
custom:
  2:
    font-size: 1.25em
-->

# Avail

```
$ npm install what-you-need
```

* There are over 100,000 modules on NPM
* Don't write any code before checking NPM for a solution
* If it's a part solution, that could at least be a springboard

---

<!--
master: section-title
-->

![](/images/brian-roaring.jpg)
# Beware

---
<!--
master: bullets
custom:
  1:
    font-size: 1.25em
-->

# Beware

* Anyone can publish anything to NPM
* Learn who to trust
* How many other modules depend on this module?
* &lt;sarcasm&gt;Does it have a shiny website?&lt;/sarcasm&gt;
* http://nodezoo.com pulls a bunch of metrics from various sources to provide a confidence level for a module

---
<!--
master: command-bullets
custom:
  1: 
    font-size: 0.8em
  2:
    font-size: 1.2em
-->

# Global modules

```
sudo npm -g install are-you-crazy
```

* On OSX and Linux, by default NPM needs root permissions to install global modules
* This is because global modules are installed in `/usr/local/lib/node_modules` and the executables are linked to from `/usr/local/bin`

---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# Global modules

* Installing a module as root should be thought about long and hard  
* It's only occassionally neccessary
* the `n` module manages node versions, it needs root perms to install different version of node
* phantomjs (which can be installed with npm) for reasons unknown requires
root permissions

---


<!--
master: multiline-command-bullets
custom:
  1:
    font-size: .8em
  2:
    font-size: 1.2em
-->

# Global modules

<pre><code>$ mkdir ~/npm-global
$ npm set prefix ~/npm-global
$ echo "export PATH=~/npm-global/
&nbsp; bin:$PATH" >> ~/.profile
$ source ~/.profile</code></pre>

* Depending on the system, it may be `~/.bash_profile` instead of `~/.profile`
* Where a system has multiple users, each user manages their own global installs

---
<!--
master: multiline-command-bullets

-->

# Global modules

* In cases where a module does need to run with with sudo localized globals will fail because the '~/npm-global' isn't in the root environments path (nor should it be)
* These edge cases can be managed by linking from /usr/local/bin to ~/npm-globals/bin
  * e.g. `ln -s ~/npm-global/bin/n /usr/local/bin/n`

---
<!--
master: section-title
-->

# 3. Know when (not) to 
# (╯°□°）╯︵ʍoɹɥʇ

---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# When to throw

* Throw to flag developer error
  * E.g. Someone is using your modules API incorrectly
* Throw from command line or build tools if neccessary
  * Kind of equivalent to developer error

---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# When not to throw

* For services, never throw after initialization time
* Never throw because of bad config
  * notify and revert to defaults instead
* Never throw in an asynchronous context

---
<!--
master: bullets
custom:
  1:
    font-size: 1.4em
-->

# Alternatives to throwing

* Debug log output
* Devops alert systems
* Kill process and restart

---
<!--
master: bullets
custom:
  1:
    font-size: 1.4em
-->

# Why to not try-catch

* It isn't asynchronous
* It's a V8 optimizations buster
  * If Try-Catch *is* used, place it into its own function

---
<!--
master: code-bullets
-->

# When try-catch may be useful

```javascript
try { 
  JSON.parse(userJson); 
} catch (e) { 
  warn('Bad user input');
}
```

* When parsing untrusted JSON input
* Possibly, as a temporary measure
  * to see errors impact on logic flow
  * As a temporary fix to get a happy path

---
<!--
master: code
-->

# When try-catch may be useful

```javascript
var letSupport;
try { 
  Function('let foo = 1;')(); 
  letSupport = true; 
} catch (e) { }

var mod = letSupport ? 
  require('es6mod') : 
  require('transpiledMod');

```

---
<!--
master: bullets
custom:
  1:
    font-size: 1.4em
-->

# When try-catch may be useful

* Sometimes to identify language support
  * More of a browser side technique
  * However, with ES6 round the corner may it
    be neccessary to support multiple Node versions

---
<!--
master: section-title
-->

# 4. Reproduce Core Callback Signatures

---
<!--
master: code
-->

# Callbacks

```javascript
var addr = 'http://get.me/json';
function getData(url, cb) {
  ajax(url, function (res) {
    cb(JSON.parse(res.data))
  });
}
getData(addr, function (data) {
    console.log(data);
});
```

---
<!--
  master: bullets
  custom: 
    1: 
      font-size: 1.1em
-->

# Callbacks

* When a function passed in as an argument to another function is
  expected to be called upon completion of an operation, the 
  passed-in function is known as a callback
* Data is sent through the callback, instead of being returned 
  from the function
* This is the only way to retrieve data asynchronously
* Callbacks are **the** fundamental unit of asynchronous operation,
  all other abstractions (promises, generators, streams, event emitters),
  are built using callbacks

---
<!--
master: code
custom: 
  1:
    zoom: 0.68
    margin-top: 0
-->

# Errbacks

```javascript
function concatStrToFile(str, path, cb) {
  fs.exists(path, function(exist) {
    if (!exists) { 
      return cb(Error(path + ' doesn\'t exist')); 
    }
    fs.readFile(path, function (err, data) {
      if (err) { return cb(err); }
      cb(null, str + data);
    })  
  })
}
concatStrToFile('foo', '/bar/baz.txt', 
  function (err, result) {
    if (err) { return console.error(err);  }
    console.log(result);
  })
```
---
<!--
master: bullets
-->

# Errbacks

* Node core uses _errback_ style callbacks
* Errbacks are error first, this helps to maintain a focus on error handling
  * Errors must be explitly recognized before data
* Convention is to handle errors first, and return early from the function
  * This helps to prevent right-creep
* Both errors and data are easily propagated from an API level to 
  an app level by passing them through the callback


---
<!--
master: section-title
-->

# 5. Use Streams

---
<!--
master: code
custom: 
  1:
    zoom: 0.75
    margin-top: 0
-->

# Avoid en bloc processing

```javascript
var request = require('request');

request('http://registry.npmjs.org/-/all', 
  function (err, res) {
     var recs = JSON.parse(res.body);
    var names = '';

    Object.keys(recs).forEach(function (key) {
     names += recs[key].name + '\n';
    });

    console.log(names);

});
```
---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# Avoid en bloc processing

* Buffering all data before processing blocks a processing pipeline
* JavaScript was never intended for use with huge memory chunks
* Processing a large chunk of data will block the event loop
* Any failure means no data is processed at all
* This could never work with a continuous realtime stream of data, 
  we would never have all data


---
<!--
master: code
custom: 
  0:
    zoom: 0.85
-->

# Use streams for incremental processing

```javascript
var request = require('request');
var JSONStream = require('JSONStream');

request('http://registry.npmjs.org/-/all')
  .pipe(JSONStream.parse('*.name', 
    function (name) { 
        return name + '\n'; 
    }))
  .pipe(process.stdout)

```
---
<!--
master: bullets
custom: 
  0:
    zoom: 0.85
  1:
    font-size: 0.95em
-->

# Use streams for incremental processing

* Streams allow for asynchronous processing
* They're essentially implementation of asynchronous 
  functional programming
  * Each stream can be though of as a function
  * The pipe method is the equivalent of passing the
    return value from one function into the next
* Streams control the memory consumption of a process
* Stream processing prevents the event loop from getting blocked
* They're perfectly congruent with realtime processing

---
<!--
master: code-bullets
custom: 
  1: 
    font-size: 0.75em
  2:
    font-size: 1.2em
-->

# Push streams

```javascript
stream.on('data', function (chunk) {
  //process chunk
});
```

* The first Node streams were push streams
* Listen to a `data` event on the stream
* If a stream receiver gets overwhelmed it 
has to pause the sending stream
  * This is known as handling back pressure

---
<!--
master: code-bullets
custom:
  1:
    font-size: 0.75em
-->

# Pull Streams
```javascript
stream.on('readable', function () {
  var chunk;
  while(chunk = stream.read()) {
    //process chunk
  }
});
```
* Streams 2 converted streams to pull, you ask
  for the next piece of data
  * In the pull case, backpressure doesn't need to
    be managed, you simply stop pulling. 
* But retains backwards compatibility with a "flow mode"
  which turns them back into pull streams

---
<!--
master: command-bullets
custom:
  2: 
    font-size: 0.9em
-->

# The pull-stream module

```sh
$ npm install pull-stream
```

* The `pull-stream` module is a minimal stream implementation
* Uses the same pipe paradigm
  * When a writable stream is connected to a readable stream
    (or a pipeline that begins with a readable stream), it 
    recursively pulls from the stream
* No data event, no read method
  * This is preferred since we should be thinking in terms of pipes
* No need for a readable event, it's just callback recursion
* Great for browser side usage, theres a 100kb
  difference between Node streams and pull-stream

---
<!--
master: command-bullets
custom: 
  1: 
    font-size: 0.84em
-->

# websocket-pull-stream

```sh
npm install websocket-pull-stream
```

* [websocket-pull-stream](http://github.com/davidmarkclements/websocket-pull-stream) supplies a transport for streaming from a Node process into the browser
* around 200kb lighter than `websocket-stream`
* has a custom built multiplexer that allows for multiple two-way channels
* Pipe Node streams on the server into `websocket-pull-stream`, get the data
on the client as pull-stream streams.
* Seamlessly pipes object streams into the client

---
<!--
master: section-title
-->

# 6. Break Out Blockers

---
<!--
master: bullets
-->
# The Event Loop

* JavaScript (and thus Node) is single threaded
* The thread is a loop which processes an event queue
* Each item in an event queue is added to that queue in
  a previous loop
  * for instance a callback is appended to an event
    queue when the initial operation resolves

---
<!--
master: bullets
-->
# Blocking Code

* Code that takes a long time to execute is said to _block_
  the event loop
* Simple self-DoS: `while(1);`
* Examples of blocking code
  * A hashing function
  * JSON.parse a huge string
  * Any of the *Sync API's
* Always prefer non-blocking logic in an asynchronous environment
  * Only use Sync apis during initialization

---
<!--
master: bullets
-->
# Use separate Processes

* Break blocking code out into seperate processes
* Communicate asynchronously with the process
* This new process is now a service for your app

---
<!--
master: section-title
-->

# 7. Deprioritize Synchronous Code Optimizations

---
<!--
master: image-no-title
-->

![](images/brian-stunned.jpg)

---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# Sync vs Async

* Asynchronous operations take much longer 
than synchronous logic
* The asynchonrous part is the bottleneck
* Focus on optimizing async co-dependencies
  * Run independant operations in parallel
  * Choose endpoint systems that can combine operations
* Synchronous logic that takes a really long time
  should be broken into seperate processes

---
<!--
master: bullets
custom:
  1:
    font-size: .9em
-->

# Cost-benefit

* There is a place for optimizing synchronous code,
  but generally the benefit of micro-optimizing needs
  to be magnified to justify developer time
* The lo-dash library is well written, optimal synchronous logic
  * Benefits are magnified because it's open source and
    has a large audience
  * Also because large amounts of data could be passed
    through
* Optimizing hot code could be worth it
  * However, V8 tends to optimize hot code very well, 
    any optimizing needs to be done in tangent with the
    V8 engine
    * Tip: `--trace-inlining` and `--trace-deopt`
    * Tip: use small functions

---
<!--
master: section-title
-->

# 8. Use and Create Small Single-Purpose Modules

---
<!--
master: bullets
-->

# Why modularize

* Isolate functionality into independant components
* Store components as seperate packages, with their own tests 
and documentation
* The up-front work of breaking logic into seperate fully-fledged
modules is worth the reduction in cognitive overhead of managing
the same logic when coupled to a large application context
* The singularity, clear boundaries, and independance of a module
undercuts overall system entropy
  * e.g. modularity brings clear order to potential disorder

---
<!--
master: bullets
-->

# Results of modularization

* Application logic should be sparse
  * Application code is what's left over when everything has been
    modularized
  * Application logic can also be stored in seperate lib modules and required into the main file
* Modules allow code to be shared, thus maginifying its impact
  * Publish to NPM so everyone benefits
  * Or.. Publish to a local npm repo so the organization benefits
    * Check out sinopia

---
<!--
master: section-title
-->

# 9. Prepare for Scale with Micro-Services

---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# Monoliths

* A monolithic application is one where all (or most) code is run in the same process
* This is potentially fine for certain scenarios - e.g. software products
* When software is a service, monolithic architectures can be problematic
  * If the monolith dies, everything dies
  * Monoliths have a glass ceiling when it comes to scaling

---
<!--
master: bullets
custom:
  1:
    font-size: 1.2em
-->

# Monoliths

* JavaScript is not designed for large scale monolithic applications
  * Large class heirarchies just don't fit well in JavaScript world
  * Throwing kills the process, the native run time throws, which means
    a process can die very easily

---
<!--
master: bullets
custom:
  1: 
    font-size: 0.9em
-->

# Micro-services

* When breaking out some blocking code, we essentially create a micro-service
* A whole codebase can be broken up into pieces that communicate together
* Communication is explicitly asynchronous
* The pieces should be small and single-purposed
* Each small piece represents a business area
* This level of granularity allows for much greater fault tolerance
  * Only a fraction of the entire deployment is degraded upon failure, everything else stays alive

---
<!--
master: bullets
-->

# Communicating between services

* STDIO
  * `process.stdout` / `process.stdin`
* IPC
  * child_process `send` method and `message` events
* TCP or UDP
  * the `net` or `dgram` modules
* HTTP

---
<!--
master: bullets
-->

# Communicating between services

* PubSub
  * e.g. redis
* Message Queues
  * e.g. rabbitmq

---
<!--
master: bullets
-->

# Seneca

* Seneca abstracts away the details of implementing micro-services,
  allowing developers to focus purely on implementing business logic
* Define actions and commands, in the same process or accross processes
* Tell seneca what transport to use
* Now you're doing micro-services

---
<!--
master: bullets
-->

# Containers

* A container is an isolated execution environment
* VM's are one type of container, but they're large and slow
* virtualized containers can be thought of as an advanced form
of chroot, where isolation is applied beyond the filesystem to
areas like networking, memory access and processor usage
  * This essentially allows a complete OS to run within a container

---
<!--
master: bullets
-->

# Containers

* Virtual Containers are much faster than VM's
* Docker provides easy management of virtualized containers, 
* Distribution of containers is also easy with docker, just send
a container build recipe file, known as a Dockerfile
* Putting each micro-service into a container ensures both
developers and sys admins have the exact same environments
for each micro-service

---
<!--
master: image-no-title
-->

![](images/brian-super-excited.gif)

---
<!--
master: section-title
-->

# 10. Expect to Fail, Recover Quickly

---
<!--
master: bullets
custom: 
  1: 
    font-size: 1.2em
-->

# Expect to fail

* Whether running one process or many, expect to fail
* Expecting to fail leads to robust, high resilience deployments
* Expectation of failure makes monitoring, notifications and fail over non-optional
* Using a chaos monkey embeds failure expectancy into a teams culture


---
<!--
master: bullets
-->

# Recover Quickly

* forever
  * Node based tool that keeps a process alive
* pm2
  * Another keep-alive tool with additional features like load balancing
* upstart
  * a kernel-level tool bundled with certain Linux distros (like Ubuntu)
  * higher quality monitoring, in the unlikely event that upstart fails it causes a kernel panic and the whole server must restart

---
<!--
master: bullets
-->

# Recover Quickly

* uptime robot
  * free third-party pinging service
  * notifies when service goes down
  * great for personal projects
* nscale
  * Entire deployment monitoring
    * Can monitors VMs, containers and remote servers (AMI's etc)
  * Can automatically fix a deployment
    * eg if a process goes down nscale can both detect and redeploy
      a service, VM, machine, etc

---
<!--
master: bullets
custom: 
  1: 
    font-size: 1.5em
-->

# Avoid Downtime

* Use failover to avoid down time
* Spin up multiple instances and load balance
* nginx works well as a reverse proxy

---
<!--
master: title-page
custom:
  1: 
    zoom: 0.7
    margin-top: .5em
    margin-bottom: 1em
-->

# FIN

![](images/brian-thumbs-up.jpg)

David Mark Clements

@davidmarkclem

<span suffix="@nearform.com">david.clements</span>

---

<!--
master: video
-->
# Bonus

<iframe width="420" height="315" src="//www.youtube.com/embed/wQAW61O5Me8" frameborder="0" allowfullscreen></iframe>


<style>[suffix]:after {display:inline;content:attr(suffix)}</style>
