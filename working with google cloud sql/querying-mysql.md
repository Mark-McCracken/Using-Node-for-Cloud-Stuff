# Querying MySQL

This process is virtually identical to [querying DB2](../working%20with%20DB2/getting-results-all-at-once.md).

[Set up your connection files](./README.md) first, then you're ready to go.

```typescript
import {getMySQLConnection} from "./config/cloud-sql-connection";

function insertToMySQL() {
    let myQuery = `insert into myschema.mytable values (?, ?, ?)`;
    let params = [1, 2, 3];
    
    let MySQLConn = getMySQLConnection();
    MySQLConn.query(myQuery, params, (err, qResult) => {
        if (err) {
            console.error(`An error occurred running query: ${myQuery}, with params ${params}, error: ${err}`);
            
            // if using flogger-ts, synchronously
            let errObj: ReusableLog = {
                error: err,
                query: myQuery,
                additionalDetails: {
                    params: params
                }
            };
            reusableLog(errObj);
            
            return process.exit(1);
        }
        console.info(`Results correctly inserted. Details:`);
        console.info(qResult);
        MySQLConn.end();
        return;
    });
}
```
#### Note:

Getting the connection to MySQL this way seems to be synchronous. Obviously any query will be async though, using a callback.

Select Queries will come back as an array of rows just like DB2.

If you insert a single row to a table with a primary auto incrementing key, results will come back with the row number for the row inserted.
Use the result value `qResult.insertId`

If you insert loads of values at once, you'll get a summary of the number of rows affected, duplicate, edited, inserted, warnings, etc...

#### Selecting data

Works exactly like DB2, results come back as an array of objects, one object per row, all at once.
Same warnings about results that return tons of data apply.

#### Streaming Data

Unlike DB2, This comes in already as a readable stream, you don't need to set one up yourself.
It's already in object mode, so your stream will come in chunks of whole rows, exactly one row at a time, unlike that pesky [CSV reading](../working%20with%20CSVs/Reading%20Contents%20of%20a%20CSV.md).

It also comes with the fields first so you can do something with those before getting all the rows.

It can pause, similarly to DB2 if you were not to use a slightly more involved while loop to fetch new data.

##### Why would I pause it?

You can use this to stop for some reason, but you can also throttle the speed at which rows come in (say max 10 rows at any given time), should you want to slow things down if you have a lot of processing to do and don't want to overload the your program. Pretty neat!
 
Check out the [documentation](https://www.npmjs.com/package/mysql#streaming-query-rows) before you try to use this.