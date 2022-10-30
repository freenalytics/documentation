# Schema

Every application has a schema that defines the structure of the data that will be uploaded. Apart from defining
the structure of the data it is also used to validate it on upload.

Schemas cannot be updated when an application is created. If you need to change the structure of the data for your
application then you need to create a new application.

This limitation comes from the fact that the schema defines the structure of the data and how it is visualized.
If this changes, fields that are removed may no longer be visualized even though data still exists for those fields.

A schema should be valid a [JSON Schema](https://json-schema.org/draft/2020-12/json-schema-core.html) written in YML.
Every schema should be of type `object` and can have any type as properties.

The server will validate data on upload with the flag `additionalProperties` set to `false`. If your schema
contains this directive it will be overridden for the top level object.

Some examples of valid schemas:

```yaml
type: object
properties:
  name:
    type: string
  phone:
    type: number
  nested_data:
    type: object
    properties:
      title:
        type: string
      subtitle:
        type: string
    required:
      - title
      - subtitle
  arr_numbers:
    type: array
    items:
      type: number
required:
  - name
  - phone
```

Or,

```yaml
type: object
properties:
    command_error:
        type: string
    command_executed:
        type: string
    command_author:
        type: string
    daily_server_count:
        type: number
```
