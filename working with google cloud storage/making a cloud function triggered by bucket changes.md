### Using Cloud Functions to trigger processes when a file is uploaded.

The biggest challenge with this is getting it deployed and understanding what's going on.

You need to have a function defined that takes 2 parameters, the event that triggered the function, and the callback.

However this time, instead of you specifying the callback function for what should happen when everything is finished,
it's you that will be deciding when something is finished, by calling this callback, to let the cloud function know you're done with it.

```typescript
export function cloudFunction(event, callback) {
    const file = event.data;

    if (file.resourceState === 'not_exists') {
        console.log(`file ${file.name} deleted.`);
        
        // do something if file was deleted, like send an email or text or whatever it is that you like to do when stuff gets deleted.  
        
        callback();
    } else if (file.metageneration === 1) {
        // From Documentation:
        // metageneration attribute is updated on metadata changes.
        // on create value is 1

        //Documentation on this seems to be wrong, uploading a file doesn't seem to run this code.

        console.log(`File ${file.name} uploaded.`);
        if (!file.name.includes(".avro")) {
            console.log(`not an avro file. Not interesting`);
            return callback();
        }
        doSomethingAsynchronousWithFile(file.name)
            .then(() => {
                callback();
            })
            .catch((err) => {
                callback(err);
            });

    } else {
        console.log(`file was edited. Not interested in edits.`);
        // documentation incorrect.

        console.log(`File ${file.name} uploaded.`);
        if (!file.name.includes(".avro")) {
            console.log(`not an avro file.`);
            callback(`not an avro file, process should end`);
            return;
        }
        doSomethingAsynchronousWithFile(file.name)
            .then(() => {
                callback();
            })
            .catch((err) => {
                callback(err);
            });

    }
}

function doSomethingAsynchronousWithFile(fileName: string) {
    return new Promise((resolve, reject) => {
        // create smaller copy of an image,
        // decode file to different format
        // export contents to BigQuery,
        // Your favourite thing to do with files,
        // etc...
        
        
        if (managedToDoWhatIWantedSuccessfully) {
            resolve();
        }  else {
            reject("error message");
        }
        
    });
}
```

Don't pass anything to the callback unless you want it to be considered as an error. It will be logged as such.