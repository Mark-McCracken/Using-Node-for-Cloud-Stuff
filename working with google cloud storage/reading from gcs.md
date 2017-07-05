# Reading from GCS


#### Show us what's in your bucket!

```typescript
import {currentTimestampString} from "flogger-ts";

const bucketName = 'my-favourite-bucket';
const bucket = gcs.bucket(bucketName);

// This is a typescript interface. It just defines an object structure.
// This helps developers from making type-based errors.
// It's not required and I invented this interface pretty quickly.
interface GcsFile {
    name: string;
    bucket: string;
    project: string;
    created: string;
} 


bucket.getFiles({}, (err, files, apiResponse) => {
                if (err) {
                    console.error(`error in reading files in bucket ${bucketName}, error: ${JSON.stringify(err)}`);
                    console.error(`Full API response`);
                    console.error(apiResponse); // this contains loads of stuff.
                    return;
                }
                console.log(`Total files: ${files.length}`);
                
                // Be careful about logging out all the files.
                // You might expect logging all the files to return an array of strings, to show what's in there, like
                // files = ['myfile1', 'myfile2'];
                // Sounds nice, but this is NOT THE CASE!!
                // Each items returned per file is a VERY VERY large JSON object. over 1600 lines to traverse if you print it out.
                // If you're curious, just print out the first one.
                // The following filters out to show only certain files, and maps that enormous object to a much much smaller one,
                // Which conforms to the interface desribed above
                
                let eligibleFiles: GcsFile[] =
                    files.filter(item => item.metadata.contentType === "binary/octet-stream")
                        .map((file): GcsFile => ({
                                name: file.name,
                                bucket: file.bucket.id,
                                project: file.bucket.parent.projectId,
                                created: currentTimestampString({date: new Date(file.metadata.timeCreated)})
                            })
                        ).filter(fileIsEligible);

                console.log(`Total Eligible Files: ${eligibleFiles.length}`);

                eligibleFiles.forEach((file: GcsFile) => {
                    
                    // do something with that file...
                    
                });
        });
```

### Gimme that file

Download a whole file into memory. You know the drill, don't let it get towards GB in size or program can run out of memory.
```typescript
const fileName = "my-favourite-file.avro";
const bucket = gcs.bucket('my-favourite-bucket');
const file = bucket.file(fileName);

file.download((err, contents) => {
    if (err) return console.error(`couldn't download file, got error: ${JSON.stringify(err)}`);
    // do something with contents, which are held in memory
});
```
Or write it a local destination.
```typescript
const fileName = "my-favourite-file.avro";
const bucket = gcs.bucket('my-favourite-bucket');
const file = bucket.file(fileName);

const localLocation = path.resolve(`${__dirname}/../volume/files/${filename}`);

const options = {
    destination: localLocation
};

file.download(options, (err) => {
    if (err) return console.error(`couldn't download file, got error: ${JSON.stringify(err)}`);
    console.log(`File Completed download to location`);
});
```


### Gimme that file, SLOWLY!

Here's how you can create a file stream from a File in GCS

```javascript
const path = require("path");
const fs   = require("fs");
const avro = require("avsc");

const bucket = gcs.bucket('my-favourite-bucket');
const fileName = "my-favourite-file-in-that-bucket";
const fileNameAvro = `${fileName}.avro`;

const localFileLocation = path.resolve(`${__dirname}/../volume/files/`);

const remoteReadStream = bucket.file(fileNameAvro).createReadStream();

remoteReadStream
    .on('error', (err) => { console.error(`an error occurred: ${err}`); }) // can stop the process at this point?
    .on("response", (response) => { console.info(`Server Response: ${response}`); })

    // decode it into objects if avro file 
    .pipe(new avro.streams.BlockDecoder())
    
    // write that to a file
    .pipe(fs.createWriteStream(`${localFileLocation}/${fileNameAvro}`))
    .on("end", () => {
       console.log(`file finished downloading and decoding and writing`); 
    });
```
Obviously if you're just writing to a file then this isn't much to talk about, but you have at least decoded it.

You could if you wanted, pump this stream to a writeable stream anywhere, for example, back to google cloud storage,
or out to some other upload location you're keen on, or you could singularly decode each item and transform them and move them on the fly.

For example:
```typescript
remoteReadStream
    .on('error', (err) => { console.error(`an error occurred: ${err}`); }) // can stop the process at this point?
    .on("response", (response) => { console.info(`Server Response: ${response}`); })

    // decode it into objects if avro file 
    .pipe(new avro.streams.BlockDecoder())
    .on(`data`, d => {
        // do something with this decoded object.
        // increment some counter or insert to mysql or whatever it is this file contains information for.
    }).on(`end`, () => {
       // do something when it has all been processed. 
    });
```