## Deserialising Avro files and doing something with them.

This was actually pretty quick. All of the avro files I've been working with contained the avro schema at the top of them.

First thing you'll need is the avsc library. You know the drill. `npm install avsc`

Then you'll need your avro file. Let's imagine it's stored locally, 

And it's actually as easy as....

```typescript
const avro = require("avsc");

const fileName = "live_GameEvent_20170506_000000";
const fileNameAvro = `${fileName}.avro`;
const localFileLocation = path.resolve(`${__dirname}/../volume/files/`);

avro.createFileDecoder(`${localFileLocation}/${fileNameAvro}`)
    .on("data", data => {
        console.log(`data chunk`);
        console.log(data);
    });
```
This will read the avro file and decode it, and start logging everything that comes out.

You'll get objects flying out to your terminal at very high speed. Feels like you're in the matrix.

<hr>

More likely, you'll want something to happen as the data comes.
Start shipping it out in batches, or transform the items. Whatever it is that you like doing with your decoded objects.
 
```typescript
let rowHolder = [];
const batchSize = 1000;
let batchCounter = 0;

console.time(`Deserialise Avro`);
avro.createFileDecoder(`${localFileLocation}/${fileNameAvro}`)
        .on('data', (chunk) => {
            rowHolder.push(chunk);
            if (rowHolder.length >= batchSize) {
                doSomethingWithArrayOfObjects(rowHolder);
                batchCounter++;
                rowHolder = [];
            }
        })
        .on('error', (err) => {
            console.error(err); // end process?
        })
        .on('end', () => {
            console.timeEnd(`Deserialise Avro`);
            doSomethingWithArrayOfObjects(rowHolder, true); // send batch that isn't full, completed boolean flag
            const totalItems = batchSize * batchCounter + rowHolder.length;
            console.log(`Total items decoded: ${totalItems}`);
            
            doSomethingElseNowThatWeAreDone();
        });
```
##### Warning:
Beware of moving stuff in batches - the file will deserialise VERY quickly.
If you're deserialising a million items, and
[shipping them in batches of 1000 to MySQL](../working%20with%20google%20cloud%20sql/inserting-in-batches.md),
the avro file will have deserialised all those object in about 10 seconds,
but the MySQL insert statements will only have run about 50 batches,
meaning you've still got 950,000 rows being held in memory.
Sound like a familiar problem yet?
Consider writing to a file then importing to MySQL if the avro file will deserialise to data in the GB.

<hr>

#### I want these streams you keep talking about

They're kinda cool looking just piping data around.

```typescript
const avro = require("avsc");

let readableAvroStream = getReadableAvroStreamFromSomewhereLikeGcs();
let writableStream = getWritableStreamFromSomewhereLikePubSubOrLocalFile();

readableAvroStream
    .pipe(new avro.streams.BlockDecoder())
    .pipe(writableStream)
    .on('end', () => console.log(`done`));
```

That's pretty much it.

Wait! You haven't written tests, and you're skipping the error handling? Come on bro...
```typescript
readableAvroStream
    .pipe(new avro.streams.BlockDecoder())
    .on("error", (e) => console.error(`error decoding block: ${JSON.stringify(e)}`))
    .pipe(writableStream)
    .on("error", (e) => console.error(`error writing to stream: ${JSON.stringify(e)}`))
    .on('end', () => console.log(`done`));
```
Little better. You may pass.