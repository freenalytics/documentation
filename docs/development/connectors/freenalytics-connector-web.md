# freenalytics-connector-web

!!! info
    This page refers to the development of the [freenalytics-connector-web](https://github.com/freenalytics/freenalytics-connector-web) connector library.
    If you're looking for the usage of this library, head over to the [Official Web Template](../../official-templates/web-template.md#usage) page.

The [freenalytics-connector-web](https://github.com/freenalytics/freenalytics-connector-web) is an official connector library
for webpages that integrate with the [Web Template](../../official-templates/web-template.md).

## Requirements

In order to develop for this connector library you will need:

* [git](https://git-scm.com/)
* [Node.js](https://nodejs.org/en/) (Version 16.15.1 was used)

## Setting Up

First, clone the repository:

```text
git clone https://github.com/freenalytics/freenalytics-connector-web
```

And then, install the dependencies:

```text
npm install
```

## Starting the Development Server

Since this library was made with TypeScript it needs to be bundled and served through a CDN.

In order to manually test functionality, you may start the bundler in watch mode (will re-bundle the project on file save) with:

```text
npm run build:watch
```

And then serve the contents of the `dist` folder with:

```text
npm run dev:serve
```

This will start an **HTTP** server on port **9000**.

### Manually Testing

In order to check functionality while developing the library, you may need to create a dummy **HTML** page that includes the library with the following tags:

```html
<script type="text/javascript" src="http://localhost:9000/connector.min.js"></script>
<script type="text/javascript" src="/analytics.js"></script>
```

Where the `analytics.js` file looks like this:

```js
const client = new freenalytics.Client({
  apiUrl: 'http://localhost:4000/api',
  domain: 'FD-107hpu34tlb7s7mro'
});
client.initialize();
```

!!! note
    You can use your own Freenalytics instance. In this case `http://localhost:4000/api` is just for example purposes. It may as well be `https://analytics.me.com/api` or whatever the URL for
    your Freenalytics instance may be.

You can then use any **HTTP** server to serve this dummy **HTML** page (such as the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension for VSCode).

Any code change will update the files that the **HTTP** server on port **9000** serves. However, you might need to hard refresh (Ctrl+Shift+R or Cmd+Shift+R) the webpage in case your browser has cached the script.

## Linting

This project uses ESLint rules to maintain a consistent code style. You can run the linter to check for any linting errors with:

```text
npm run lint
```

And fix any fixable errors automatically with:

```text
npm run lint:fix
```

## Unit Testing

This project contains unit tests that try to cover the code as much as possible. You can run the unit tests with:

```text
npm run test
```

Or, if you want to run the test suites in watch mode (will re-run relevant tests on file save), you can use:

```text
npm run test:watch
```
