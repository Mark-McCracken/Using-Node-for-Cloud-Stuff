### Putting Something Into an Avro file

This is it. The big moment. You're finally going to know how to do it. We're going to lift back the curtain on this mysterious process and expose it. Maybe you'll like avro afterwards...

You'll need to grab yourself a readable stream of objects that will conform to a single shape you can describe. (Schema)

We'll get to the describing later, don't panic.

#### But I only have an array of stuff?

fine, make it into a stream like this:
```typescript
const Stream = require("stream");

interface item {
    name: string;
    quantity: number;
}
let items: item[] = [
    {
        name: "desk",
        quantity: 1
    },
    {
        name: "chair",
        quantity: 4
    }
];

let readableStream = new Stream.Readable({objectMode: true});

items.forEach(result => readableStream.push(result));
readableStream.push(null);
// to specify that the stream is finished.
```

Great work, now for the happenings to happen. 

```typescript
const avro = require("avsc");

const fileName = `items`;
const avroFileName = `${fileName}.avro`;
const outputDirectory = path.resolve(`${__dirname}/../volume/files/`);
const avroOutputFilePath = path.resolve(`${outputDirectory}/${avroFileName}`);

const schema = avro.Type.forSchema({
     name: "items",
     type: "record",
     fields: [
        {name: "name", type: ['string', 'null'] },
        {name: "quantity", type: ['double', 'long', 'null'] }
     ]
});

// make us a DUPLEX stream.
// Accepts a readable stream as input and makes a writable stream as output.
let avroStream = avro.createFileEncoder(avroOutputFilePath, schema);
                    // that's the writable stream as ouput done, with schema defined.

// take your readable stream (wherever you got it from, can be GCS or file system or websocket feed, whaterver)
readableStream.pipe(avroStream);
// and pass it to your avro stream as the input.

// Success! you've wired up both ends of your avro stream. Now what?

// Just listen for events on that stream you care about
avroStream
    .on('error', e => {
        console.error(`Error writing to file`);
        console.error(e);
        return process.exit(1);
    })
    .on('finish', () => {
        // all done!
        
        // start some process involving this file.
    });

```

You now know about streams.

#### There must be more to it than that?

Well yes and no. This really is most of the process for making avro files,
but there's a few other things to know, plus tips and tricks.

##### Compression

You can pass in options to your stream to specify that it should have compression included to make the file even smaller. Not going into details, read the library.

The [avsc library contains a lot of documentation](https://github.com/mtth/avsc/wiki), but not many examples. However do have a look.

##### What If my object structure is massive?
 
A few shortcuts. Behold...

Work out the bits and pieces a bit more dynamically:
````typescript
const schema = avro.Type.forSchema({
    name: "locale",
    type: "record",
    fields: Object.keys(items[0]).map(key => {
        return {
            name: key,
            type: switchType(results[0][key])
        }
    })
});

function switchType(key) {
    switch (typeof key) {
        case "string": return ['string', 'null'];
        case "number": return ['int', 'null'];
        default: return ['string', 'null'];
    }
}
````
Good, but not great. What if the first item has nulls all over the place?

````typescript
const schema = avro.Type.forValue(items[0]);
````
Easier, but still the same problem...

The very finest solution, is... Don't be so lazy and make the darned schema properly and design it well!

However, if you have the means, and you plan for this data to land into a MySQL table for example,
you could query the database with something like
```typescript
select * from information_schema.TABLES where TABLE_NAME = 'mytable'
```
in order to find out the exact data types of the target table, which columns are null-able, and what your schema should look like.

You could then transform those results into a dynamically generated schema in your code.

But this sort of thing is probably not always a good idea,
you'll probably want to consider very strongly how your data is structured and modelled,
should you ever want to give it to someone else.

#### Gimme more stream stuff!

You might not want to write to a file. Writing to a file only means you'll have to read from a file again at some point.

Stream it instead! For this, you can use
```typescript
const encoder = new avro.streams.BlockEncoder()
```
This is another duplex stream that you'll have to wire up at both ends.

Pipe a readable stream to it, then pipe it to a writeable stream.

This has tons of options and decoding bit and pieces, so [read the docs](https://github.com/mtth/avsc/wiki/API#class-blockencoderschema-opts).

Careful with the header option if you're adding data to an existing avro file.
By this I mean this is an entirely new readable stream you're processing, that you want to add on to an existing avro file.
I don't mean if you're halfway though the stream.