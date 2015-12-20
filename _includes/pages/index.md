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

```
ember g route login
```

Then let's modify the template for our template to accept user input with fields for `username` and `password`:

```hbs
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
