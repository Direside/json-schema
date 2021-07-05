# JSON Schema

This is a fork from the original [json-schema](https://github.com/ruby-json-schema/json-schema) library to address an issue with nesting mutliple artifacts inside each other.

## Problem

The original library will only traverse the properties of a schema and load any refs. If the loaded ref is just a simple JSON object it won't parse any of the $ref values inside of those objects and therefore will fail to validate them when you run the `validate!` function.

Essentially it stops parsing refs after the main schema, so a ref nested inside another ref will not be loaded.

### This Works

One level of nesting is fine as it will find the ref and load the `common_defs_schema.json` into memory.

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "title": "Directory of People",
  "properties": {
    "people":{
      "type": "array",
      "items": {
        "$ref": "./common_defs_schema.json#/person"
      }
    }
  }
}
```

`common_defs_schema.json`

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "person": {
    "title": "Person",
    "type": "string",
    "properties": {
      "name": {
        "type": "string"
      }
    }
  }
}
```

### Doesn't Work

Notice here we have a nested ref that points to a third file. This third file will not be loaded and when the validation is called you will get an error. `The property '#/persom//0' was not a valid schema in schema common_defs_schema.json#`.

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "title": "Directory of People",
  "properties": {
    "people":{
      "type": "array",
      "items": {
        "$ref": "./common_defs_schema.json#/person"
      }
    }
  }
}
```

`common_defs_schema.json`

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "person": {
    "title": "Person",
    "type": "string",
    "properties": {
      "$ref": "./name.json"
    }
  }
}
```

`name.json`

```json
{
  "name": {
    "type": "string"
  }
}
```

## Fix

I've implemented a basic recursive pass of the schemas after calling `build_schema` that looks for any `$ref` values in that schema and loads them as well. This isn't very efficient as it's loading all the refs in the file, not just the ones referenced.

At some point I may fix that, but it seems I'll need to mess with the logic a bit and didn't want to spend time on that for this fix.
