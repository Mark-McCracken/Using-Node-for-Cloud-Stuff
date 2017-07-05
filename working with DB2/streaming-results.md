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
 
[Move batches of results to MySQL]()

[Make a stream from them and convert them into avro format]()

[Make a CSV from them to store locally]()

### Moving results in batches

Using the above method, can setup things to move in batches as soon we we've received the results.

I haven't tested the most optimal number, but I choose 1000 because that's 10 times bigger than 100. Choose however many you like, but 1000 seems to work well.

```typescript
let myQuery = `select * from largeTable`;

DB2conn.queryResult(myQuery, function (err, qResult) {
    if (err) return console.error(`error in running query ${myQuery}, error: ${err}`);
    
    let data;
    let batch = [];
    let batchCounter = 0;
    const batchSize = 1000;
    let totalRows;

    while(data = qResult.fetchSync()) {
        batch.push(data);
        if (batch.length >= batchSize) {
            // do something with the batch
            insertIntoMySQL(batch);
            //empty the batch
            batch = [];
            //increase the counter
            batchCounter++;
        }
    }
    // all results now received.
    DB2conn.closeSync(); // or async, doesn't matter.
    
    // there may still be some values in a not full batch...  
    insertIntoMySQL(batch);
    
    totalRows = batchSize * batchCounter + batch.length;
    console.info(`Total Rows: ${totalRows}`);
});
```

### Moving results as a stream
You can put your results into a readable stream the can be fed elsewhere (eg to an avro file or to gcs or whereever you find a writable stream to pipe it to.)

```typescript
//other setup...
const Stream = require(`stream`); // don't need to install this, comes with node

let readableStream = new Stream.Readable({objectMode: true});

DB2conn.queryResult(myQuery, function (err, qResult) {
    if (err) return console.error(`error in running query ${myQuery}, error: ${err}`);
    
    let data;

    while(data = qResult.fetchSync()) {
        readableStream.push(data);
    }
    // all results now received.
    DB2conn.closeSync(); // or async, doesn't matter.
    
    readableStream.push(null); //to indicate stream is complete.
});
```
You can then do something like pipe this to process.stdout or anywhere that accepts a writable stream, such as writing to file system, writing to avro, or writing to Google Cloud Storage.
```typescript
let writableStream = getWritableStreamFromSomewhere();

readableStream
    .pipe(writableStream)
    .on(`finish`, () => {
        //do something with completed written stream.
    });
```