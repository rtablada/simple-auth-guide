---
layout: page
title: Getting Started
---

## Installation

To get started with Ember Simple Auth, we will need to install Ember Simple Auth in our Ember CLI project:

```sh
ember install ember-simple-auth
```

## Setup

Now to work with simple auth, start by setting up the Application Route.
If this is a new project, we can generate the application route:

```sh
ember g route application
```

Then let's modify this new file:

```js
// app/routes/application.js
import Ember from 'ember';
import ApplicationRouteMixin from 'ember-simple-auth/mixins/application-route-mixin';

export default Ember.Route.extend(ApplicationRouteMixin);
```

My setting up this application route, our application will now capture the `authenticationSucceeded` and `invalidationSucceeded`.
These events are fired when a user successfully logins or logins.
By default this will redirect to a route named `index` if the user logs in.

If the current project has another route listed as the `path: '/'`, we will need to configure the `routeAfterAuthentication` setting in the `ember-simple-auth` config (more on this in the recipes) section.

## Logging In Users

Ember Simple Auth allows our app to communicate back and forth with our API to login our users.
To do this, we'll start by creating a login route:

```sh
ember g route login
```

Then let's modify the template for our template to accept user input with fields for `username` and `password`:

```handlebars
<!-- app/templates/login.hbs -->
<h2>Login</h2>

<form {{action "loginUser" username password on="submit"}}>
  {{input placeholder="Username" value=username}}
  {{input placeholder="Password" type="password" value=password}}

  <button>Login</button>
</form>
```

Before writing an action handler for `loginUser`, we will need to define an authenticator for our application to use.
Authenticators describe to Ember Simple Auth how to connect to a particular authentication API.
For this example, we'll use the built in OAuth2 password authenticator.
We'll need a new file `app/authenticators/application.js`:

```js
// app/authenticators/application.js
import OAuth2PasswordGrant from 'ember-simple-auth/authenticators/oauth2-password-grant';

export default OAuth2PasswordGrant.extend();
```

Now we can go into our route handler to create our `loginUser` action handler to login our user:

```js
// app/routes/login.js
import Ember from 'ember';

export default Ember.Controller.extend({
  session: Ember.inject.service(),

  actions: {
    loginUser(email, password) {
      this.get('session').authenticate('authenticator:application', email, password).catch((reason) => {
        console.log(reson);
      });
    },
  },
});
```

We inject the Ember Simple Auth session into our route using `Ember.inject.service`,
then retrieve it in our action handler with `this.get('session')`.
On this session object, we'll call `authenticate` which takes three arguments:
the named of the authenticator we are using (`authenticator:application`), the email, and password of our user.
Finally, we'll use ES6 promises `.catch` to handle any server or session errors that my occur during the login process.

Before we can use our login strategy, we will need to return to our application authenticator
and define our `serverTokenEndpoint` based on our API's token endpoint.
For this example, we'll connect to an API endpoint an `http://localhost:3000/oauth/token`:

```js
// app/authenticators/application.js
import OAuth2PasswordGrant from 'ember-simple-auth/authenticators/oauth2-password-grant';

export default OAuth2PasswordGrant.extend({
  serverTokenEndpoint: 'http://localhost:3000/oauth/token',
});
```

Now our user can login to our server using their username and password.

## Protecting routes

In every application, there are certain routes that should be blocked from guest users.
To require a user to be logged in to access a route, we can use the `AuthenticatedRoute` mix in:

```js
// app/routes/index.js
import AuthenticatedRouteMixin from 'ember-simple-auth/mixins/authenticated-route-mixin';

export default Ember.Route.extend(AuthenticatedRouteMixin);
```

Now if we try to visit the home page with the user logged out, we will be redirected to the login page instead.

## Displaying Session Info in Components

To show the information from the current session, we have to inject the `session` service into a component.
For this example, we'll create a `top-nav` component using `ember g component top-nav`:

```js
// app/components/top-nav.js
import Ember from 'ember';

export default Ember.Component.extend({
  session: Ember.inject.service(),
});
```

Now we can use the `isAuthenticated` property of the session to show different menu items to guests vs logged in users:

```handlebars
<!-- app/templates/components/top-nav.hbs -->
<nav>
  <ul>
    {{#if session.isAuthenticated}}
      {{!-- These are only shown to logged in users --}}
      <li>{{#link-to "admin.dogs"}}Dogs{{/link-to}}</li>
      <li>{{#link-to "admin.cats"}}Cats{{/link-to}}</li>
    {{else}}
      {{!-- This is only shown to guest users --}}
      <li>{{#link-to "login"}}Login{{/link-to}}</li>
    {{/if}}
  </ul>
</nav>
```

## Logging Out Users

While most API implementations will time out user sessions, we also want to allow our user to log themselves out.
For this, let's add an link with a click action to our `top-nav` component:

```handlebars
{{!-- app/templates/components/top-nav.hbs --}}
<nav>
  <ul>
    {{#if session.isAuthenticated}}
      {{!-- These are only shown to logged in users --}}
      <li>{{#link-to "admin.dogs"}}Dogs{{/link-to}}</li>
      <li>{{#link-to "admin.cats"}}Cats{{/link-to}}</li>
      <li><a href="#" onclick={{action 'logout'}}>Logout</a></li>
    {{else}}
      {{!-- This is only shown to guest users --}}
      <li>{{#link-to "login"}}Login{{/link-to}}</li>
    {{/if}}
  </ul>
</nav>
```

Then we will need to handle this new `logout` action in our component.
Here we will run the `invalidate` method on the Ember Simple Auth session:

```js
// app/components/top-nav.js
import Ember from 'ember';

export default Ember.Component.extend({
  session: Ember.inject.service(),

  actions: {
    logout() {
      this.get('session').invalidate();
    },
  },
});
```

Now if our app is logged in, we should be able to click on the "Logout" button to invalidate the current session.
Then our app will reload and we should be redirected to the `login` route.
