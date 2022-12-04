# Official Web Template

The official web template allows you to integrate Freenalytics into your web project.

## Usage

In order to use this connector, you need to create a new application on your Freenalytics instance with the **Web Template** option on the create form.

This template will create a new application with the following schema:

```yaml
type: object
properties:
  page_title:
    type: string
  url_route:
    type: string
  user_time_in_page:
    type: number
  user_scrolled:
    type: boolean
  user_first_visit:
    type: boolean
  user_location:
    type: string
  referrer:
    type: string
  num_of_clicks:
    type: integer
  element_clicked:
    type: object
    properties:
      url_route:
        type: string
      tag_name:
        type: string
      class_name:
        type: string
      id:
        type: string
      page_x:
        type: integer
      page_y:
        type: integer
      client_x:
        type: integer
      client_y:
        type: integer
```

Once you have your new application set up, you should include the following script tags inside your `head` tag in your *html* page.

```html
<script type="text/javascript" src="https://cdn.jsdelivr.net/gh/freenalytics/freenalytics-connector-web@v1.0.0/connector.min.js"></script>
<script type="text/javascript" src="/analytics.js"></script>
```

Notice that the second `script` tag includes a local script. This script should be created by on your end in order to connect the connector
library to your own Freenalytics instance.

In this case, the `analytics.js` file could look like:

```js
const client = new freenalytics.Client({
  apiUrl: 'http://localhost:4000/api',
  domain: 'FD-107hpu34tlb7s7mro'
});
client.initialize();
```

!!! note
        Make sure to replace the `apiUrl` with your actual Freenalytics's API URL (must end with `/api`). Same goes with the `domain` which
        corresponds to your application's domain.

Once set, the connector will send the relevant information from the client visiting your website periodically.

## Example Application

An example application that uses this library has been built as an illustration. For more information, check out the [Webpage example](../examples/webpage-example.md) section.

## Development

If you're interested in developing this library, head over to the [Web connector](../development/connectors/freenalytics-connector-web.md) development guide.
