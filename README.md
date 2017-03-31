

# Node.js Best Practices | RisingStack

We get asked about [Node.js best practices][1], tips all the time - so this post intends to clean things up, and summarizes the basics of how we write Node.js at [RisingStack][2].

Some of these Node.js best practices fall under the category of [_Coding style_][3], some deal with _Developer workflow_.

### Coding style

#### Callback convention

Modules should expose an error-first callback interface. 

It should be like this:
    
    
    module.exports = function (dragonName, callback) {  
      // do some stuff here
      var dragon = createDragon(dragonName);
    
      // note, that the first parameter is the error
      // which is null here
      // but if an error occurs, then a new Error
      // should be passed here
      return callback(null, dragon);
    }
    

[Need help with enterprise-grade Node.js Development?   
**Hire the experts of RisingStack!**   
][4]

#### Always check for errors in callbacks

To better understand why this is a must, first start with an example that is **broken** in every possible way, then fix it.
    
    
    // this example is **BROKEN**, we will fix it soon :)
    var fs = require('fs');
    
    function readJSON(filePath, callback) {  
      fs.readFile(filePath, function(err, data) {  
        callback(JSON.parse(data));
      });
    }
    
    readJSON('./package.json', function (err, pkg) { ... }
    

The very first problem with this `readJSON` function, is that it never checks, if an `Error` happened during the execution. You should always check for them.

The improved version:
    
    
    // this example is **STILL BROKEN**, we are fixing it!
    function readJSON(filePath, callback) {  
      fs.readFile(filePath, function(err, data) {
        // here we check, if an error happened
        if (err) {
          // yep, pass the error to the callback
          // remember: error-first callbacks
          callback(err);
        }
    
        // no error, pass a null and the JSON
        callback(null, JSON.parse(data));
      });
    }
    

#### Return on callbacks

One of the problems that still exists in the above example, is that if an `Error` occurs, then the execution will not stop in the `if` statement, but will continue. This can lead to lots of unexpected things. As of a rule of thumb, always return on callbacks.
    
    
    // this example is **STILL BROKEN**, we are fixing it!
    function readJSON(filePath, callback) {  
      fs.readFile(filePath, function(err, data) {
        if (err) {
          return callback(err);
        }
    
        return callback(null, JSON.parse(data));
      });
    }
    

#### Use try-catch in sync code only

Almost there! One more thing we have to take care of is the `JSON.parse`. `JSON.parse` can throw an exception, if it cannot parse the input string to a valid `JSON` format.

As `JSON.parse` will happen synchronously, we can surround it with a `try-catch` block. **Please note, that you can only do this with synchronous codeblocks, but it won't work for callbacks!**
    
    
    // this example **WORKS**! :)
    function readJSON(filePath, callback) {  
      fs.readFile(filePath, function(err, data) {
        var parsedJson;
    
        // Handle error
        if (err) {
           return callback(err);
        }
    
        // Parse JSON
        try {
          parsedJson = JSON.parse(data);
        } catch (exception) {
          return callback(exception);
        }
    
        // Everything is ok
        return callback(null, parsedJson);
      });
    }
    

#### Try to avoid `this` and `new`

Binding to a specific context in Node is not a win, because Node involves passing around lots of callbacks, and heavy use of higher-level functions to manage control flow. Using a functional style will save you a lot of trouble.   
Of course, there are some cases, when prototypes can be more efficient, but if possible, try to avoid them.

#### Create small modules

Do it the unix-way:

&gt; Developers should build a program out of simple parts connected by well defined interfaces, so problems are local, and parts of the program can be replaced in future versions to support new features.

Do not build _Deathstars_ \- keep it simple, a module should do one thing, but that thing well.

#### Use good async patterns

Use [async][5].

#### Error handling

Errors can be divided into two main parts: **operational errors** and **programmer errors**. 

##### Operational errors

Operational errors can happen in well-written applications as well, because they are not bugs, but problems with the system / a remote service, like:

* request timeout
* system is out of memory
* failed to connect to a remote service

##### Handling operational errors

Depending on the type of the operational error, you can do the followings:

* Try to solve the error - if a file is missing, you may need to create one first
* Retry the operation, when dealing with network communication
* Tell the client, that something is not ok - can be used, when handling user inputs
* Crash the process, when the error condition is unlikely to change on its own, like the application cannot read its configuration file

Also, it is true for all the above: **log everything**.

##### Programmer errors

Programmer errors are bugs. This is the thing you can avoid, like:

* called an async function without a callback
* cannot read property of `undefined`

##### Handling programmer errors

Crash immediately - as these errors are bugs, you won't know in which state your application is. A process control system should restart the application when it happens, like: [supervisord][6] or [monit][7]. 

### Workflow tips

#### Start a new project with `npm init`

The `init` command helps you create the application's `package.json` file. It sets some defaults, which can be later modified.

Start writing your fancy new application should begin with:
    
    
    mkdir my-awesome-new-project  
    cd my-awesome-new-project  
    npm init  
    

#### Specify a start and test script

In your `package.json` file you can set scripts under the `scripts` section. By default, `npm init` generates two, `start` and `test`. These can be run with `npm start` and `npm test`. 

Also, as a bonus point: you can define custom scripts here and can be invoked with `npm run-script <script_name>`. 

Note, that NPM will set up `$PATH` to look in `node_modules/.bin` for executables. This helps avoid global installs of NPM modules.

#### Environment variables

Production/staging deployments should be done with environment variables. The most common way to do this is to set the `NODE_ENV` variable to either `production` or `staging`.

Depending on your environment variable, you can load your configuration, with modules like [nconf][8].

Of course, you can use other environment variables in your Node.js applications with `process.env`, which is an object that contains the user environment.

#### Do not reinvent the wheel

Always look for existing solutions first. NPM has a crazy amount of packages, there is a pretty good chance you will find the functionality that you are looking for.

#### Use a [style guide][3]

It is much easier to understand a large codebase, when all the code is written in a consistent style. It should include indent rules, variable naming conventions, best practices and lots of other things.

For a real example, check out [RisingStack][2]'s [Node.js style guide][3].

### Next up

I hope this post will help you succeed with Node.js, and saves you some headaches.

This post will continue with another one dealing with operational tips and best practices.

You can read about deployment tips here: [Continuous Deployment of Node.js Applications][9].

[2]: http://risingstack.com
[3]: https://github.com/RisingStack/node-style-guide
[4]: https://rstck.typeform.com/to/PbubEn?utm<em>source=rsblog&amp;utm</em>medium=in-article
[5]: https://github.com/caolan/async
[6]: http://supervisord.org/
[7]: http://mmonit.com/monit/
[8]: https://github.com/flatiron/nconf
[9]: http://blog.risingstack.com/continuous-deployment-of-node-js-applications/

  </script_name>
