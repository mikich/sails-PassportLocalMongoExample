#sails-PassportLocalMongoExample
This is a working example of how to get authentication working on sailsjs 0.9.3.

#Disclaimer
This example probably doesnt adhere to best-practices, and probably shouldn't be used as-is in production.. Its just meant to give you an idea of where to start.
That being said... on with it.

----

This uses the following npm libraries:

- passport
- pasport-local
- mongodb
- bcrypt
- sails-mongo

##Setup & Install
You need to have sailsjs 0.9 installed globally on your machine..

`npm install sails -g`

###Clone the repository, and cd into its folder:
I store all my web projects in `~/Web/Sites/`..

`git clone https://github.com/RelativeMedia/sails-PassportLocalMongoExample.git sails-auth-example; cd $_;`

###Run NPM Install:

`npm install`

###Install mongodb, and start it with `mongod`
I wont outline this step as it can vary from system to system. For me (on osx 10.8) it was just using brew to install

**Note:** if you want to use disk based db, change the `default: 'mongo'` to `default: 'disk'` in the  `/config/adapters.js` file.

##Start Sails
run `sails lift` in the app's directory

##Getting it Working
With the server running.

Create a User by POSTing to `/user` with the following:

- username
- password

You should get a JSON response back of the user you just created.

then try to access via GET `/user`. You should get a JSON response of "Not Authorized"

a route has been defined in `config/routes.js` to alias `POST /login` to `POST /auth/login`. So if you aren't cloning the repository
make sure you add that route.

POST to `/login` with the following fields:

- username
- password


You should get a json response of "Login Successful"

then again try to access via GET '/user'. You should get a JSON response of the users in the database (in this example, only one.)

You will notice that the password is not returned.

### Logout
Logout is now working, just simply access `/logout`

### Require authentication to access view
Add `'*':'authenticated'` to the controller object in `config/policies.js`

Example:
```javascript
AdminController: {
    '*':'authenticated'
}
```


## Enabling & Using CSRF
Sails has an API accessible via GET `/csrfToken` to obtain a CSRF token. It expects this token to be passed for all non GET requests via a parameter titled `_csrf`

To enable this modify the file `/config/csrf.js` and set `module.exports.csrf` to `true`.
Before POSTing to `/login` or `/usr` issue a `GET /csrfToken` and include that as a parameter titled `_csrf`.

Here is an ajax Example of how you could utilize CSRF with login.. i dont necessarily recommend this in production:

```javascript

//pull the CSRF token, append it to the hidden form element #csrfField
$.get('/csrfToken', function(csrf){ $('#csrfField').val(csrf._csrf); });
$('#loginForm .submit').click(function(e){
  e.preventDefault();
  $.ajax({
    url: '/login',
    type: 'POST',
    data: {
      username: $('#username').val();
      password: $('#password').val();
      _csrf: $('#csrfField').val();
    },
    success: function(d){
      console.log(d);
    }
  });
});
```

##Explanation of Files, and what they do

- `/config/passport.js` injects the custom middleware for passport into express
- `/config/policies.js` this is where you define what controllers and actions are locked down by passport
- `/config/routes.js` routes were setup for login and logout
- `/api/controllers/AuthController.js` handles login/logout
- `/api/controllers/UserController.js` handles user CRUD
- `/api/models/User.js` has a beforeCreate method which runs the plaintext password through bcrypt, also strips out the password field from json responses
- `/api/services/passport.js` the meat and potatoes of getting passport to work, this is where we find the user, and perform validation for login/logout
- `/api/policies/authenticated.js` Session validation file, makes sure users are actually logged in when needed
