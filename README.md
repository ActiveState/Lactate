# Lactate

Very simple static file handler, with a few electives.

`npm install lactate`

`npm test lactate`

## Benchmark

[Preliminary benchmarks](https://github.com/Weltschmerz/Lactate/blob/master/benchmark/results.md) show that Lactate has a significant advantage over  most worthy competitors on the [node modules wiki](https://github.com/joyent/node/wiki/Modules#wiki-web-frameworks-static)

*See /benchmarks for details*

## Example

Just pass three arguments to the serve function `path` [optional], `request`, `response`. Lactate will stream your file to the client in the most efficient way, by piping: readFile > gZip > response.

```js

var express = require('express')
var app = express.createServer()

var Lactate = require('lactate')
var lactate = Lactate.Lactate()

lactate.set({
  root:process.cwd(),
  expires:'one day and 12 minutes',
  debug:true
})

app.get('/', function(req, res) {
  return lactate.serve('pages/land.html', req, res)
})

var files = Lactate.dir('files', {
  root:'files',
  public:'files',
  expires:'ten years'
}).toMiddleware()

app.get('/files/*', files)

app.listen(8080)

```

##The varieties of Lactate experience

In the general case, the `Lactate` method returns an object with the methods `serve` `set` and `get`, importantly. However, there are more convenient methods exported by Lactate. They follow.

###Serving an individual file

To serve an individual file, use the `file` method.

```js
  var Lactate = require('lactate')

  app.get('*', function(req, res) {
    return Lactate.file('images/somn.jpg', req, res)
  })
```

###Namespacing a directory

The `dir` method allows you to namespace a directory, for convenience.

```js
var Lactate = require('lactate')
var images = Lactate.dir('images', {expires:'one day'})

app.get('/images/:image', function(req, res) {
  return images.serve(req.params.image, req, res)
})
```

###Middleware

For maximum convenience, you may use the `toMiddleware` method on directories.

```js
var Lactate = require('lactate')

var images = Lactate.dir('images', {
  expires:'one day'
}).toMiddleware()

app.use(images) //That's it!
```

You may also pass additional options to the `toMiddleware` function.

```js
var images = Lactate.dir('images', {
  expires:'one day'
})

var middleware = images.toMiddleware({
  public:'images'
})

app.use(middleware)
```

##Options

Options can be passed to the initialization function or using the `set` method.

### Setting options

```js

//Passing to initialization function
var lactate = require('lactate').Lactate({
  expires:'two days'
})

//Set method
lactate.set('expires', null)

//Either function accepts (key, value) or an object.

```

### Options available

+ `root` **string**

Local directory from which to serve files. By default, the current working directory.

+ `public` **string**

Public directory exposed to clients. If set, only requests from /*directory* will complete.

+ `cache` **boolean**

Keep files in-memory. Enabled by default, and no great reason to disable.

+ `expires` **number** or **string**

Pass this function a number (of seconds) or a string and appropriate headers will be set for client-side caching. Lactate comes with expiration defaults, such as 'two days' or '5 years and sixteen days' See [Expire](https://github.com/Weltschmerz/Expire) for details.

```code
lactate.set('expires', 87500)
//87500 seconds
lactate.set('expires', 'two days')
//172800 seconds
lactate.set'expires', 'five weeks and one minute and ten seconds')
//3024070 seconds
lactate.set('expires', 'one year and 2 months and seven weeks and 16 seconds')
//41050028 seconds

```

+ `debug` **boolean** (*optional*) **number** (*optional*) **function** (*optional*) 

Debugging in Lactate is level-based (*bases: `0`, `1`*). Level `0` logs completed request information, status codes, etc.. Level `1` provides more details along the service. You may override the default debug function (*console.log*) with your own.

```js

var lactate = require('lactate')({
  debug:true // Will use console.log to debug all events
})

lactate.set('debug', 0, function(level, msg, path, statusCode) {
  /* 
    Captures all level 0 events

    Note however that statusCode arguments are only
    given for level 0 listeners
  */
})

lactate.set('debug', 1, console.log)
lactate.set({debug:false})

```

##TODO

+ ~~Express middleware~~
+ ~~Expiration defaults, e.g. 'two days,' 'one month'~~
+ External (Redis) caching ability
