# asyncback

![Build Status](https://travis-ci.org/codebysd/node-asyncback.svg?branch=master)
[![npm downloads](https://img.shields.io/npm/dt/asyncback.svg)](https://www.npmjs.com/package/asyncback)
[![Open issues](https://img.shields.io/github/issues/codebysd/node-asyncback.svg)](https://github.com/codebysd/node-asyncback/issues)
[![Pull requests](https://img.shields.io/github/issues-pr/codebysd/node-asyncback.svg)]()

Convert async functions to callback style functions.

## Why ?

Async functions along with `await` has been touted as a very elegant way of writing async code. However there is a lot of Javascript API that requires callback functions. For example, [ExpressJS](https://expressjs.com/) requires [middleware functions](https://expressjs.com/en/guide/writing-middleware.html) to be plain old functions that accept a `next` callback.

The `asyncback` module converts an async function to a callback accepting function.

### In Short

This:

```javascript
async function(arg1, arg2, .., argN)
``` 
is converted to:

```javascript
function(arg1, arg2, .., argN, callback)
```

Which is awesome in cases like writing an expressjs request handler, while using mongoosejs models:

```javascript
const asyncback = require('asyncback');

app.get('/users', asyncback(async (req, res) => {
    const users = await User.find({ 'banned': false });
    const offer = await Offers.findOne({ 'active': true });
    res.json({ 'users': users, 'offer': offer });
}));
```


## Usage

Install module:

```bash
npm install asyncback --save
```

Wrap async functions with `asyncback` before registering them with ExpressJS:

```javascript
const asyncback = require('asyncback');

app.get('/path', asyncback(async (req, res) => {
    // async await style code
}));
```

Better yet, if middleware functions are in separate files, simply export the wrapped function:

```javascript
const asyncback = require('asyncback');

async function someMiddleware(req, res){
    // async await style code
}

module.exports = asyncback(someMiddleware);
```

# Notes

1. The `next` callback supplied by ExpressJS will be automatically called after async middleware function returns.
2. In case the async middleware function throws an error, the `next` callback will be called with the thrown error.

# Flexibility

The `asyncback` function is really generic that it converts an async function into a callback style function which accepts a callback as its last parameter. Hence its usage is not limited to just ExpressJS middleware. This also works for functions that return a promise or promise-like result.

# Safety

The `asyncback` can safely wrap non async/promised functions and the returned function can be invoked without the callback parameter. In such cases the wrapped function will be called synchronously and with all supplied arguments. However such usage is discouraged as it serves no beneficial purpose.
