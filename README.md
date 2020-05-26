# Backend Technologies
## `passport.js`
1. What is `passport.js`?

   `passport.js` according to the [documentation](http://www.passportjs.org/) is a **Simple, unobtrusive authentication for Node.js**
   
      **Unobtrusive**: Not conspicuous(clearly visible) or attracting attention.
2. How do you set up passport?
     -  Require passport
     
         `const passport = require('passport')`
     - Use `app.use` to mount application-level middleware which intializes `passport`. 
        ```javascript
          passport.use(passport.initialize());
          passport.use(passport.session());
          ```
          Only `passport.initialize()` is necessary for initializing `passport`. If your application uses persistent login sessions,  
          `passport.session()` middleware must also be used.

         **NOTE**
         > Enabling session support is entirely optional, though it is recommended for most applications. If enabled, be sure to use     
           passport.session() before passport.session() to ensure that the login session is restored in the correct order.E.g.

        ```javascript
              app.use(session({ secret: "cats" }));
              app.use(passport.initialize());
         ```
         The above doesn't make sense. Look it up.
     -  Require strategy you want to use for authentication. If you want to use local strategy, then:
     
         `const LocalStrategy = require('local-strategy').Strategy`
         ## **NOTE**: 
         -  In real-world applications you `require` at the top of the application.
         -  The local authentication strategy authenticates users using a username and password. Username and password are also called **credentials**. These credentials are submitted through an HTML-based login form. For example:
         ```html
             <form action = '/login' method = 'POST'>
                   <label for = 'username'> UserName: </label>
                   <input type =  'text' id = 'username' name = 'username' required >
                   <label for = 'password'> Password: </label>
                   <input type =  'password' id = 'password' name = 'password' required >
             </form>
         ```
         -  LocalStrategy is a class( Not sure. Need to look it up). When creating an instance of LocalStrategy, you pass one argument which is a function. The function passed as an argument is called `verify callback`.
     - Create an instance of LocalStrategy:
        ```javascript
          const localStratey = new LocalStrategy(function(username, password, done){   })   // Check whether it can be assigned to a variable
        ```
          -  The function `verify callback` takes three arguments: `password`, `username` and `done`. `done` is a callback function which is invoked with 1, 2, or 3 arguments.
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
          
              When passport is authenticating a user, it first parses the credentials in the `request` object e.g. `request.body` It then invokes the `verify callback` with those credentials i.e. username, password. If the credentials are valid, passport then calls `done` to supply passport with the user who has been  authenticated.
              ```javascript
                  return done(null, user)
              ```
          -  How does `done` work?
          
              `done` is a callback function which is passed to the `verify callback`. `done`'s function signature is: `done(err, user)`. `verify callback`  calls `done` supplying a `user` if the credentials are valid, setting `err` to `null`. Set user to `false` if the credentials are not valid, and `err` to `null`. If an exception occurred, `err` shouldn't be null. It should be `err`.`err` is the error object if an error occurs, `user` is the authenticated user if the user has been authenticated. This is further summarized below:
              1. If the credentials are valid, `done` is called:  `return done(null, user)`. Note the `return` keyword. `done` supplies the valid credentials to `passport` (**'Supplies valid credentials to passport'**- Isn't `done` part of `passport`?) and sets the `err` to `null`.
              2. If the credentials are not valid e.g. `password` or `username` is incorrect (**NOTE**: No error has occured!), `done` should be invoked with `false` as second argument instead of `user` to show an authentication failure (**NOT server error**). 
              ```javascript
                    done(null, false);
              ```
              3. If an exception occurred while verifying the credentials (for example, if the database is not available), done should be invoked with an `error` i.e. first argument shouldn't be `null`. It should be `err`. In conventional Node style you can call it like: `return done(err)`. Why isn't the second argument passed to `done`? Is it redundancy? What will happen if passed? 
              4. If there is an authentication failure, an additional info message can be supplied to `done` indicating the reason for the failure. This is useful for displaying a flash message (**What is flash message?**) prompting the user to try again. The message should be passed as an object (**NOT `JSON`**) with a `message` key: `{message: 'Incorrect password' }`. The value of `message` key depends on the reason for authentication failure. 
              ```javascript
                   done(null, false, {message: 'Incorrect password'})
              ```
              ## **NOTE**
               > It is important to differentiate between an authentication failure (which is not a server error) and the server throwing an exception. The latter is a server exception, in which `err` is set to a non-null value. Authentication failures are natural conditions, in which the server is operating normally. Ensure that `err` remains null, and use the final argument to pass additional details about reasons for the authentication failure.
   
    - Use `app.use` to mount the strategy. An instance of your strategy is passed to `app.use`
    
       ```javascript
           app.use(localStrategy); // localStrategy is an instance of your LocalStrategy class
       ```
    - Use `passport.authenticate()`, specifying the strategy, to authenticate requests.
    
       `passport.authenticate` is a router-level middleware (Check correctness of this). The code below illustrates how it is mounted.
        ```javascript
        app.post('/login', passport.authenticate('local', { failureRedirect: '/login' }), function(req, res) {
           res.redirect('/'); 
         });
       ```
        **More about passport.authenticate**
        
        
     
     
    
    
    
         
        
    

