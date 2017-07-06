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

And that's the majority of it.

To deploy a function, you need to take all the files inside your `/dist` directory,
and zip them into a .zip file, credentials and all. Make sure nobody else has access to your staging bucket when you set this up.

Then go to [console.cloud.google.com](https://console.cloud.google.com/) and go to cloud functions.

Create a function, give it your desired memory, and time to complete

Set the trigger to be cloud storage bucket

Source code is your zip file, so upload it. Pick a stage bucket (This can be a subfolder in a GCS bucket).

Function to execute in this case is `cloudFunction`, the actual function that will be called, accepting the callback parameter.

Hit create, give it a minute to install whatever you demanded in you package.json and get ready*. Then it should be good to go. You can see any logging you've done in the cloud console.
 
*behind the scenes it's using docker to create an image, so should be a roughly equivalent time to the first time you ever try to download node on docker + npm installing your packages.