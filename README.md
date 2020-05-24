# Backend Technologies
## `passport.js`
1. What is `passport.js`?
   `passport.js` according to the [documentation](http://www.passportjs.org/) is a "Simple, unobtrusive authentication for Node.js"
    **unobtrusive**: not conspicuous or attracting attention.
2. How to set up passport
   -  First require passport
      `const passport = require('passport')`
   -  Require strategy you want to use for authentication. If you want to use local strategy, then:
      `const LocalStrategy = require('local-strategy').Strategy`
      **NOTE**: 
      -  LocalStrategy is a constructor(class- Need to look it up!). When creating an instance of LocalStrategy, you pass one argument which is a function
      -  The function takes in three arguments: `password`, `username` and `done`. `done` is a callback function which also takes two
         arguments. What purpose does `done` serve?
    
    
    

