# Asynchronous Javascript

Asynchronous code, is code that does not happen immediately. When you call an asynchronous function, it will get added to the javascript event queue, and then the next synchronous task will be executed immediately

For example, the following code:
```javascript
console.log(`Start`);

databaseConnection.query(`select * from table`, (err, results) => {
    console.log(`results`);   
});

console.log(`End`);
```
Will print out in the following way
```bash
Start
End
results
```

There are **many** ways to deal with this in Javascript.

You should be cautious if not entirely familiar with these concepts.

The main ways to deal with these concepts are:
- [Callback](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch2.md)
  - Create a function(a) that deals with results once a function(b) has completed.
  The function you make(a) gets called by function(b) as soon as it's finished whatever asynchronous stuff it was up to. You do this by passing function(a) as a parameter to function (b).
  Normally, this function is written inside the parameters.
  ```javascript 
  b(param1, param2, function a (error, results) {
    // do stuff with errors or results
  })
   ```
   In Node, the convention is that the first parameter to your callback function is always the error, forcing you to always consider it. Hopefully, it won't exist, and you continue on. But click the link, this summary hasn't done it justice.  
- [Promise](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch3.md)
  - This helps deal with the scenario where you have lots of asynchronous things you want to do in order,
  and the code becomes messy with callbacks. They have a lot of handy features to solve race conditions, and other things, but essentially, call a function that returns a promise, and you can deal with the values it might return immediately in your code quite neatly:
   ```javascript
    functionThatReturnsAPromise(parameter1, parameter2)
          .then((valueReturned) => {
              // promise resolved successfully, so do some good stuff.
          })
          .catch((errorReturned) => {
              // promise rejected due to an error, deal with the error.
          });
    ```
   - It's much easier to create a function that returns a promise than a function that uses a callback.
    ```javascript
    function insertValuesIntoDatabase(valuesParamter) {
          return new Promise((resolve, reject) => {
              
              databaseConnection.query(`select * from table`, (error, results) => {
                  if (error) {
                      console.log(`an error occurred:`);
                      reject(error);
                  }
                  console.log(`no errors :)`);
                  resolve(results);
            })
            
          });
    }
    ```
    Now we can call that using the following code:
    ```javascript
    insertValuesIntoDatabase(myValues)
      .then(results => {
        // this worked successfully. Use results.
      })
      .catch(error => {
        // something went wrong. handle error.
      });
    ```
    A promise can either be rejected, resolved, or still pending. Once you resolve a promise in your function `insertValuesIntoDatabase`,
    you can't then reject it, or vice versa. There's no way to re-send that message.
     However, there is a slight bug in the function `insertValuesIntoDatabase`, in that even if there is an error, it will still log `no errors :)`.
     After `reject(error)`, we should have had a line saying `return;` to stop the function progressing. Despite this, it cannot resolve successfully as it has already been rejected.

But also: 
- [Generators & Iterators](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch4.md)
  - This is a bit complicated, but instead of allowing the asynchronous function to call it's callback whenever it's ready, this lets you decide when to pull the next value.
- [async/await](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20%26%20beyond/ch8.md)
  - The latest and greatest. By far the best syntax, as this makes asynchronous code, but the code patterns look like synchronous code.
  Consider the following 3 pieces of code:
  ```javascript
  function makeDirectorySynchronously(directoryPath) {
    // do some stuff that synchronously does something, blocking other things from happening.
    // maybe throw an error if the path already exists  
  }

  function doSomethingWithFile() {
    console.log(`A`);
    try {
      makeDirectorySynchronously(someFilePath);
      console.log(`made that directory at ${someFilePath}`);
    } catch(error) {
      console.error(`wasn't able to make directory, got error: ${error}`);  
    }
    console.log(`B`);
  }

  doSomethingWithFile();
  ```
  This will log to the console `A`, then make the directorySynchronously, however long that takes, **THEN** log B to the console.
  Sort of what you'd expect.
  
  Now this:
  ```javascript
  function makeDirectoryAsynchronously(someFilePath, callback) {
    // do this asynchronously.
    // successful?
    callback(null);
    // or if an error occurred
    callback(error);
  }

  function doSomethingWithFile() {
      console.log(`A`);
      makeDirectoryAsynchronously(someFilePath, (error) => {
          if (error) {
              console.error(`wasn't able to make directory, got error: ${error}`);
              return;
          }
          console.log(`made that directory at ${someFilePath}`)
      });
      console.log(`B`);
  }

  doSomethingWithFile();
  ```
  Will instead log to the console `A`, then log to the console `B`, then at some point either error or have made the directory.
  This is out of order of what actually appears in the code.
  
  Lastly:
  ```javascript
  function makeDirectoryAsynchronously(someFilePath) {
      return new Promise((resolve, reject) => {
          // do this asynchronously.
          // successful?
          resolve(null);
          // or if an error occurred
          reject(error);
      });
  }

  async function doSomethingWithFile() {
    console.log(`A`);
    try {
      await makeDirectoryAsynchronously(someFilePath);
      console.log(`made that directory at ${someFilePath}`);
    } catch(error) {
      console.error(`wasn't able to make directory, got error: ${error}`);  
    }
    console.log(`B`);
  }

  doSomethingWithFile()
    // .then().catch()
    // we can wait on this function to complete in order to do something else once it has finished,
    // as by default, any function marked async will return a promise.
  ```
  This combines the best of both worlds. We can do something asynchronously,
  but still make it look like a synchronous function, in that it reads from top to bottom.
  Under the hood, this uses a combination of promises and generators/iterators, but the pattern essentially means that the function doSomethingWithFile **stops executing and waits for makeDirectoryAsynchronously to have finished**.
  
  This is pretty neat.
  It will print out to the console `A`, then the relevant error/success message sometime later, then pickup where it left off and log `B`;