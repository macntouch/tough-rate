CallServer
==========

TODO: add Node.js clustering

    module.exports = class CallServer
      constructor: (@port,@options) ->
        options.respond ?= true
        router = options.router ? new Router options
        @server = FS.server CallHandler router, options
        @server.listen port

      stop: ->
        new Promise (resolve,reject) =>
          try
            @server.close resolve
            delete @server
          catch exception
            reject exception

Toolbox
=======

    pkg = require './package.json'
    FS = require 'esl'
    Promise = require 'bluebird'
    Router = require './router'
    CallHandler = require './call_handler'