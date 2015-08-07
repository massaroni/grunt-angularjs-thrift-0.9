Thrift 0.9.x Angular Code Generator
=============================
Make Thrift rpc calls to your Thrift http server, from javascript in a browser, in a website or cordova/phonegap app.  This is a grunt module that compiles Thrift IDL into commonjs javascript, for use with browserify or webpack.  The generated angularjs module generates an angular service for each Thrift service, at runtime.  For Thrift 1.0 support, see [grunt-angularjs-thrift](https://github.com/massaroni/grunt-angularjs-thrift).

Dependencies
============
Install a thrift 0.9.x compiler on your machine: [Apache Thrift](https://thrift.apache.org/)

Installation
============
npm install grunt-angularjs-thrift-0.9

Example Gruntfile.js Configuration
==================================

```javascript

    module.exports = function (grunt) {
      // load the grunt plugin
      require('grunt-angularjs-thrift-0.9/compiler/grunt-thrift-generator')(grunt);

      // configure the grunt plugin
      grunt.initConfig({
        thriftcompiler: {
          deluxe: {
            files: [
              {
                src: 'app/src/thrift/**/*.thrift',
                dest: 'app/src/generated'
              }
            ],
            thriftSrcRoot: 'app/src/thrift/', // root directory of the above src path
            thriftBin: 'thrift', // this is your Thrift 0.9.x compiler

            // (optional) with this property, it will generate angular services (at runtime, so you won't find any generated js files with angular services)
            angularServices: {
              rootEndpointUrl: '/thrift-api/v1/', // root uri path of all your thrift service endpoints (domain name is configured elsewhere)
              thriftRetryHandler: { // user-provided error handler, for thrift and http errors (see the example below)
                angularModule: 'thrift-retry',
                angularService: 'CustomThriftRetryHandler'
              }
            }
          },
          compileOnly: {
            files: [
              {
                src: 'app/src/thrift/**/*.thrift',
                dest: 'app/src/generated'
              }
            ],
            thriftSrcRoot: 'app/src/thrift/',
            thriftBin: 'thrift'
          }
        }
      });

    };
```

Example: User-Provided Thrift Error Handler
==========================================

For example, always retry the same exact thrift call once, for whatever reason.

```javascript

    // app/src/thrift/thrift-retry-service.js

    require('angular');

    var thriftRetryModule = angular.module('thrift-retry', []);
    thriftRetryModule.service('CustomThriftRetryHandler', [function CustomThriftRetryHandler() {

      this.handleError = function handleError(thriftClient, reason, state) {
        if (state.callCount > 2) {
          thriftClient.reject(reason);
        } else {
          thriftClient.retry();
        }
      };

    }]);

```

Example: Load Thrift In Your App
================================

```javascript

    // app/src/thrift/thrift-services.js

    // the generated thrift js module returns a configuration object with instructions for generating angular services
    var generated = require('../generated/compiled_thrift');

    // the generated angular thrift services will depend on your custom angular service, for error handling
    require('./thrift-retry-service');

    // generate the angular thrift services, in the ngThriftServices module
    require('grunt-angularjs-thrift-0.9/runtime/ng-thrift-services')(generated);

    // your module must depend on the ngThriftServices module, provided by grunt-angularjs-thrift-0.9
    angular.module('my-module', ['ngThriftServices']);

    // the naming convention for angular thrift services is: [Thrift][your service name]. So, a thrift service named
    // SomethingService would have an angular thrift service name ThriftSomethingService
    angular.module('my-module').service('MyService', ['ThriftSomethingService', function (ThriftSomethingService) {
      ThriftSomethingService.callRoomService().then(function (response) {
        console.log('response from server:', response);
      }, function (reason) {
        console.log('thrift rpc call failed:', reason);
      });
    }]);

```

```idl

    // app/src/thrift/services.thrift

    service SomethingService {
     string callRoomService()
    }

```

TODO
====

1) more documentation
2) example project
