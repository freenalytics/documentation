# Custom Connector Development

[Connectors](../../reference/connector.md) are a way to share a common interface that can integrate your particular projects with Freenalytics.

Connectors generally depend on the [schema](../../reference/schema.md) and the type of application you're trying to connect.

This section will guide you through the development process of creating your own connector library.

## Naming

For the sake of ease of finding your connector library, it is recommended that your library's name begins with **freenalytics-connector-**. (i.e [freenalytics-connector-web](https://github.com/freenalytics/freenalytics-connector-web))

This is obviously not mandatory, but it is recommended.

## Define a Schema

Since connectors depend on a [schema](../../reference/schema.md), you should first define the structure of the data that you plan to retrieve and save in your Freenalytics application.

As you may know, schemas are defined using [JSONSchema](https://json-schema.org/). As a requirement, these schemas **must** be of type `object` at the top level.

An example of what a schema could look like:

```yaml
type: object
properties:
  command_name:
    type: string
  command_author:
    type: string
  command_success:
    type: boolean
  command_error_message:
    type: string
  server_count:
    type: integer
  member_count:
    type: integer
```

!!! note
    This is the schema used by the [Discord Bot Example](../../examples/discord-bot-example.md).

## Creating the Connector Client

Connector clients are just an interface that integrates to Freenalytics through **HTTP(s)** API calls.

You're free to use any language of your choice.

### Required Parameters

Your connector client may have multiple parameters that may adjust its functionality that could be specified
by a developer using your library. However, there are some parameters that your client should take into account:

1. `apiUrl`: The developer should be able to specify the **URL** of their Freenalytics API instance.
2. `domain`: The developer should also be able to include the [application domain](../../reference/application.md) to use for their own project.

### Uploading Data

In order to upload data payloads, your client should make an **HTTP POST** request with a `application/json` body containing the data payload to the following address:

```text
$API_URL$/applications/$DOMAIN$/data
```

The shape of the payload may differ across different types of requests. After all, your schema may not necessarily describe data with entries that are interdependent with each other.

### Additional Considerations

It is generally expected that connector libraries serve as some sort of automatic data upload. The developer shouldn't need to "think" about when to upload the analytics data,
it is up to the connector to try and upload this data automatically, generally through the use of events.

Your connector may expose data upload methods to allow the developer to explicitly to upload something. However, in the case that your connector library is able to
register an event handler that can automate data upload, it should be the preferred way.

## Example

As an example of what a connector client may look like, the code below includes the client used by the [Discord Bot Example](../../examples/discord-bot-example.md) which integrates
with the schema displayed above.

<script src="https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Ffreenalytics%2Fexample-discord-bot%2Fblob%2Fmaster%2Fsrc%2Fthird-party%2Ffreenalytics%2FFreenalyticsClient.js&style=github-gist&showBorder=on&showLineNumbers=on&showFileMeta=on&showCopy=on&fetchFromJsDelivr=on"></script>

In this case, the client has 3 parameters that allow the developer to customize the behavior of the client:

1. `options.apiUrl` to specify to what Freenalytics instance the data will go to.
2. `options.domain` to specify to what Freenalytics application the data will go to.
3. `options.interval` to specify a polling interval that the project will use to upload the `server_count` and `member_count` variables.

Next, the client exposes a `initialize()` method that registers event handlers and interval jobs that will automatically upload the relevant data to the Freenalytics instance.

In this case, the `client` object (which is a [ExtendedClient from @greencoast/discord.js-extended](https://docs.greencoaststudios.com/discord.js-extended/master/classes/Discord_js_Extended.ExtendedClient.html))
is the main client that represents the Discord bot, it is an `EventEmitter` that emits events whenever a command is issued and whenever it runs into an error.
Using these events, the connector client can easily register handlers for these events that can automatically upload data regarding command execution by the users of the bot, through the
`handleCommandExecution()` and `handleCommandError()` methods.

Additionally, the `postIntervalHandler()` method calculates the number of servers and members that the bot has access to and uploads this data automatically.

With this in mind, the developer that wants to use this connector client will only need to instantiate and initialize it as such:

```js
const client = new ExtendedClient(); // This is the @greencoast/discord.js-extended Client. It's functionality is irrelevant to the example.

client.analytics = new FreenalyticsClient(client, {
  apiUrl: 'http://localhost:4000/api',
  domain: 'FD-107hpuvshlb8i8mb1',
  interval: 10000
});
client.analytics.initialize();
```

And that's it. The developer can now "forget" that this analytics client exists.
