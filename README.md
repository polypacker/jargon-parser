# Jargon Parser
### argument parsers from json schemas, using [subarg](https://github.com/substack/subarg) syntax and the [ajv validator](https://github.com/epoberezkin/ajv)
`jargon-parser` is designed to be a drop in `cli` builder that you don't have to manage or configure beyond the natural data schema for your application. This means you can easily use the same logic for your `.json` configurator, `REST` api, and `cli`, without losing power.
  
## Example Usage
With the [example/parser.js](example/parser.js) <small>(rough equivalent in `es6`)</small>:
```javascript
import newParser from 'jargon-parser'

const cli = newParser({ schema: __dirname + '/schema.json' })

console.log(JSON.stringify(cli()))
```
The invocation
```bash
node example/parser.js                              \
    --billing_address [                             \
        --street_address "1234 fake st" ]           \
    --shipping_address [                            \
        --street_address "2222 muddle drive"        \
        --city houston --type business ]            \
    --items [                                       \
        [ --name foo ]                              \
        [ --price 0.99 --name bar ] ]               \
    | python -mjson.tool # for pretty printing
```
yields
```json
{
    "billing_address": {
        "city": "Austin",
        "state": "TX",
        "street_address": "1234 fake st"
    },
    "shipping_address": {
        "city": "houston",
        "state": "TX",
        "street_address": "2222 muddle drive",
        "type": "business"
    },
    "items": [
        {
            "name": "foo"
        },
        {
            "name": "bar",
            "price": 0.99
        }
    ]
}
```
given the sample schema in [`examples/schema.json`](example/schema.json) (notice that `default`s are applied).
  
## In depth usage / customization
  
The [`default` export `newParser({schema, schemaCaster, ...options})`](src/parser.js#L31-L41) returns a `parser` that takes an optional array of `argv` arguments,
defaulting to `process.argv.slice(2)`, and returns the validated/cast result.

`schema` and `schemaCaster` are mutually exclusive, and `schemaCaster` takes precedence.
If `schema` is provided, either as as a relative path or object,
it will be passed to [newCaster](src/schema.js#L22-L38) with the [default ajv compiler](src/schema.js#L5-L9),
which will construct [`Avj`](https://github.com/epoberezkin/ajv) with the arguments `{ coerceTypes: true, useDefaults: true, v5: true }`.
The result of `newParser` 
  
## Notes
Aside from [`avj`](https://github.com/epoberezkin/ajv) type coercion and default filling, `jargon-parser`s function mostly the same as [subarg](https://github.com/substack/subarg)'s, except that arrays and dictionaries are mutually exclusive in json. This means that `cli command --sub [ args args -f ]` is invalid, because the `subarg` result is
```json
{
  "_": [ "command" ],
  "sub": {
    "_": [ "args" , "args" ],
    "f": true
  }
}
```
So `[ "command" ]` and `{ "sub": ... }` would be in conflict,
as would `[ "args" , "args" ]` and `{ "f": true }`.  
A valid alternative would be `cli command [ --sub [ --list [ args args ] -f ] ]`, resulting in 
```json
{
  "command": {
    "sub": {
      "list": [ "args" , "args" ],
      "f": true
    }
  }
}
```
