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
      **NOTE**: 
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
           `done` is a callback function which is passed to the `verify callback`. It 
   - Create an instance of LocalStrategy
     ```javascript
        new LocalStrategy( function(username, password, done){
             //Do whatever you want here. For examle retrieving a user from a database
             
             
             done(null, null)
             })
          
     ```
    
    
    

