## Connecting to DB2

For all of the times I've cursed DB2 for being awkward to work with for not supporting x/y/z feature, I forgive it for have this package.

This was shockingly simple to set up, and I had run my first query within a matter of a few minutes reading the docs.

You essentially just need to run `npm install ibm_db` and add one setup file and the connection details, and you're off the the races.


Generally, it's easier to store the connection function somewhere else,
so I usually store mine in a file, `config/db2_connections.ts` that looks like this
```typescript
require("dotenv").config();
// sets up environment variables in file .env in root

const ibmdb = require('ibm_db');

export function getDWConnection() {
    try {
        let conn = ibmdb.openSync(`DATABASE=${process.env.DATABASE_NAME};HOSTNAME=${process.env.DATABASE_HOSTNAME};UID=${process.env.DB2_USER};PWD=${process.env.DB2_PW};PORT=${process.env.DB2_PORT};PROTOCOL=${process.env.DB2_PROTOCOL}`);
        return conn;
    } catch (e) {
        throw e;
    }
}
```

And you're pretty much done!

<hr>

#### A few notes:
- When importing this at the top of another file, your editor might do this for you.
Type `getDB` and it may autocomplete the rest of the method, and if you hit enter, it might even add the import statement for you!
- This is a synchronous operation.
When you call it from a script,
you need to be aware that it might throw an error,
that you should be prepared to catch. 
   You could do something like this:
   ```typescript
   import { getDB2Connection } from "./config/db2_connections"; 
   // at the end of this file path, don't use .ts for the extension.
   // when the typescript is compiled to javascript, javascript will know where to find the file without the extension  

   let DB2Conn;

   try { 
     DB2Conn = getDB2Connection();
   } catch (err) {
     console.error(`unable to connect to db2, got error: ${err}`);
   }
   ```
   Then later try to use it (in a different file):
   ```typescript
 
   if (DB2Conn) {
      let query = `select * from table`;
      DB2Conn.query(query, (err, results) => {
         if (err) return console.error(`error occurred running query ${query}, error: ${err}`);
         // do something with results;
      });
   } 
   ```
   You shouldn't get an error in here about connecting.
   
- Beware of running a long running query or something that takes a long time. You may get a connection timeout. You might need to look into how to extend the connection timeout for your setup. However I haven't come across this issue yet with DB2.

- You could make this operation asynchronous. Here's an example using a promise that will resolve if successful or reject if an error: 
   ```typescript
    const ibmdb = require('ibm_db');
    
    export function getDB2Connection(): Promise<any> {
        return new Promise((resolve, reject) => {
            ibmdb.open(`DATABASE=${process.env.DATABASE_NAME};HOSTNAME=${process.env.DATABASE_HOSTNAME};UID=${process.env.DB2_USER};PWD=${process.env.DB2_PW};PORT=${process.env.DB2_PORT};PROTOCOL=${process.env.DB2_PROTOCOL}`, (err,conn) => {
                if (err) reject(err);
                
                // remember, you can't reject and resolve a promise, it will do the first one it comes across.
                // this function isn't doing anything else, so we don't need to return after rejecting with an error,
                // we don't need to worry about it being resolved.
                
                resolve(conn);
            });
        });
    }
   ```
   But then be aware that you'll need to call is asynchronously, using `.then` and `.catch`
   ```typescript
   getDB2Connection()
      .then(connection => {
          // do stuff with your connection
          let myQuery = `select 'it worked' from sysibm.sysdummy1`;
          connection.query(myQuery, (err, results) => {
              // typical error handling needed, but won't involve initial connection error.
              if (err) return console.log(`error occured running query: ${myQuery}, error: ${err}`);
              //do stuff with results.
          });
      })
      .catch(error => {
          console.error(`an error occurred getting a connection to DB2: ${error}`);
          // more error handling
      });
   ```


<hr>

### Doing it the cool way with async/await

Here's an example setup:

`config/db2_connections`
```typescript
const ibmdb = require('ibm_db');

export function getDWConnection(): Promise<any> {
    return new Promise((resolve, reject) => {
        ibmdb.open(`DATABASE=${process.env.DATABASE_NAME};HOSTNAME=${process.env.DATABASE_HOSTNAME};UID=${process.env.DB2_USER};PWD=${process.env.DB2_PW};PORT=${process.env.DB2_PORT};PROTOCOL=${process.env.DB2_PROTOCOL}`, (err,conn) => {
            if (err) reject(err);
            resolve(conn);
        });
    });
}
```
and usage

`index.ts`
```typescript
async function readDB2() {
    let DB2Conn;
    try { DB2Conn = await getDWConnection(); } catch (e) { console.log(`error retrieved connecting to db2`); return; }
                                                                                      // don't want to continue without connection, so return;
    let query = `select * from sysibm.sysdummy1`;

    DB2Conn.query(query, (err, qResult) => {
        if (err) return console.error(`error returned running query: ${query}, error: ${err}`);
        
        // do stuff with query results 
        
    });

}
```
Looks sync, but works async :)