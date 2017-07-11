# jsontemplate

Simple JSON value templating.

This package is an adaption (clone) of https://github.com/datavis-tech/json-templates. The only difference being change of regex to find token.

## Usage

Here's how you can use this library. Begin by installing via NPM:

`npm install json-templates`

Here's a small example of usage showing the simplest case, a single string.

```js
var parse = require("json-templates");

var template = parse("{{foo}}");

console.log(template.parameters); // Prints [{ key: "foo" }]

console.log(template({ foo: "bar" })); // Prints "bar"
```

Parameters can have default values, specified using a colon.

```js
var template = parse("{{foo:bar}}");

console.log(template.parameters); // Prints [{ key: "foo", defaultValue: "bar" }]

console.log(template()); // Prints "bar", using the default value.

console.log(template({ foo: "baz" })); // Prints "baz", using the given value.
```

Parameters can come from a nested object.

```js
var template = parse("{{foo.value:baz}}");

console.log(template.parameters); // Prints [{ key: "foo.value", defaultValue: "baz" }]

console.log(template()); // Prints "baz", using the default value.

console.log(template({ foo: { value: 'bar' } })); // Prints "bar", using the given value.

// Example with parameter coming from array
var template = parse({ a: "{{foo.1:baz}}" });

console.log(template.parameters); // Prints [{ key: "foo.1", defaultValue: "baz" }]

console.log(template()); // Prints { a: "baz" }, using the default value.

console.log(template({ foo: ["baq", "bar"] })); // Prints { a: "bar" }, using the given value of array.
```

The kind of templating you can see in the above examples gets applied to any string values in complex object structures such as ElasticSearch queries. Here's an example of an ElasticSearch query.

```js
var template = parse({
  index: "myindex",
  body: {
    query: {
      match: {
        title: "{{myTitle}}"
      }
    },
    facets: {
      tags: {
        terms: {
          field: "tags"
        }
      }
    }
  }
});

console.log(template.parameters); // Prints [{ key: "myTitle" }]

console.log(template({ title: "test" }));
```

The last line prints the following structure:

```js
{
  index: "myindex",
  body: {
    query: {
      match: {
        title: "test"
      }
    },
    facets: {
      tags: {
        terms: {
          field: "tags"
        }
      }
    }
  }
}
```

The parse function also handles nested arrays and arbitrary leaf values. For more detailed examples, check out the [tests](https://github.com/curran/json-templates/blob/master/test.js).


## Why?

The use case for this came about while working with ElasticSearch queries that need to be parameterized. We wanted the ability to *speficy query templates within JSON*, and also make any of the string values parameterizable. The ideas was to make something kind of like [Handlebars](http://handlebarsjs.com/), but just for the values within the query.

We also needed to know which parameters are required to "fill in" a given query template (in order to check if we have the right context parameters to actually execute the query). Related to this requirement, sometimes certain parameters should have default values. These parameters are not strictly required from the context. If not specified, the default value from the template will be used, otherwise the value from the context will be used.

Here's how the above `title` parameter could have a default value of `test`:

```
{
  index: 'myindex',
  body: {
    query: {
      match: {
        title: '{{title:test}}'
      }
    },
    facets: {
      tags: {
        terms: {
          field: 'tags'
        }
      }
    }
  }
}
```

Also it was a fun challenge and a great opportunity to write some heady recursive functional code.

## Related Work

 * [json-templater](https://www.npmjs.com/package/json-templater)
 * [bodybuilder](https://github.com/danpaz/bodybuilder)
 * [elasticsearch-query-builder](https://github.com/leonardw/elasticsearch-query-builder)
