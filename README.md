# Hapi JSON View [![Build Status](https://travis-ci.org/gergoerdosi/hapi-json-view.svg)](https://travis-ci.org/gergoerdosi/hapi-json-view)

A view engine for the hapi framework.

## Installation

```
npm install hapi-json-view
```

## Usage

server.js:

```
var Path = require('path');
var Hapi = require('hapi');
var HapiJsonView = require('hapi-json-view');

var server = new Hapi.Server(3000);
server.views({
    engines: {
        js: {
            module: HapiJsonView.create(),
            compileMode: 'async'
        }
    },
    path: Path.join(__dirname, 'templates')
});

server.route({
    method: 'GET',
    path: '/article',
    handler: function (request, reply) {

        var article = {
            _id: '507f1f77bcf86cd799439011'
            title: 'Node.js',
            author: {
                _id: '507f191e810c19729de860ea'
                name: 'John Doe'
            }
        };

        reply.view('article', { article: article });
    }
});
```

templates/articles/read.js:

```
json.set('title', article.title);
json.set('author', function (json) {

    json.set('name', article.author.name);
});
```

This template generates the following object:

```
{
    title: 'Node.js',
    author: {
        name: 'John Doe'
    }
}
```

## Functions

### set()

It assigns a value to a key.

```
json.set('title', 'Node.js');

// => { title: 'Node.js' }
```

The value can be a function. If `json.set()` is called with a key, it creates an object:

```
json.set('author', function (json) {

    json.set('name', 'John Doe');
});

// => { author: { name: 'John Doe' } }
```

If `json.set()` is called without a key, it assign the value to the parent key:


```
json.set('title', function (json) {

    json.set('Node.js');
});

// => { title: 'Node.js' }
```

### array()

It creates a new array by iterating through an existing array:

```
var numbers = ['one', 'two'];

json.set('numbers', json.array(numbers, function (json, number) {

    json.set('number': number);
}));

// => { numbers: [{ number: 'one' }, { number: 'two' }] }
```

### extract()

It extracts values from an object and assigns them to the result object:

```
var numbers = { one: 'one', two: 'two', three: 'three' };

json.extract(numbers, ['two', 'three']);

// => { two: 'two', three: 'three' }
```

### helper()

Helpers can be registered through the engine configuration:

```
var Path = require('path');
var Hapi = require('hapi');
var HapiJsonView = require('hapi-json-view');

var server = new Hapi.Server(3000);
server.views({
    engines: {
        js: {
            module: HapiJsonView.create(),
            compileMode: 'async'
        }
    },
    path: Path.join(__dirname, 'templates'),
    helpersPath: Path.join(__dirname, 'templates/helpers');
});
```

They can then be used by their name:

```
json.set('title', json.helper('uppercase', article.title));
```

### partial()

Partials can be registered through the engine configuration:

```
var Path = require('path');
var Hapi = require('hapi');
var HapiJsonView = require('hapi-json-view');

var server = new Hapi.Server(3000);
server.views({
    engines: {
        js: {
            module: HapiJsonView.create(),
            compileMode: 'async'
        }
    },
    path: Path.join(__dirname, 'templates'),
    partialsPath: Path.join(__dirname, 'templates/partials');
});
```

They can then be used by their name:

```
json.set('author', json.partial('author', { author: article.author }));
```
