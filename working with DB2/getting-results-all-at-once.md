### Getting all results from DB2

Set up your connection as usual

`index.ts` (or whatever you like to name your file)
```typescript
import { getDB2Connection } from "./config/db2_connections"

let DB2Conn;
try { DB2Conn = getDB2Connection(); } catch(err) { console.error(`error returning connection. Error: ${error}`); process.exit(1); }
//synchronous approach

let query = `seleeeeeect * from mytable`;

// although we've got the connection synchronously, this will still perform asynchronously with a callback.
DB2Conn.query(query, (err, results) => {
    if (err) return console.error(`error returned running query ${query}, error: ${JSON.stringify(err)}`);
    
    // results are returned as an array of objects (even if you only get one row back!!)  
    results
        .map(item => item.COLUMN_NAME)                  // do some conversion
        .filter(rowValue => rowValue > 1000)            // get rid of some you don't want
        .forEach(row => executeAnotherFunction(row)); // do something with everything that's left 
});

```
The Above should hopefully throw an error, telling you that you have a SQL Syntax error.

Fix that, try again, and you're off to the races!

To run this process, best way is to ensure your `prestart` and `start` scripts are set up correctly in your [package.json](../Setting%20Things%20Up/package.json%20and%20installing.md), and from the root directory in your terminal, type `npm start`

<hr>

#### Warning
If you try to run a query that returns 20GB of data, your application will most likely crash as it will run out of memory.

If you want to move large results, you'll need to [stream the results](./streaming-results.md).

Consider exporting them to a csv or streaming them to somewhere else directly.