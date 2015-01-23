# Fetching Data

Data is organized into services.
Services are where you define your CRUD operations for a specific resource.
A resource is a unique string that identifies the data (e.g. 'user', 'photo', 'comment').

For example, your application might have a User Service which defines how to create, read, update and delete a user account.

[Fetchr](https://github.com/yahoo/fetchr) manages an application's services and provides an isomorphic interface for calling the services.
Fetchr transparently changes how it calls the services based on environment:
on the server, calls are made directly to the services, while on the client,
calls are executed via XHR to a route that proxies to the individual services.
So, the service code that you write is always executed on the server,
but can be access transparently from actions without any knowledge of whether it's on the server or client.
Fetchr provides an appropriate abstraction so that you can fetch (CRUD) the data needed in your stores using the same exact syntax on server and client side.

## Using The Plugin

[fluxible-plugin-fetchr](https://github.com/yahoo/fluxible-plugin-fetchr) is how we will use Fetchr in our Fluxible applications.

```js
var Fluxible = require('fluxible');
var fetchrPlugin = require('fluxible-plugin-fetchr');
var pluginInstance = fetchrPlugin({
    xhrPath: '/api' // Path for XHR to be served from
});
var app = new Fluxible();

app.plug(pluginInstance);
```


## Creating Your Services

```js
// UserService.js
module.exports = {
    // Name is the resource. Required.
    name: 'user',
    // At least one of the CRUD methods is Required
    read: function(req, resource, params, config, callback) {
        var data = DATABASE.getUser(params.userId);
        callback(null, data);
    },
    // other methods
    // create: function(req, resource, params, body, config, callback) {},
    // update: function(req, resource, params, body, config, callback) {},
    // delete: function(req, resource, params, config, callback) {}
}
```


## Registering Your Services

```js
var userService = require('services/UserService.js');
pluginInstance.registerService(userService);
```

Or if you need to do this from your application without direct access to the plugin

```js
var userService = require('services/UserService.js');
app.getPlugin('FetchrPlugin').registerService(userService);
```


## Exposing Your Services

Fetchr also contains an express/connect middleware that can be used as your REST endpoint from the client.

```js
var server = express();
server.use(pluginInstance.getXhrPath(), pluginInstance.getMiddleware());
```


## Accessing Your Services

To maintain the Flux unidirectional data flow, fetchers are only accessible from action creators via the actionContext.

```js

/**
 * Calls the user service and sends the info to the store
 *
 * @method loadUser
 * @param {Object} context The context object
 * @param {Object} payload The payload object
 * @param {Function} done Called when action has completed
 * @async
 */

module.exports = function loadUser(context, payload, done) {

    context.service.read('user', {}, {}, function (err, userInfo) {

        if (err || !userInfo) {
            context.dispatch('RECEIVE_USER_INFO_FAILURE', err);
        } else {
            context.dispatch('RECEIVE_USER_INFO_SUCCESS', userInfo);
        }

        if (done) {
            done(err);
        }
    });
};

```