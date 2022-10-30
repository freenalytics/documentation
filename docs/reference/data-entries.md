# Data Entries

Every application exposes an endpoint on the server to upload data entries.

Data entries need to conform to the [schema](./schema.md) defined in the application.

Depending on the schema defined, you may or may not upload incomplete data. This is useful in the case
where your schema includes data properties for which you might not be able to upload data at the same time.

For example, in the example seen in [creating your first application](../getting-started/creating-your-first-application.md), for the following schema:

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

The property `daily_server_count` is supposed to be uploaded once every day on a cronjob basis.

For the case of the rest of the properties, they're supposed to be uploaded whenever a user executes a command, and whether
this command errors or not it may or may not include the `command_error` property.

Given this case, there will be rows for each data entry where not everything will have a value.

For more information on how to upload data, check out [uploading data](../getting-started/uploading-data.md).
