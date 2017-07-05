### Writing to GCS

Fairly straightforward, pretty much the exact opposite of download.
```typescript
const path = require("path");

const bucketName = 'my-favourite-bucket';
const bucket = gcs.bucket(bucketName);

const location = path.resolve(`${__dirname}/..volume/files/`);
const fileName = "my-file-name.txt";
const filePath = `${location}/${fileName}`;

const uploadOptions = {
    destination: gcsFileUploadName,
    gzip: false
};

bucket.upload(filePath, uploadOptions, (err, file, apiResponse) => {
    if (err) return console.error(`error in uploading to gcs, error: ${JSON.stringify(err)}`);
    
    console.info(`file ${filePath} completed uploading to bucket ${bucketName}`);
    
    // file - this is the same object returned when using bucket.getFiles, so again, you probably don't want to log it all.
});
```

#### Upload this file, but as a streaming process!

Get yourself a readable stream from wherever you wish to read from. Then,
```typescript
const localFilePath = '/Users/stephen/Photos/birthday-at-the-zoo/panda.jpg';
let myReadableStream = fs.createReadStream(localFilePath);
                        // or get a readable stream from somewhere else

const newFileName = "filename_to_upload.jpg";

let file = bucket.file(newFileName); //doesn't exist yet.
let options = {
  metadata: {
    contentType: 'image/jpeg',
    metadata: {
      custom: 'metadata'
    }
  },
  gzip: true
};

myReadableStream
  .pipe(file.createWriteStream(options))
  .on('error', err => {
      console.error(`error occurred while uploading local file at ${localFilepath} to gcs file ${newFileName}, error: ${JSON.stringify(err)}`);
      return process.exit(1);
  })
  .on('finish', function() {
    // The file upload is complete.
    // do something else now that that's done.
    // or resolve some promise, or whatever.
  });
```

<hr>

#### I don't want that file anymore now that I've uploaded it successfully

async:
```typescript
const fs = require("fs");

fs.unlink(filePath, err => {
    if (err) return console.error(`error deleting file at ${filePath}, error: ${JSON.stringify(err)}`);
    console.log(`file ${filePath} deleted.`);
})
```
or go synchronously
```typescript
const fs = require("fs");

try         { fs.unlinkSync(filePath); }
catch (err) { console.error(`error deleting file at ${filePath}`);
              console.error(err); // using synchronous code, 
            }                     // can keep this on separate console.error and will still come out in order
                                  // don't need to JSON.stringify it
```