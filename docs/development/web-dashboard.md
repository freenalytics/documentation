# Web Dashboard Development

!!! warning
        If you wish to develop the web dashboard, please head over to the [Server Development](./server.md) page and follow the set up
        steps first. You need a local Freenalytics server when developing the web dashboard.

The [freenalytics/freenalytics](https://github.com/freenalytics/freenalytics) repository is a monorepo containing both the web dashboard and the server.

Any changes made to the web dashboard should be made in this repository.

## Requirements

In order to develop for the web dashboard you will need:

* [git](https://git-scm.com/)
* [Node.js](https://nodejs.org/en/) (Version 16.15.1 was used)

## Setting Up

First, clone the repository:

```text
git clone https://github.com/freenalytics/freenalytics
```

Head over to the `web-dashboard` folder:

```text
cd web-dashboard
```

And then, install the dependencies:

```text
npm install
```

## Setting Up the Development Environment

As mentioned before, the web dashboard requires you to have a local server running on port **4000**. If you don't have the server set up
properly yet, please visit the [Server Development](./server.md) page.

Once you have the server set up correctly, start it up with:

```text
npm run dev:watch
```

## Starting the Server

To start the development server, run:

```text
npm run dev
```

A React development server will start on port **3000**. This server has hot-reload capabilities, meaning that if you change
a file while the server is running, the changes will be displayed live on your browser.

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

This project contains unit tests for certain functionality. Visual components are currently not tested, only utilities are.

You can run the unit tests with:

```text
npm run test
```

Or, if you want to run the test suites in watch mode (will re-run relevant tests on file save), you can use:

```text
npm run test:watch
```

## Component Structure

This project follows the following structure:

* Any style sheet should be imported in the `src/index.tsx` file.
* Any `ContextProvider` should be included in the `src/App.tsx` file.
* Page components should be placed inside `src/pages/page_name/PageComponent.tsx`.
* Components used only in a certain page should be placed inside `src/components/pageComponents/page_name/component_name/Component.tsx`.
* Components used in multiple places should be placed inside `src/components/common/component_name/Component.tsx`.
* Data visualization components should be placed inside `src/components/dataVisualization/component_name/Component.tsx`.
* Form components should be placed inside `src/components/forms/form_name/FormComponent.tsx`.
* Form control components should be placed inside `src/components/common/form/component_name/Component.tsx`.

Component folder names should be in `camelCase` and component files should be in `PascalCase` and should have a `.tsx` extension.
Inside each component folder there should be a `index.ts` file that proxy exports the relevant components inside the folder.

Following this structure ensures that the code remains consistent.

Inside any component, it is recommended to organize the imports in the following way:

1. Anything from the `react` package.
2. Any React component that comes from a third-party library.
3. Any React hook that comes from a third-party library.
4. Local React components.
5. Local React hooks.
6. Anything else.

## Component Code

As mentioned before, component folder names should be in `camelCase` and component files should be in `PascalCase` and should have a `.tsx` extension.
Inside each component folder there should be a `index.ts` file that proxy exports the relevant components inside the folder.

### Example Component

Let's say we want to create a common component named `MyComponent`.

We should then create a folder named `src/components/common/myComponent` and inside this folder we will create the following files:

* `MyComponent.tsx`: This file will contain the code relevant to the component itself.
* `index.ts`: This file will proxy export the component. (More on this later.)
* `index.scss`: This file will contain any styles relevant to this component.

!!! note
        This folder may contain more components if they're closely related to `MyComponent` (for example, sub-components).
        Other files like typings can also be included in here.

!!! note
        The `index.scss` file is not required if the component does not need any custom styles.

#### myComponent/MyComponent.tsx

Let's create the component itself first.

```ts
import React, { useState } from 'react';

interface Props {
  name: string
}

const MyComponent: React.FC<Props> = ({ name }) => {
  const [count, setCount] = useState<number>(0);

  const handleButtonClick = () => {
    setCount(count + 1);
  };

  return (
    <div className="my-style">
      <p>
        Hello there {name}!
      </p>
      <button onClick={handleButtonClick}>
        You have clicked this button {count} times.
      </button>
    </div>
  );
};

export default MyComponent;
```

This is a simple component that greets the user specified with the `name` prop. It also includes a button that when clicked will update the counter
inside.

This example illustrates how `Props` should be an interface that describes the props that `MyComponent` requires, and how event handlers should
be defined inside the component with a name that begins with `handle`.

#### myComponent/index.ts

As mentioned before, this component proxy exports `MyComponent.tsx`. What this means is that this file should import `MyComponent` and then re-export it.

Why? Because this way we can have components inside their own folder and keep them inside a file with a relevant name and not with `index.tsx`. This way,
when we import this component we will use:

```ts
import MyComponent from './components/common/myComponent';
```

Instead of:

```ts
import MyComponent from './components/common/myComponent/MyComponent';
```

With that been said, the content of this `index.ts` file would be:

```ts
import MyComponent from './MyComponent';

export default MyComponent;
```

#### myComponent/index.scss

Since our component has some styles associated to it, we need to create this file.

An example of what these styles could look like is:

```scss
.my-style {
    text-align: center;

    p {
        color: red;
    }

    button {
        font-weight: 700;
        color: gray;
    }
}
```

In order to include these styles, it is necessary to add an import of this file inside the `src/styles/main.scss` file.

```scss
@import '../components/common/myComponent
```

Import order does matter and generally should be kept as:

1. Common components
2. Data visualization components
3. Form components
4. Page Components (`src/components/pageComponents`)
5. Pages (`src/pages`)

## Page Code

As with components, pages should also be located in their respective folders with the folder name being in `camelCase` and
the page component file in `PascalCase` with a `.tsx` extension. Inside this folder there should also be a `index.ts` file
that proxy exports the page component.

### Example Page

For the sake of example, let's say we want to create a page named `MyPage`.

We should then create a folder named `src/pages/myPage` and inside this folder we will create the following files:

* `MyPage.tsx`: This file will contain the code relevant to the page itself.
* `index.ts`: This file will proxy export the page component.
* `index.scss`: This file will contain any styles relevant to the page.

!!! note
        The `index.scss` file is not required if the page component does not need any custom styles.

#### myPage/MyPage.tsx

Let's create the page component itself first.

```ts
import React from 'react';
import PageWrapper from '../../components/common/pageWrapper';
import useTitle from '../../hooks/title';

const MyPage: React.FC = () => {
  useTitle('pages.my_page.title');
  const { t } = useLocale();

  return (
    <PageWrapper>
      <div className="my-page">
        This is my cool page.
      </div>
    </PageWrapper>
  );
};

export default MyPage;
```

This is a simple page that simply says "This is my cool page.".

In this example, the component `<PageWrapper>` will include the navbar and the footer with your page. If your page
should render the navbar and footer then wrap your content in this component, if not you can omit it.

!!! note
        Notice the string passed to `useTitle()`. This hook will update the title of the page with a string located
        in the `src/i18n/strings` resource folder. We still haven't seen how this project tackles [Localization](#localization)
        so keep reading, it will then be explained.

Additionally, an entry inside `src/i18n/strings/en.ts` will be inserted, inside the `PAGES` object, with the key:

```text
'pages.my_page.title'
```

And with a value that should include the title of the page, which will be then displayed in the browser's window and tab.

#### myPage/index.ts

As it is the case for regular components, page components need to be proxy exported too. This file should then look like this:

```ts
import MyPage from './MyPage';

export default MyPage;
```

#### myPage/index.scss

Since our page component has some styles associated to it, we need to create this file.

An example of what these styles could look like is:

```scss
.my-page {
    text-align: center;
    color: white;
    padding: 2rem;
}
```

In order to include these styles, it is necessary to add an import of this file inside the `src/styles/main.scss` file.

```scss
@import '../pages/myPage
```

This import should be added at the end of the file because we want to import page styles at the end.

#### Adding to the Router

Once the page component has been created, it is time to add it to the router.

Currently, all routes are located in the `src/constants/routes.ts` file and are separated by access type.
If the route is accessible by any user (whether they're logged in or not) then the route should be inside
the `PUBLIC_ROUTES` object. Otherwise, if the user should be logged in to see the page, then add the
route inside `PROTECTED_ROUTES`.

Additionally, if your route is dynamic, meaning that it has a parameter in it, then create a function inside
the `DYNAMIC_PROTECTED_ROUTES` that returns the route with the parameter applied to it.

Once this is done, we can now import the page component inside our router and add a `<Route>` that renders
this page component.

For this, enter the `src/router/Router.tsx` file and modify it to import your page component:

```ts
import MyPage from '../pages/myPage';
```

And add the route to it.

* If the page is public, then add the route like this:

```ts
<Route path={PUBLIC_ROUTES.myPage} element={<MyPage />} />
```

* If it's protected, then add the route like this:

```ts
<Route element={<ProtectedRoute allowed={loggedIn} redirectPath={PUBLIC_ROUTES.login} />}>
    {/* ... */}
    <Route path={PROTECTED_ROUTES.myPage} element={<MyPage />} />
</Route>
```

* If the route is dynamic and protected, then add the route like this:

```ts
<Route element={<ProtectedRoute allowed={loggedIn} redirectPath={PUBLIC_ROUTES.login} />}>
    {/* ... */}
    <Route path={DYNAMIC_PROTECTED_ROUTES.myPage(':param')} element={<MyPage />} />
</Route>
```

## Localization

You may have come across the weird string passed to the `useTitle()` hook in [myPage/MyPage.tsx](#mypagemypagetsx). This string is nothing more than
a key that maps to an actual string that will be displayed on the page.

Why? Localization.

In the future, this project will be available in more languages other than English. For the time being, these strings will only exist in English but
when more languages will be supported, each of these strings will need to be translated and added into their own files.

### Adding Strings

To add strings to the localizations service, open the `src/i18n/strings` folder and access the file that you wish to edit. In this case, only English
strings are available, so the file that we will update is `en.ts`.

In here, you will find multiple objects. Each object serves as a way to organize these strings by type. You can find an object for:

* Strings inside pages. (`PAGES`)
* Strings inside common components. (`COMMON`)
* Strings inside forms. (`FORMS`)
* Strings inside error messages. (`ERRORS`)

Each key should be well structured so it can clearly reflect where it is being used.

Generally, the key should have this structure:

```text
<type>.<page>.<component>.<name>.<type_of_string>
```

So, in this case, a component used in a page named `myPage` that is displayed inside a component named `myComponent` that is displayed in a button that should
say "Accept" and that is part of a group of buttons should be:

```text
pages.my_page.my_component.buttons.accept.text
```

The naming can be a bit subjective and maybe confusing, but as long as it extensively describes its usage it should be fine.

The values of these strings should be in [ICU Format](https://unicode-org.github.io/icu/userguide/format_parse/messages/). This allows for dynamic variables to be inserted,
including modifying the text in cases where plural and singular versions differ.

For example, if you want to include a variable named `name` inside a text, you can do so by defining it as:

```text
"Hello, my name is {name}."
```

### Using Strings

Using these strings inside the components is as easy as importing the `useLocale()` hook and calling the `t()` function with the key of the string to use.

```ts
import useLocale from '../../hooks/locale';

const { t } = useLocale();
t('common.my_string.text')
```

In the case that `common.my_string.text` includes a variable named `name` inside it, you can include an object in the `t()` call to include all the named
variables in the resulting text.

```ts
t('common.my_string.text', { name: 'my name' })
```

## Using the API

To use the API, a service file should be created inside the `src/services/api` folder, or use an already existing one. Services are separated by the type
of entities that are used. In this case, applications and authenticate are the only entities available, so there is only `ApplicationService.ts` and `AuthService.ts`.

These service files typically include interfaces to type the responses of the API, along with methods that call the API through Axios.

Methods are separated in two, one method that makes the API call and the other one that returns an object with a key associated to the data fetched or
mutated, and the function reference of the method that makes the API call.

Why? This project uses [TanStack Query](https://tanstack.com/query/v4) to cache the responses.

### Example Request

Let's say we want to create a `GET` request to the `/applications/:id` endpoint.

In this case, since we're talking about an `application` we should check the `ApplicationService.ts` file.

Inside, we should create two methods as such:

```ts
private async doGetApplicationByDomain(domain: string): Promise<ApplicationModel> {
  try {
    const response = await this.client.instance.get(`/applications/${domain}`);
    return response.data.data;
  } catch (error) {
    this.client.handleRequestError(error);
    throw this.client.createRequestError(error);
  }
}

public getApplicationByDomain(domain: string) {
  return {
    key: ['applications', domain],
    fn: () => this.doGetApplicationByDomain(domain)
  };
}
```

In this case, `doGetApplicationByDomain()` fetches the application data through the Axios instance in `this.client.instance` and returns the response.
The `getApplicationByDomain()` returns an object with the key that will be used to cache this response and with a function reference that will actually
fetch the data. This will be visited again in [making requests inside components](#making-requests-inside-components).

Notice how the `doGetApplicationByDomain()` method returns a `Promise<ApplicationModel>`. This is an interface defined in this same file and could be
defined as such (this is what the API returns):

```ts
export const VALID_APPLICATION_TYPES = ['mobile', 'web', 'server', 'desktop', 'other'] as const;
export type ApplicationType = typeof VALID_APPLICATION_TYPES[number];

export interface TemplateModel {
  raw_schema: string
  schema: object
}

export interface ConnectorModel {
  package_url: string
  language: string
}

export interface ApplicationModel {
  name: string
  owner: string
  type: ApplicationType
  domain: string
  template: TemplateModel
  connectors: ConnectorModel[]
  createdAt: string
  lastModifiedAt: string
}
```

It is recommended to type the responses to avoid potential bugs when consuming these services.

### Making Requests Inside Components

Components that make requests should only do requests and present the data obtained through another component who's
responsibility is to display this data.

An example of what type of components could make fetch requests would be pages since technically they shouldn't render
any visuals which should be rendered by other components instead.

Anyways, let's use the above example to make a request that will get the data for a particular application.

The page component that displays the application data should then import the following hooks and components:

```ts
import { useQuery } from '@tanstack/react-query';
import Loading from '../../components/common/loading';
import RequestErrorMessageFullPage from '../../components/common/requestErrorMessageFullPage';
import useApi from '../../hooks/api';
```

And can then make the requests inside like:

```ts
const ApplicationPage: React.FC = () => {
  const { client } = useApi();
  const request = client.application.getApplicationByDomain(domain); // Domain could be acquired through the router params with useParams().
  const { isLoading, error, data: application } = useQuery(request.key, request.fn);

  if (isLoading) {
    return (
      <Loading />
    );
  }

  if (error) {
    return (
      <RequestErrorMessageFullPage error={error as Error} />
    );
  }

  return (
    <PageWrapper>
      <MyApplication application={application} />
    </PageWrapper>
  );
};
```

Notice how in this case, this component does not render anything visual with the data, it instead renders `<MyApplication>` which in itself
is the one responsible of rendering the application data.

As for the error, if the error should be displayed in full screen then the `<RequestErrorMessageFullPage>` component should be used.
Otherwise, use `<RequestErrorMessage>`.

## Creating Forms

Forms are an essential part of any web application and they can be quite cumbersome to implement them in a clean and modular way.
In this project, the following structure is used for any form.

1. A form component named `MyForm.tsx` will do any of the requests necessary to fetch data or to upload the form data. This component
will render the next component.
2. A form logic component named `MyFormLogic.tsx` will do any sort of validation prior to sending data, it will define the structure
of the data that will be uploaded. This component will render the next component.
3. A form view component named `MyFormView.tsx` will do the actual rendering of the form.
4. An `index.ts` file that will proxy export `MyForm.tsx`.
5. A `types.ts` file that will contain the interfaces of the request bodies (if used by a service) or any other structure that needs
typing.

This ensures that the components follow a proper separation of concerns.

Any form should be placed in its own folder inside `src/components/forms`.

### Example Form

Let's check an example. Let's say we want to create a form named `MyForm`.

For this, let's create a folder inside `src/components/forms` named `myForm`.

#### myForm/types.ts

Let's define first the shape of the data that will be uploaded inside the `types.ts` file:

```ts
export interface MyData {
  name: string
  age: number
}

```

#### myForm/MyForm.tsx

Let's define the component that will do all the requests.

```ts
import React from 'react';
import { useMutation } from '@tanstack/react-query';
import Loading from '../../common/loading';
import RequestErrorMessageFullPage from '../../common/requestErrorMessageFullPage';
import MyFormLogic from './MyFormLogic';
import useApi from '../../../hooks/api';
import { MyData } from './types';

const MyForm: React.FC = () => {
  const { client } = useApi();
  const request = client.application.postMyData();
  const mutation = useMutation(request.key, request.fn);

  const handleSubmit = async (data: MyData) => {
    await mutation.mutateAsync(data);
  };

  return (
    <MyFormLogic onSubmit={handleSubmit} />
  );
};

export default MyForm;
```

This assumes a `postMyData()` method exists in `client.application` as a service. This is obviously not the case and is only here
as an illustration.

#### myForm/MyFormLogic.tsx

Now, it's time to implement the logic component that will validate that the data inserted is correct.

```ts
import React, { useState } from 'react';
import { useForm, SubmitHandler } from 'react-hook-form';
import { joiResolver } from '@hookform/resolvers/joi';
import Joi from 'joi';
import MyFormView from './MyFormView';
import useLocale from '../../../hooks/locale';
import { MyData } from './types';

interface Props {
  onSubmit: SubmitHandler<MyData>
}

const MyFormLogic: React.FC<Props> = ({ onSubmit }) => {
  const { t } = useLocale();

  const CreateMyDataSchema = Joi.object<MyData>({
    name: Joi.string().min(1).trim().required()
      .messages({
        'string.min': t('forms.my_form.errors.fields.name.min'),
        'any.required': t('forms.my_form.errors.fields.name.required'),
        'string.empty': t('forms.my_form.errors.fields.name.required')
      }),
    age: Joi.number().integer().positive().required()
      .messages({
        'number.base': t('forms.my_form.errors.fields.age.base'),
        'number.positive': t('forms.my_form.errors.fields.age.positive'),
        'any.required': t('forms.my_form.errors.fields.age.required')
      })
  });

  const form = useForm<MyData>({
    mode: 'onSubmit',
    resolver: joiResolver(CreateMyDataSchema)
  });
  const [error, setError] = useState<Error | null>(null);

  const handleSubmit = async (data: MyData) => {
    try {
      await onSubmit(data);
    } catch (error) {
      setError(error as Error);
    }
  };

  return (
    <MyFormView form={form} onSubmit={handleSubmit} error={error} />
  );
};

export default MyFormLogic;
```

Notice how the `CreateMyDataSchema` contains localized string keys in the `messages()` method. This `messages()` method
will define the validation error messages of the [Joi](https://joi.dev/api/) schema used.

In this case, inside the `src/i18n/strings/en.ts` file we should add the following strings inside the `FORMS` object:

```ts
'forms.my_form.errors.fields.name.min': 'Your name should be at least 1 character long.',
'forms.my_form.errors.fields.name.required': 'Your name is required.',
'forms.my_form.errors.fields.age.base': 'Your age should be an integer.',
'forms.my_form.errors.fields.age.positive': 'Your age should be a positive integer.',
'forms.my_form.errors.fields.age.required': 'Your age is required.',
```

Needless to say, notice the structure of these keys. We already covered on why it was important to keep a concise naming
convention for these keys.

In this case, these keys convey the following information:

1. They are used in a form because of the `forms`.
2. They are used in `MyForm` because of the `my_form`.
3. They are used in error messages because of the `errors`.
4. They are used for errors related to a fields in the form because of `fields`.
5. They are used in the `name` and `age` fields because of the `name` and `age`.
6. They represent a specific error because of the `min`, `required`, `base` and `positive`.

#### myForm/MyFormView.tsx

Lastly, we need to create the form view.

```ts
import React from 'react';
import { Box, Form, Button } from 'react-bulma-components';
import { SubmitHandler, UseFormReturn } from 'react-hook-form';
import RequestErrorMessage from '../../common/requestErrorMessage';
import useLocale from '../../../hooks/locale';
import useFormHelper from '../../../hooks/formHelper';
import { MyData } from './types';

interface Props {
  form: UseFormReturn<MyData>
  onSubmit: SubmitHandler<MyData>
  error?: Error | null
}

const MyFormView: React.FC<Props> = ({ form, onSubmit, error }) => {
  const { t } = useLocale();
  const { handleChangeNoValidation, handleBlurValidate } = useFormHelper<MyData>(form);
  const { handleSubmit, formState: { isSubmitting, errors } } = form;

  return (
    <Box>
      <form onSubmit={handleSubmit(onSubmit)}>
        <RequestErrorMessage error={error} />

        <Form.Field>
          <Form.Label>
            {t('forms.my_form.name.label')}
          </Form.Label>
          <Form.Control>
            <Form.Input type="text" name="name" onChange={handleChangeNoValidation} onBlur={handleBlurValidate} />
          </Form.Control>
          <Form.Help color="danger">
            {errors.name?.message}
          </Form.Help>
        </Form.Field>

        <Form.Field>
          <Form.Label>
            {t('forms.my_form.age.label')}
          </Form.Label>
          <Form.Control>
            <Form.Input type="number" name="age" onChange={handleChangeNoValidation} onBlur={handleBlurValidate} />
          </Form.Control>
          <Form.Help color="danger">
            {errors.age?.message}
          </Form.Help>
        </Form.Field>

        <Button.Group align="right">
          <Button color="primary" submit loading={isSubmitting}>
            {t('forms.my_form.buttons.submit.label')}
          </Button>
        </Button.Group>
      </form>
    </Box>
  );
};

export default MyFormView;
```

In this case, what ties the form control with the data it controls is the `name` prop. Notice that the `name` prop is the same as the one we defined in the
`CreateMyDataSchema`.

`handleChangeNoValidation()` and `handleBlurValidate()` are just functions exported by the `useFormHelper()` hook that allow to validate the data once the
user loses focus on the input.

This file also takes into account that the following strings need to be added in `src/i18n/strings/en.ts` inside the `FORMS` object:

```ts
'forms.my_form.name.label': 'Your Name',
'forms.my_form.age.label': 'Your Age',
'forms.my_form.buttons.submit.label': 'Submit',
```

#### myForm/index.ts

Finally, we can proxy export the `MyForm.tsx` component:

```ts
import MyForm from './MyForm';

export default MyForm;
```
