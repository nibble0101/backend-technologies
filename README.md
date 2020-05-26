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
         > Enabling session support is entirely optional, though it is recommended for most applications. If enabled, be sure to use passport.session() before passport.session() to ensure that the login session is restored in the correct order.E.g.

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
         -  LocalStrategy is a class( Not sure. Need to look it up) therefore you can use it to create an instance.
     - Create an instance of LocalStrategy:
          - When creating an instance of LocalStrategy, you pass one argument which is a function. The function passed as an argument is called `verify callback`.
        ```javascript
          const localStratey = new LocalStrategy(function(username, password, done){   })   // Check whether it can be assigned to a variable
        ```
          -  The function `verify callback` takes three arguments: `password`, `username` and `done`. `done` is a callback function which is invoked with 1, 2, or 3 arguments.Below is an example of `verify callback`.
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
                  return done(null, user);
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
                   done(null, false, {message: 'Incorrect password'});
              ```
              ## **NOTE**
               > It is important to differentiate between an authentication failure (which is not a server error) and the server throwing an exception. The latter is a server exception, in which `err` is set to a non-null value. Authentication failures are natural conditions, in which the server is operating normally. Ensure that `err` remains null, and use the final argument to pass additional details about reasons for the authentication failure.
   
    - Use `app.use` to mount the strategy. An instance of your strategy is passed to `app.use`.
    
       ```javascript
           app.use(localStrategy); // localStrategy is an instance of your LocalStrategy class
       ```
    - Use `passport.authenticate`, specifying the strategy, to authenticate requests.
    
       `passport.authenticate` is a router-level middleware (Check correctness of this). The code below illustrates how it is mounted.
        ```javascript
        app.post('/login', passport.authenticate('local'), function(req, res) {
           res.redirect('/'); 
         });
       ```
        **More about `passport.authenticate`**
        
        `passport.authenticate` takes the type of authentication strategy and an optional object as arguments. The strategy passed as argument must first be configured.
        
        By default, if authentication fails, Passport will respond with a 401 Unauthorized status, and any additional route handlers will not be invoked. If authentication succeeds, the next handler will be invoked and the req.user property will be set to the authenticated user.
        
        `passport.authenticate` can be passed an object as second argument. The second argument can be used for setting `redirect` options, `flash messages`e.t.c
         - **Redirects**
         
            Redirects are usually issued after authenticating request. Below is an illustration of how to issue redirects after authentication:
            ```javascript
                app.post('/',
                              passport.authenticate('local', 
                                                               { 
                                                              successRedirect: '/',
                                                              failureRedirect: '/login'
                                                              }
                                                      )
                           
                        );
            ```
            In this case, the redirect options override the default behavior of calling the next route handler if authentication is successful or responding with a 401 Unauthorized status message if authentication is not successful. Due to the second argument to `passport.authenticate`, upon successful authentication, the user will be redirected to the home page. If authentication fails, the user will be redirected back to the login page for another attempt.
       - **Flash Messages**
       
          `failureFlash` property can be added to the object passed as second argument to `passport.authenticate` in order to display status information to the user. Below is an illustration of how.
          ```javascript
              app.post('/',
                              passport.authenticate('local', 
                                                               { 
                                                              successRedirect: '/',
                                                              failureRedirect: '/login',
                                                              failureFlash: true
                                                              }
                                                      )
                           
                        );
          ```
         Setting the `failureFlash` option to true instructs Passport to flash an error message using the message given by the strategy's `verify callback`, if any. This is often the best approach, because the `verify callback` can make the most accurate determination of why authentication failed.Alternatively, the flash message can be set specifically.
         ```javascript
              passport.authenticate('local', { failureFlash: 'Invalid username or password.' });
         ```
         A `successFlash` option is also available which flashes a success message when authentication succeeds.
         ```javscript
             passport.authenticate('local', { successFlash: 'Welcome!' });
         ```
         Is it possible ( logical ) to provide `failureFlash` and `successFlash` concurrently to the object ?
         
         **NOTE**
         
           Using flash messages requires a `req.flash()` function. Express 2.x provided this functionality, however it was removed from `Express 3.x.` ( Why was it removed if useful?). Use of [connect-flash](https://github.com/jaredhanson/connect-flash) middleware is recommended to provide this functionality when using `Express 3.x.`
      - **Disable Sessions**
      
        After successfully authenticating a user, passport will establish a persistent login. What is a persistent login? This is a situation where a user provides login credentials once. The client will again be prompted to enter login credentials after logging out. This is useful for the common scenario of users accessing a web application via a browser. However, in some cases, session support is not necessary. For example, API servers typically require credentials to be supplied with each request. When this is the case, session support can be safely disabled by setting the session option to false.
        ```javascript
            app.get('/api/users/me', passport.authenticate('basic', { session: false }), function (req, res) {});                            
        ```
      - **Custom Callback**
      
        It should also be noted that `passport.authenticate` can be called from within the route handler, rather than being used as route middleware. This gives the callback access to the req and res objects through closure.**QUESTION**: What will necessitate calling `passport.authenticate` from the route handler rather than using it as a `route middleware`? After all it will pass execution to subsequent handlers once authentication is successful. Why can't the handler wait to have access to req, res objects after succesful authentication?
        
        Similarly, if the built-in options are not sufficient for handling an authentication request, a custom callback can be provided to allow the application to handle success or failure. The callback is passed as second argument to `passport.authenticate` instead of passing an object as second argument.
        ```javascript
            app.get('/login', function(req, res, next) {
            passport.authenticate('local', function(err, user, info) {
                if (err) { return next(err); }
                if (!user) { return res.redirect('/login'); }
                req.logIn(user, function(err) {
                  if (err) { return next(err); }
                  return res.redirect('/users/' + user.username);
                });
              })(req, res, next);
          });
        ```
        
# References
1. [toon.io](http://toon.io/understanding-passportjs-authentication-flow/)
2. [node.js user authentication with passport local strategy](https://medium.com/@johnnysitu/node-js-user-authentication-with-passport-local-strategy-37605fd99715)
3. [www.passportjs.org/docs](http://www.passportjs.org/docs/downloads/html/)
     
    
    
    
         
        
    

