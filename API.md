---------------------------------------------------------------------

***Please post documentation comments, questions, and suggestions to
[[issue #70|https://github.com/amark/gun/issues/70]].***

---------------------------------------------------------------------
# Core API
 - [Gun constructor](#Gun)
 - [gun.put](#put)
 - [gun.get](#get)
 - [gun.back](#back)
 - [gun.opt](#opt)

# API
 - [gun.path](#path)
 - [gun.on](#on)
 - [gun.val](#val)
 - [gun.set](#set)
 - [gun.map](#map)
 - [gun.not](#not)
 - [gun.init](#init)
 - [gun.key](#key)

# <a name="Gun"></a>Gun(options)

<a href="https://youtu.be/zvo6jC1OA3Y" title="GUN constructor"><img src="http://img.youtube.com/vi/zvo6jC1OA3Y/0.jpg" width="425px"></a><br>
Used to creates a new gun database instance.

```javascript
var gun = Gun(options)
```
> **note:** `Gun` works with or without the `new` operator.

## Options

 - no parameters `undefined` creates a local datastore using the default persistence layer, either localStorage or a JSON file.

 - passing a URL `string` creates the above local datastore that also tries to sync with the URL.

   - or you can pass in an `array` of URLs to sync with multiple peers.

 - the previous options are actually aggregated into an `object`, which you can pass in yourself.

   - `options.peers` is an object, where the URLs are properties, and the value is an empty object.

   - `options.uuid` allows you to override the default 24 random alphanumeric soul generator with
      your own function.

   - `options['module name']` allows you to pass options to a 3rd party module. Their project README
     will likely list the exposed options.
     [Here is a list of such modules...](Modules)

### Examples
Sync with one peer
```javascript
Gun('http://yourdomain.com/gun')
```

Sync with many peers
```javascript
Gun(['http://server1.com/gun', 'http://server2.com/gun'])
```

Working with modules
```javascript
Gun({
  // Amazon S3 (comes bundled)
  s3: {
    key: '',
    secret: '',
    bucket: ''
  },

  // simple JSON persistence (bundled)
  // meant for ease of getting started
  // NOT meant for production
  file: 'file/path.json',

  // set your own UUID function
  uuid: function () {...}
})
```
---------------------------------------------------------
<h1><a name="put"></a>gun.put(data, callback)</h1>

<a href="https://youtu.be/QLg-Z-y5sVo" title="GUN put"><img src="http://img.youtube.com/vi/QLg-Z-y5sVo/0.jpg" width="425px"></a><br>

Save data into gun, syncing it with your connected peers.

It has three parameters, and only the first is required:

 1. the `data` to save
 2. an optional `callback`, invoked on each acknowledgment

`gun.get('key').put({hello: "world"}, function(ack){})`

You do not need to re-save the entire object every time, gun will automatically merge your data into what already exists as a "partial" update.

## Allowed types

`.put` restricts the input to a specific subset:

 - `objects`:
   [partial](Partials-and-Circular-References-(v0.2.x)),
   [circular](Partials-and-Circular-References-(v0.2.x)#circular-references), and nested
 - `strings`
 - `numbers`
 - `booleans`
 - `null`

Other values, like `undefined`, `NaN`, `Infinity`, `arrays`, will be rejected.

> Traditional arrays are dangerous in real-time apps. Use [gun.set](#set) instead.

> **Note:** when using `.put`, if any part of the chain does not exist yet, it will implicitly create it as an empty object.
```javascript
gun.get('something').path('that.does.not.exist.yet').put("Hello World!");
// `.put` will if needed, backwards create a document
// so "Hello World!" has a place to be saved.
```

## Callback(ack)
  
 - `ack.err`, if there was an error during save.
 - `ack.ok`, if there was a success message (none is required though).

The `callback` is fired for each peer that responds with an error or successful persistence message, including the local cache. Acknowledgement can be slow, but the write propagates across networks as fast as the pipes connecting them.

If the error property is undefined, then the operation succeeded, although the exact values are left up to the module developer.

## Examples

Saving objects
```javascript
gun.get('key').put({
  property: 'value',
  object: {
    nested: true
  }
})
```

Saving primitives
```javascript
// strings
gun.get('person').path('name.first').put('Alice')

// numbers
gun.get('IoT').path('temperature').put(58.6)

// booleans
gun.get('player').path('alive').put(true)
```

Using the callback
```javascript
gun.get('survey').path('submission').put(submission, function(ack){
  if(ack.err){
    return ui.show.error(ack.err)
  }
  ui.show.success(true)
})
```

## Chain context
`gun.put` does not change the gun context.
```javascript
gun.get('key').put(data) /* same context as */ gun.get('key')
```

## Unexpected behavior

You cannot save primitive values at the root level.
```javascript
Gun().put("oops");
```
All data is normalized to a parent node.
```javascript
Gun().put({foo: 'bar'}); // internally becomes...
Gun().get(randomUUID).put({foo: 'bar'});

Gun().get('user').path('alice').put(data); // internally becomes...
Gun().get('user').put({'alice': data});
// An update to both user and alice happens, not just alice.
```
You can save a gun chain reference,
```javascript
var ref = Gun().put({text: 'Hello world!'})
Gun().get('message').path('first').put(ref)
```
But you cannot save it inline.
```javascript
var sender = Gun().put({name: 'Tom'})
var msg = Gun().put({
  text: 'Hello world!',
  sender: sender // this will fail
})
// however
msg.path('sender').put(sender) // this will succeed
``` 
Be careful saving deeply nested objects,
```javascript
Gun().put({
  foo: {
    bar: {
      lol: {
        yay: true
      }
    }
  }
}):
```
For the most part, gun will handle this perfectly fine. It will attempt to automatically merge every nested object as a partial. However, if it cannot find data (due to a network failure, or a peer it has never spoken with) to merge with it will generate new random UUIDs. You are unlikely to see this in practice, because your apps will probably save data based on user interaction (with previously loaded data). But if you do have this problem, consider giving each one of your sub-objects a deterministic ID.

------------------------------------------------------------------------------------------
# <a name="get"></a>gun.get(key)
Where to read data from.

<a href="https://youtu.be/wNrIrrLffs4" title="GUN get"><img src="http://img.youtube.com/vi/wNrIrrLffs4/0.jpg" width="425px"></a><br>

It takes three parameters:

 - `key`
 - `callback`

`gun.get('key').get('property', function(ack){})`

You will usually be using [gun.on](#on) or [gun.val](#val) to actually retrieve your data, not this `callback` (it is intended for more low level control, for module and extensions).

## Key
The `key` is the ID or property name of the data that you saved from earlier (or that will be saved later).

> Note that if you use `.put` at any depth after a `get` it first reads the data and then writes, merging the data as a partial update.

```javascript
gun.get('key').put({property: 'value'})

gun.get('key').on(function(data, key){
  // {property: 'value'}, 'key'
})
```

## Callback(ack)

 - `ack.put`, the raw data.
 - `ack.get`, the key, ID, or property name of the data.

The callback is a listener for read errors, not found, and updates. It may be called multiple times for a single request, since gun uses a reactive streaming architecture. Generally, you'll find [`.not`](#not), [`.on`](#on), and [`.val`](#val) as more convenient for every day use. Skip to those!

```javascript
gun.get(key, function(ack){
  // called many times
})
```

## Examples

Retrieving a key
```javascript
// retrieve all available users
gun.get('users').map().on(ui.show.users)
```

Using the callback
```javascript
gun.get(key, function(ack){
  if(ack.err){
    server.log(error)
  } else
  if(!ack.put){
    // not found
  } else {
    // data!
  }
})
```

## Chain context
Chaining multiple `get`s together changes the context of the chain, allowing you to access, traverse, and navigate a graph, node, table, or document.

> Note: For users upgrading versions, prior to v0.5.x `get` used to always return a context from the absolute root of the database. If you want to go back to the root, either save a reference `var root = Gun();` or now use [`.back(-1)`](#back).

```javascript
gun.get('user').get('alice') /* same context as */ gun.get('users').path('alice')
```

## Unexpected behavior

Most callbacks in gun will be called multiple times.

-----------------------------
# <a name="back"></a>gun.back(amount)

Move up to the parent context on the chain.

Every time a new chain is created, a reference to the old context is kept to go `back` to.

## Amount

The number of times you want to go back up the chain. `-1` or `Infinity` will take you to the root.

## Examples
Moving to a parent context
```javascript
gun.get('users')
  /* now change the context to alice */
  .get('alice')
  .put(data)
  /* go back up the chain once, to 'users' */
  .back().map(...)
```

Another example
```javascript
gun.get('player').path('game.score').back(1)
// is the same as...
gun.get('player').path('game')
```

## Chain context
The context will always be different, returning you to the
```javascript
gun.get('key').get('property')
/* is not the same as */
gun.get('key').get('property').back()
```

--------------------------------------
# <a name="opt"></a> gun.opt(options)
Change the configuration of the gun database instance.

The `options` argument is the same object you pass to the [constructor](#Gun). The `options`'s properties replace those in the instance's configuration but `options.peers` are **added** to peers known to the gun instance.

## Examples
Create the gun instance.
```javascript
gun = Gun('http://yourdomain.com/gun')
```
Change UUID generator:
```javascript
gun.opt({
  uuid: function () {
    return Math.floor(Math.random() * 4294967296);
  }
});
```
Add more peers:
```javascript
gun.opt({peers: ['http://anotherdomain.com/gun']})
/* Now gun syncs with ['http://yourdomain.com/gun', 'http://anotherdomain.com/gun']. */
```

---------------------------------------
# <a name="path"></a>gun.path(key)

<a href="https://youtu.be/UDZGVYLNLAU" title="GUN path"><img src="http://img.youtube.com/vi/UDZGVYLNLAU/0.jpg" width="425px"></a><br>

Path does the same thing as `get` but has some conveniences built in.

## Key
The key `property` is the name of the field to move to.

```javascript
// move to the "themes" field on the settings object
gun.get('settings').path('themes')
```

Once you've changed the context, you can read, write, and `path` again from that field. While you can just chain one `path` after another, it becomes verbose, so there are two shorthand styles:

 - dot format
 - array format

Here's dot notation in action:
```javascript
// verbose
gun.get('settings').path('themes').path('active')

// shorthand
gun.get('settings').path('themes.active')
```

And the array format, which really becomes useful when using variables instead of literal strings:
```javascript
gun.get('settings').path(['themes', themeName])
```

### Unexpected behavior
The dot notation can do some strange things if you're not expecting it. Under the hood, everything is changed into a string, including floating point numbers. If you use a decimal in your path, it will split into two paths...
```javascript
gun.path(30.5)
// interprets to
gun.path(30).path(5)
```

This can be especially confusing as the chain might never resolve to a value.

> Note: For users upgrading from versions prior to v0.5.x, `path` used to be necessary - now it is purely a convenience wrapper around `get`.

## Examples
Navigating to a property
```javascript
/*
  where `user` is {
    name: 'Bob'
  }
*/
gun.get('user').path('name')
```
Once you've focused on the `name` property, you can chain other methods like [`.put`](#put) or [`.on`](#on) to interact with it.

Moving through multiple properties
```javascript
/*
  where `user` is {
    name: { first: 'bob' }
  }
*/
gun.get('user').path('name').path('first')
// or the shorthand...
gun.get('user').path('name.first')
```

## Chain context
`gun.path` creates a new context each time it's called, and is always a result of the previous context.
```javascript
gun.get('API').path('path').path('chain')
/* is different from */
gun.get('API').path('path')
/* and is different from */
gun.get('API')
```

-----------------------------
# <a name="on"></a> gun.on(callback, option)

<a href="https://youtu.be/SEneRvDQysE" title="GUN on"><img src="http://img.youtube.com/vi/SEneRvDQysE/0.jpg" width="425px"></a><br>

Subscribe to updates and changes on a node or property in realtime.

## Callback(data, key)
When the property or node you're focused on changes, this callback is immediately fired with the data as it is at that point in time.

Since gun streams data, the callback will probably be called multiple times as new chunk comes in.

## Option
Currently, the only option is to filter out old data, and just be given the changes. If you're listening to a node with 100 fields, and just one changes, you'll instead be passed a node with a single property representing that change rather than the full node every time.

**Longhand syntax**
```javascript
gun.get('foo').on(callback, {
  change: true
})
```

**Shorthand syntax**
```javascript
gun.get('foo').on(callback, true)
```

## Examples
Listening for updates on a key
```javascript
gun.get('users').path(username).on(function(user){
  // update in real-time
  if (user.online) {
    view.show.active(user.name)
  } else {
    view.show.offline(user.name)
  }
})
```

Listening to updates on a field
```javascript
gun.get('lights').path('living room').on(function(state, room){
  // update the UI when the living room lights change state
  view.lights[room].show(state)
})
```

## Chain Context
`gun.on` does not change the chain context.
```javascript
gun.get(key).on(handler) /* is the same as */ gun.get(key)
```

## Unexpected behavior

Data is only 1 layer deep, a full document is not returned (there are extensions available that do that), this helps keep things fast.

It will be called many times.

-------------------------------------
# <a name="val"></a> gun.val(callback, option)

<a href="https://youtu.be/k-CkP43-uJo" title="GUN val"><img src="http://img.youtube.com/vi/k-CkP43-uJo/0.jpg" width="425px"></a><br>

Get the current data without subscribing to updates.

## Option

 - `wait` controls the asynchronous timing (see unexpected behavior, below). `gun.get('foo').val(cb, {wait: 0})`

## Callback(data, key)
The data is the value for that chain at that given point in time. And they key is the last property name or ID of the node.

## Examples
```javascript
gun.get('peer').path(userID).path('profile').val(function(profile){
  // render it, but only once. No updates.
  view.show.user(profile)
})
```

Reading a property
```javascript
gun.get('IoT').path('temperature').val(function(number){
  view.show.temp(number)
})
```

## Chain Context
`gun.val` does not currently change the context of the chain, but it is being discussed for future versions that it will - so try to avoid chaining off of `.val` for now.

## Unexpected behavior

`.val` is synchronous and immediate (at extremely high performance) if the data has already been loaded.

`.val` is asynchronous and on a **debounce timeout** while data is still being loaded - so it may be called completely out of order compared to other functions. This is intended because gun streams partials of data, so `val` avoids firing immediately because it may not represent the "complete" data set yet. You can control this timeout with the `wait` option.

Data is only 1 layer deep, a full document is not returned (there are extensions available that do that), this helps keep things fast.

---------------------------------------------------------
# <a name="set"></a>gun.set(data, callback)

Add a unique item to an unordered list.

`gun.set` works like a mathematical set, where each item in the list is unique. If the item is added twice, it will be merged. This means only objects, for now, are supported.

## Data
Data should be a gun reference or an object.
```javascript
var user = gun.get('alice').put({name: "Alice"});
gun.get('users').set(user);
```

## Callback
The callback is invoked exactly the same as `.put`, since `.set` is just a convenience wrapper around `.put`.

## Examples

```javascript
var gun = Gun();
var bob = gun.get('bob').put({name: "Bob"});
var dave = gun.get('dave').put({name: "Dave"});

dave.path('friends').set(bob);
bob.path('friends').set(dave);
```
The "friends" example is perfect, since the set guarantees that you won't have duplicates in your list.

## Chain Context
`gun.set` changes the chain context, it returns the item reference.
```javascript
gun.path('friends') /* is not the same as */ gun.path('friends').set(friend)
```

------------------------------------
# <a name="map"></a>gun.map(callback)

<a href="https://youtu.be/F2FSMsxMSic" title="GUN map"><img src="http://img.youtube.com/vi/F2FSMsxMSic/0.jpg" width="425px"></a><br>

Loop over each property in a node, and listen to each property as well as any changes on the node itself (such as future inserts). It is essentially performing a [`.path`](#path) on each field.

If you don't know the property names to [`.path`](#path) into, or need to `"forEach"` over a group of data, the `.map` function is usually your best choice. It accepts two arguments:

 - a `callback` function.

> Note: In future versions of gun the `callback` function will act as a transform filter. If no callback is specified, it will default to a transform filter that makes no changes to the data (therefore acting like a `forEach`). In the meantime, we highly recommend you do not pass `map` a callback, and instead either `map().on(cb)` or `map().val(cb)` to get the data.

## Examples
Iterate over an object
```javascript
/*
  where `stats` are {
    'new customers': 35,
    'returning': 65
  }
*/
gun.get('stats').map().on(function(percent, category) {
  pie.chart(category, percent)
})
```
Or `forEach`ing through every user.
```javascript
gun.get('users').map().val(function(user, id){
  ui.list.user(user);
});
```

## Chain context
`.map` changes the context of the chain to hold many chains simultaneously. Check out this example:
```javascript
gun.get('users').map().path('name').on(cb);
```
Everything after the `map()` will be done for every item in the list, such that you'll get called with each name for every user in the list. This can be combined in really expressive and powerful ways.
```javascript
gun.get('users').map().path('friends').map().path('pet').on(cb);
```
This will give you each pet of every friend of every user!

```javascript
gun.get(key).map() /* is not the same as */ gun.get(key)
```

--------------------------------------
# <a name="not"></a> gun.not(callback)

Handle cases where data can't be found.

If you need to know whether a property or key exists, you can check with `.not`. It will consult the connected peers and invoke the callback if there's reasonable certainty that none of them have the data available.

> **Warning:** `.not` has no guarantees, since data could theoretically exist on an unrelated peer that we have no knowledge of. If you only have one server, and data is synced through it, then you have a pretty reasonable assurance that a `not` found means that the data doesn't exist yet. Just be mindful of how you use it.

## Callback(key)
If there's reason to believe the data doesn't exist, the callback will be invoked. This can be used as a check to prevent implicitly writing data (as described in [`.put`](#put)).

### Key
The name of the property or key that could not be found.

## Examples
Providing defaults if they aren't found
```javascript
// if not found
gun.get('players/3').not(function(key){
  // put in an object and key it
  gun.get(key).put({
    active: false
  });
}).on(handler)
// listen for changes on that key
```

Setting a property if it isn't found
```javascript
gun.get('chat').path('enabled').not(function(path){
  this.put(false)
})
```

## Chain context
`.not` does not change the context of the chain.
```javascript
gun.get(key).not(handler) /* is the same as */ gun.get(key)
```


--------------------------------
# <a name="init"></a> gun.init()
Is being deprecated, now in 0.5.x and above.

--------------------------------------------------
# <a name="key"></a>gun.key(name)
Currently disabled. Potentially will be deprecated or made into an extension.