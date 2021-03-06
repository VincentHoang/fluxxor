---
title: Stores
template: page.ejs
---

Stores
======

In a Flux application, the stores are responsible for managing business logic and data. They're akin to models or collections in MVC systems, but stores may manage more than a single piece of data or a single collection, as they are responsible for a *domain* of the application.

The *only* way to update stores is to send them an action by way of the dispatcher; stores should not have setter methods or properties that allow users to manipulate the store directly. Stores register their intent to respond to certain action types and actions with those types are routed to the appropriate handlers in the stores. Handlers are called with the action's payload and type as parameters.

Fluxxor supports dependencies between stores when necessary. If a store depends on data from other stores, it can wait for those stores to finish handling the currently dispatched action with the `waitFor` method.

Action dispatches are synchronous; stores can perform asynchronous updates, but should fire new actions at the end of the asynchronous operation if the rest of the system should be notified of a change in state as a result of the async operation. (Alternatively, perform asynchronous operations in the action methods themselves.) Once a store returns anything (including `undefined`) from an action handler, that store is considered to be done with that action's dispatch (unless it calls `waitFor`).

Stores can be added to a `Flux` instance programmatically using `Flux#addStore` or `Flux#addStores`; see the [Flux documentation](/documentation/flux.html) for more details.

## `Fluxxor.createStore(spec)`

Create a new store constructor.

* `spec` - An object describing the stores created with the returned constructor.

`spec` may contain any properties; functions will be automatically bound to the store instance and attached to it, and other properties will simply be attached to it. The following properties of `spec` behave specially:

* `actions` - An object map of action types to method names. Actions dispatched to the store with the given type will automatically be handled by the method with the corresponding name. Dynamic action types can be bound with the `bindActions` method. (`bindActions` is the recommended approach, since you can refer to constants defined elsewhere, which helps catch errors earlier and better supports minification.)

* `initialize(options)` - A function that will be called right after a store is instantiated. `options` is an optional object passed from the constructor.

Example:

```javascript
var MyStore = Fluxxor.createStore({
  actions: {
    "ACTION_TYPE": "handleActionType"
  },

  initialize: function(options) {
    this.value = options.value;

    // We could also use this in place of the `actions` hash, above:
    this.bindActions(
      "ACTION_TYPE", this.handleActionType
    );
  },

  handleActionType: function(payload, type) {
    // ...
  }
});

var myStore = new MyStore({value: 123});
```

## `Store#flux`

The [Flux](/documentation/flux.html) instance this store is contained within.

## `Store#bindActions(type, handler[, ...])`
## `Store#bindActions(actions)`

Binds action types to methods on the store.

`bindActions(type, handler[, ...])` takes any even number of arguments:

* `type` - The action type to bind to.
* `handler` - A function reference or method name (as a string) to call when actions of that type are dispatched to the store.

`bindActions(actions)` takes an actions hash in the same format as `spec.actions` described in `createStore`, above.

Using `bindActions` with constants defined elsewhere is less error prone and supports minification better than using the `actions` hash in the `createStore` spec or passing a hash to `bindActions`.

Example:

```javascript
var ACTION_TYPE = "ACTION_TYPE_1",
    OTHER_ACTION_TYPE = "ACTION_TYPE_2";

var MyStore = Fluxxor.createStore({
  initialize: function() {
    this.bindActions(
      ACTION_TYPE, this.handleActionType,
      OTHER_ACTION_TYPE, "handleOtherActionType"
    );

    // OR:

    this.bindActions({
      "ACTION_TYPE_1": "handleActionType",
      "ACTION_TYPE_2": "handleOtherActionType"
    });
  },

  handleActionType: function(payload, type) {
    // ...
  },

  handleOtherActionType: function(payload, type) {
    // ...
  }
});
```

## `Store#waitFor(stores, callback)`

Waits for other stores to finish dispatching the current action, executing `callback` afterwards. Since action handlers are synchronous, a store is "finished" handling an action when the handler function returns.

* `stores` - An array of names of stores to wait for.
* `callback(stores)` - A function to call after all the specified stores have handled the current dispatch. The function is called with the store instances (corresponding to the array of names).

`waitFor` will throw if it detects circular dependencies among the stores being dispatched to.

Example:

```javascript
Fluxxor.createStore({
  actions: {
    "ACTION_TYPE": "actionHandler"
  },

  actionHandler: function(payload, type) {
    this.waitFor(["otherStore", "anotherStore"], function(other, another) {
      // ...
    });
  }
});
```

Note that the callback is called synchronously.

## `EventEmitter` methods

Stores are instances of EventEmitters, and thus [inherit all the EventEmitter methods](http://nodejs.org/api/events.html#events_class_events_eventemitter). Most notably, stores should `emit` an event to notify the views that their data has changed.

The [`StoreWatchMixin`](/documentation/store-watch-mixin.html) assists with attaching event handlers to store change events in React applications.

Example:

```javascript
var MyStore = Fluxxor.createStore({
  // ...

  actionHandler: function(payload, type) {
    // some update to the store's data here
    this.emit("change");
  }
});

var myStore = new MyStore();

myStore.on("change", function() {
  // the store has updated its data
});
```
