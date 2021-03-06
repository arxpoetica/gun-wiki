GUN can be used in both browsers and servers. We have made it easy to install in many different environments.

## Browser

There are a couple choices:

### Script Tag

The easiest is to just add GUN into your HTML:

```html
<script src="https://cdn.jsdelivr.net/npm/gun/gun.js"></script>
<script>
  // your code here
</script>
```

### Require

Assuming you are using something like **Webpack** or **Browserify**, first follow the [npm](#node) install, then add this to your browser code:

```javascript
var Gun = require('gun/gun');
```

### Import

Same as with `require`, but using the latest ES6 syntax:

```javascript
import Gun from 'gun/gun'
```

> Note: For now, even though you import it, GUN is still defined and used as a global variable.

## Node

First you need to install GUN with NPM or Yarn via the command line:

$`npm install gun`

or

$`yarn add gun`

> **Note:** If you don't have [node](http://nodejs.org/) or [npm](https://www.npmjs.com/) installed, [read this](https://docs.npmjs.com/getting-started/installing-node).

Then add this to your server code:

```javascript
var Gun = require('gun');
```

> Note: GUN comes with many default NodeJS adapters for storage and networking. If you do not want these, just do `require('gun/gun')` instead.

### Server

If you want to actually install GUN to a server, you need to pass it an HTTP instance:

```javascript
var server = require('http').createServer().listen(8080);
var gun = Gun({web: server});
```

For more examples on how to use it with something like **Express** or **Hapi**, check out these [samples](https://github.com/amark/gun/tree/master/examples).