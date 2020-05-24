# Backend Technologies
## `passport.js`
1. What is `passport.js`?
   `passport.js` according to the [documentation](http://www.passportjs.org/) is a "Simple, unobtrusive authentication for Node.js"
    **unobtrusive**: not conspicuous or attracting attention.
2. How to set up passport
   -  Require passport
      `const passport = require('passport')`
   -  Require strategy you want to use for authentication. If you want to use local strategy, then:
      `const LocalStrategy = require('local-strategy').Strategy`
      ## **NOTE**: 
      -  The local authentication strategy authenticates requests based on the credentials submitted through an HTML-based login form.For example:
      ```html
          <form action = '/login' method = 'POST'>
                <label for = 'username'> UserName: </label>
                <input type =  'text' id = 'username' name = 'username' required >
                <label for = 'password'> Password: </label>
                <input type =  'password' id = 'password' name = 'password' required >
          </form>
      ```
      -  LocalStrategy is a constructor(class- Need to look it up!). When creating an instance of LocalStrategy, you pass one argument which is a function
      -  The function passed in as an argument (called `verify callback`) to LocalStrategy takes three arguments: `password`, `username` and `done`. `done` is a callback function which also takes two
         arguments. What purpose does `done` serve?
         ```javascript
             function(username, password, done) {
                  User.findOne({ username: username }, function (err, user) {
                      if (err) { return done(err); }
                      if (!user) {
                          return done(null, false, { message: 'Incorrect username.' });
                       }
                      if (!user.validPassword(password)) {
                          return done(null, false, { message: 'Incorrect password.' });
                      }
                      return done(null, user);
                      });
                    }
            ```
       -  How does `verify callback` work?
           When passport authenticates a user, it first parses the credentials in the `request` object e.g. `request.body` It then invokes the `verify callback` with those credentials i.e. username, password. If the credentials are valid, passport then calls `done` to supply passport with the user who has been  authenticated.
           ```javascript
               return done(null, user)
           ```
       -  How does `done` work?
           `done` is a callback function which is passed to the `verify callback`. `verify callback`  calls the `done` callback supplying a `user`, which should be set to `false` if the credentials are not valid. If an exception occurred, `err` should be set. `done` can be called like: `done(err, user)`. `err` is the error object if an error occurs, `user` is the authenticated user if the user has been authenticated.
           1. If the credentials are valid, `done` is called like:  `return done(null, user)`. Note the `return` keyword. `done` supplies the valid credentials to `passport` ('Supplies valid credentials to passport'- Isn't `done` part of `passport`?) and sets the error to null(first argument to `done`).
           2. If the credentials are not valid e.g. `password` or `username` is incorrect (**NOTE**: No error has occured!), `done` should be invoked with `false` instead of `user` to show an authentication failure (not server error). 
           ```javascript
                 done(null, false);
           ```
           3. If an exception occurred while verifying the credentials (for example, if the database is not available), done should be invoked with an error, in conventional Node style like: `return done(err)`. Why isn't the second argument passed to `done`? Is it redundancy? What will happen if passed? 
           4. If there is an authentication failure, an additional info message can be supplied to `done` indicating the reason for the failure. This is useful for displaying a flash message (what is flash message?) prompting the user to try again. The message should be passed as an object (not `JSON`) with `message` key like: `{message: 'Incorrect password' }`. The value of `message` key depends on the reason for authentication failure. 
           ```javascript
                done(null, false, {message: 'Incorrect password'})
           ```
           ## **NOTE**
            > It is important to differentiate between an authentication failure (which is not a server error) and the server throwing an exception. The latter is a server exception, in which err is set to a non-null value. Authentication failures are natural conditions, in which the server is operating normally. Ensure that err remains null, and use the final argument to pass additional details about reasons for the authentication failure.
   - Create an instance of LocalStrategy
     ```javascript
        new LocalStrategy( function(username, password, done){
             //Do whatever you want here. For examle retrieving a user from a database
             
             
             done(null, null)
             })
          
     ```
    
    
    

