# Server Development

The [freenalytics/freenalytics](https://github.com/freenalytics/freenalytics) repository is a monorepo containing both the web dashboard and the server.

Any changes made to the server should be made in this repository.

## Requirements

In order to develop for the server you will need:

* [git](https://git-scm.com/)
* [Node.js](https://nodejs.org/en/) (Version 16.15.1 was used)
* [Docker](https://www.docker.com/) (With `docker-compose`)

## Setting Up

First, clone the repository:

```text
git clone https://github.com/freenalytics/freenalytics
```

Head over to the `server` folder:

```text
cd server
```

And then, install the dependencies:

```text
npm install
```

## Setting Up the Development Environment

The server requires [MongoDB](https://www.mongodb.com/) and [Redis](https://redis.io/) instances in order to run. The `server` folder includes
a `dev` folder with a `docker-compose.yml` file inside with a MongoDB and Redis service.

Head over to this folder and run:

```text
docker-compose up
```

This will start both a MongoDB server at port **27017** and a Redis server at port **6379**.

### Environment Variables

Inside the `server` folder create a `.env` file with the following content:

```text
MONGODB_URI=mongodb://root:password@localhost:27017/freenalytics?authSource=admin
REDIS_URI=redis://localhost:6379
JWT_SECRET=my_super_secret
JWT_TOKEN_DURATION=604800
REGISTRATION_OPEN=true
```

!!! note
        Nothing in this folder needs to be updated. It already sets the MongoDB and Redis URIs to the appropriate values, taking into account
        that you have started these services with the `docker-compose.yml` file that was previously mentioned.

## Starting the Server

### Development Mode

There are two ways to start the server in development mode:

* If you want to run the server without automatically restarting on file save you can use:

```text
npm run dev
```

* If you wish to use watch mode (server restarts on file save), use:

```text
npm run dev:watch
```

In both cases the server will start on port **4000**.

### Production Mode

In case you wish to start the server in production mode:

You need to first build the server:

```text
npm run build
```

And then start it:

```text
npm run start
```

### Manually Testing

In order to check functionality while developing the server, you may need an **HTTP** client such as [Insomnia](https://insomnia.rest/) or [Postman](https://www.postman.com/).

The URL to test will be: `http://localhost:4000/api`. Check the `src/routes` folder for the relevant routers that you wish to test.

## Considerations

### Linting

This project uses ESLint rules to maintain a consistent code style. You can run the linter to check for any linting errors with:

```text
npm run lint
```

And fix any fixable errors automatically with:

```text
npm run lint:fix
```

### Unit Testing

This project contains unit tests for every component. You can run the unit tests with:

```text
npm run test
```

Or, if you want to run the test suites in watch mode (will re-run relevant tests on file save), you can use:

```text
npm run test:watch
```

## Developing Routes

Route development will usually go like this:

1. You create a service function to query or mutate the Mongo database.
2. You create a controller function that will handle the incoming request.
3. You create a route function that will map the controller to a specific route and HTTP method.

Each of these entities are independent to ensure modularity and ease of testing. Each of these functions
should be unit tested. Routers do not need to be tested because the logic inside of them is already
tested in the unit test for the controller.

* Services that talk to the database are located inside the `src/services` folder.
* Controllers that handle requests are located inside the `src/controllers` folder.
* Routers that map the request handlers to the HTTP route and method are located in `src/routes` folder.

### Example

Let's see the example of the data upload route which is the most complex one currently in the application.
This route includes schema fetching from the Redis cache, body validation against the stored schema, and a
Mongo database insertion.

#### Model

Inside the `src/models` folder you will find [mongoose](https://mongoosejs.com/) models that will be used
to define the shape of the data stored in MongoDB. These models are used to interact with the database.

```ts
// src/models/data.ts
import { Schema, model } from 'mongoose';

export interface DataModel {
  payload: object
  domain: string
  createdAt: Date
}

const dataSchema = new Schema<DataModel>({
  payload: {
    type: Schema.Types.Mixed,
    required: true
  },
  domain: {
    type: String,
    required: true
  }
}, {
  timestamps: {
    createdAt: 'createdAt',
    updatedAt: false
  }
});

export default model<DataModel>('Data', dataSchema);
```

In here, the `Data` model defines the shape that the data entry will have and exports the model.

#### Service

Next, we create a service function that will get the application schema from the Redis cache or from the Mongo database
in case it does not exist in the database. This function will also set the schema inside the cache for a faster fetching
the next time the schema is needed.

```ts
// src/services/dataService.ts
export const getApplicationSchema = async (domain: string): Promise<object> => {
  const key = `${domain}:schema`;
  const cacheHit = await redisClient.exists(key);

  if (cacheHit) {
    const cachedSchema = await redisClient.get(key);
    return JSON.parse(cachedSchema!);
  }

  const application = await Application.findOne({ domain }).exec();

  if (!application) {
    throw new ResourceNotFoundError(`Application ${domain} was not found.`);
  }

  await redisClient.set(key, JSON.stringify(application.template.schema));
  return application.template.schema;
};
```

Next, we create a service function that will insert the data payload inside the Mongo database.

```ts
// src/services/dataService.ts
export const createDataForApplication = async (domain: string, validData: object): Promise<DataModel> => {
  const data = { domain, payload: validData } as DataModel;
  await new Data(data).save();
  return data;
};
```

#### Controller

We can now create a controller function that will handle all the logic of the route handler.

```ts
// src/controllers/dataController.ts
export const create = async (req: Request, res: Response, next: NextFunction) => {
  const { domain } = req.params;

  try {
    const schema = await getApplicationSchema(domain);
    validateDataWithTemplate(req.body, schema);

    const data = await createDataForApplication(domain, req.body);
    const response = new ResponseBuilder()
      .withStatusCode(HttpStatus.CREATED)
      .withData(data);

    res.status(response.statusCode).send(response.build());
  } catch (error) {
    next(error);
  }
};
```

All the logic is here. We fetch the schema for the current application, we validate that the request body conforms to
the fetched schema, then we insert the data into the Mongo database and finally we respond to the requesting party
with the data that was inserted.

#### Route

Finally, we register this controller in the appropriate router with the corresponding route and method.

```ts
// src/routes/applicationRouter.ts
router.route('/:domain/data')
  .get(verifyUser, dataController.get)
  .post(jsonBodyRequired, dataController.create) // This is the line that has been added in this case.
  .all(onlySupportedMethods('GET', 'POST'));
```

And that's it. Since this route was defined inside the `applicationRouter.ts` file, the endpoint for this route will be:

```text
POST http://localhost:4000/api/applications/:domain/data
```

## Creating Documentation

The server has a [documentation site](https://freenalytics.github.io/api-docs/) which includes information of all the routes
exposed in the API.

Inside the `server` folder there is a `documentation` folder, which includes a script that generates the [OpenAPI](https://swagger.io/specification/)
specification file that will contain all the data to render the documentation site.

In order to create a new entry, you should:

1. Inside the `documentation/schemas` folder, open the file corresponding to the router where you have created your route handler.
2. Create a new `RequestSchema` (if your route requires a request body) and a `ResponseSchema` that contains the structure of the data
returned by the route handler. These schemas are [joi](https://joi.dev/api/) schemas and in this particular use case they only serve
as a way to represent the requests and responses, and are not used for any sort of validation.
3. Inside the `documentation/routes` folder, open the folder corresponding to the route of your created route handler and
create a new file with the same name as the service that your route handler executes. The idea is that these files should be descriptive
of what route exactly is it that they document.
4. Inside this newly created file, export a [RouteData](#routedata) object with all the information corresponding to your route.
5. Inside the `documentation/documentation.ts` file, import your newly created route data file and include it inside the exported object
in the `paths` object inside the corresponding object (depending on the name of the folder that you created your route data file in).
These entries are in the same order that will be displayed in the documentation site, so keep that in mind.

### RouteData

Your route data file should export an object of type `RouteData` which includes the following fields:

* `path`: The path of the route handler. (**Required** `string`)
* `method`: The method of the route handler. (**Required** One of: `get`, `put`, `post`, `delete`, `patch`)
* `summary`: A little summary of what the route handler does. (**Required** `string`)
* `description`: A more detailed description of what the route handler does. (**Required** `string`)
* `throws`: An array of the instances of the potential `HttpError` errors that your route handler may throw. (*Optional* `HttpError[]`)
* `success`: An object that describes the information of a successful response. (**Required**)
    * `success.code`: The HTTP response code that your route handler responds with on successful response. (**Required** `number`)
    * `success.schema`: The name of the `ResponseSchema` that your route handler responds with (if any). (*Optional* `string`)
    * `success.isArray`: Whether the response object is an array of `ResponseSchema` objects or not. (*Optional* `boolean` - Default: `false`)
    * `success.binaryType`: The MIME-type of the response in case the response is a binary blob. (*Optional* `string`)
* `pathParams`: An array of objects that describe each path parameter inside the route. (*Optional*)
    * `pathParams[].name`: The name of the path parameter. (**Required** `string`)
    * `pathParams[].description`: A description of what the path parameter represents. (**Required** `string`)
    * `pathParams[].type`: The type of the path parameter. (**Required** One of: `string`, `number`)
* `queryParams`: An array of objects that describe each query parameter that can be used in the route. (*Optional*)
    * `queryParams[].name`: The name of the query parameter. (**Required** `string`)
    * `queryParams[].description`: A description of what the query parameter represents. (**Required** `string`)
    * `queryParams[].required`: Whether the query parameter is required or not. (**Required** `boolean`)
    * `queryParams[].type`: The type of the query parameter. (**Required** One of: `string`, `boolean`, `number`)
    * `queryParams[].isArray`: Whether the query parameter is an array or not. (**Required** `boolean`)
* `bodySchema`: The name of the `RequestSchema` that your route handler needs as a **JSON** body (if any). (*Optional* `string`)
* `tokenRequired`: Whether the route handler requires a bearer token to be specified in the `Authorization` header. (*Optional* `boolean`)

!!! note
        If your route has `tokenRequired` set to true, you need to add a `UnauthorizedError` instance inside `throws`.

### Example Documentation

Here we'll use the same example as used above, for the route that handles data upload.

#### Schema

The relevant schemas are as following:

```ts
// documentation/schemas/application.ts
export const ApplicationDataRequestSchema = Joi.object({
  example: Joi.string().required()
});

export const ApplicationDataResponseSchema = Joi.object({
  domain: Joi.string().required(),
  payload: ApplicationDataRequestSchema
});
```

In this case, the `example` is really only an example. Since data can come in any shape or form, an example body was necessary.

#### RouteData File

The file with the information of this route will then be:

```ts
// documentation/routes/applications/createApplicationData.ts
import HttpStatus from 'http-status-codes';
import { RouteData } from '../../types';
import { UnauthorizedRequestError, SchemaValidationError, ResourceNotFoundError } from '../../../src/errors/http';

const data: RouteData = {
  path: '/applications/:domain/data',
  method: 'post',
  summary: 'Create a new data entry for an application for the requesting user.',
  description: `Create a new data entry for an application for the requesting user.
  The shape of the request body will depend on the schema that was used to create the application.`,
  throws: [
    new UnauthorizedRequestError(),
    new SchemaValidationError(''),
    new ResourceNotFoundError('')
  ],
  success: {
    code: HttpStatus.CREATED,
    schema: 'ApplicationDataResponseSchema'
  },
  pathParams: [
    {
      name: 'domain',
      description: 'The domain of the application to upload the data to.',
      type: 'string'
    }
  ],
  bodySchema: 'ApplicationDataRequestSchema',
  tokenRequired: true
};

export default data;
```

#### Including the RouteData File

Finally, we need to include this route data file into the `documentation/documentation.ts` file. Just import this file and add it into the exported object.

```ts
// documentation/documentation.ts
import createApplicationData from './routes/applications/createApplicationData';

const documentationData = {
  metadata,
  info,
  paths: {
    Application: {
      getAllApplicationsForUser,
      createApplicationForUser,
      getApplicationByDomain,
      updateApplicationByDomain,
      deleteApplicationByDomain,

      createApplicationData,
      getApplicationData,
      getApplicationDataAsCsv
    }
  }
};

export default documentationData;
```

The rest of the file was omitted as it is not relevant. Here you can see that `createApplicationData` is exported near the end of the `paths.Application` object.
