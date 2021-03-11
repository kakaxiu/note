# EventEmitter3

[github](https://github.com/primus/eventemitter3)

>EventEmitter3 is a high performance EventEmitter. **It has been micro-optimized for various of code paths making this**, one of, if not the fastest EventEmitter available for Node.js and browser. The module is API compatible with the EventEmitter that by default with Node.js but there are some slight differences:
>
> * Domain support has been removed.
> * We do not `throw` an error when you emit an `error` event and nobody is listening.
> * The `newListener` and `removeListener` event has been removed as they are useful only in some uncommon use-cases.
> * The `setMaxListeners`, `getMaxListeners`, `prependListener` and `prependOnceListener` methods are not available.
> * The `removeListener` method removes all matching listeners, not only the first.
>
> It's drop in replacement for existing EventEmitters, but just faster. Free performance, who would't want that? EventEmitter is written in Ecmascript 3 so it will work in the oldest browsers and node versions that you need to support.

## Usage
> `var EventEmitter = require('eventemitter3');`

### Contextual emits
> We've upgraded the API of the `EventEmitter.on`, `EventEmitter.once` and `EventEmitter.removeListener` to accpet an extra argument which is the `context` or `this` value that should be set for the emitted events. This means you **no longer have the overhead of an event** that required `fn.bind` in order to get a custom `this` value.

```js
var EE = new EventEmitter(),
    context = { foo: 'bar' }

function emitted () {
  console.log(this === context)
}

EE.once('event-name', emitted, context)
EE.on('another-event', emitted, context)
EE.removeListener('another-event', emitted, context)
```

# Source code

This module exports es5 funcation base `class`.
```js
/**
 * Minimal `EventEmitter` interface that is molded against the Node.js
 * `EventEmitter` interface.
 *
 * @constructor
 * @public
 */
function EventEmitter() {
  this._events = new Events();
  this._eventsCount = 0;
}

//
// Expose the module.
//
if ('undefined' !== typeof module) {
  module.exports = EventEmitter;
}
```

The property of `EE` instance:
* eventNames
* listeners
* listenerCount
* emit
* on / addListener
* once
* removeListener / off
* removeAllListeners

### on
```js
/**
 * Representation of a single event listener.
 *
 * @param {Function} fn The listener function.
 * @param {*} context The context to invoke the listener with.
 * @param {Boolean} [once=false] Specify if the listener is a one-time listener.
 * @constructor
 * @private
 */
function EE(fn, context, once) {
  this.fn = fn;
  this.context = context;
  this.once = once || false;
}

/**
 * Add a listener for a given event.
 *
 * @param {EventEmitter} emitter Reference to the `EventEmitter` instance.
 * @param {(String|Symbol)} event The event name.
 * @param {Function} fn The listener function.
 * @param {*} context The context to invoke the listener with.
 * @param {Boolean} once Specify if the listener is a one-time listener.
 * @returns {EventEmitter}
 * @private
 */
function addListener(emitter, event, fn, context, once) {
  if (typeof fn !== 'function') {
    throw new TypeError('The listener must be a function');
  }

  // The context will replaces emitter instance if it exist.
  var listener = new EE(fn, context || emitter, once)
    , evt = prefix ? prefix + event : event;

  if (!emitter._events[evt]) emitter._events[evt] = listener, emitter._eventsCount++;
  else if (!emitter._events[evt].fn) emitter._events[evt].push(listener);
  else emitter._events[evt] = [emitter._events[evt], listener];

  return emitter;
}

/**
 * Add a listener for a given event.
 *
 * @param {(String|Symbol)} event The event name.
 * @param {Function} fn The listener function.
 * @param {*} [context=this] The context to invoke the listener with.
 * @returns {EventEmitter} `this`.
 * @public
 */
EventEmitter.prototype.on = function on(event, fn, context) {
  return addListener(this, event, fn, context, false);
};
```