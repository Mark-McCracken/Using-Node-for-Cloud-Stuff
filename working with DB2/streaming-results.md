### Streaming results from DB2

We can fetch the results one at a time and process them before the query has finished returning the results using the following technique.

```typescript
// assuming you've already set up connection

// instead of using method query,
// use queryResult
let myQuery = `select * from largeTable`;

DB2conn.queryResult(myQuery, function (err, qResult) {
    if (err) return console.error(`error in running query ${myQuery}, error: ${err}`);

    let data;
    // fetch them one at a time
    while(data = qResult.fetchSync()) {
       // do something for every row object, data
       console.log(`data: ${JSON.stringify(data)}`);
    }
    // then close connection.
    DB2conn.closeSync();
});
```

#### What can I do with those one at a time results?

Move them into batches (see below)
 
Move batches of results to MySQL.

Make a stream from them and convert them into avro format

Make a CSV from them to store locally