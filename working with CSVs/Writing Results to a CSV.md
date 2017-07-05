### Writing results to a CSV

I often add the following code into a file I usually name something like `file-system-helper.ts`
```typescript
const fs = require("fs");
function makeDirectoryRecursivelySync(directory: string): void {
    let parts = directory.split("/");
    let subParts = parts.filter((_, idx, arr) => idx < arr.length);
    while (!fs.existsSync(subParts.join("/"))) {
        subParts = subParts.filter((_, idx) => idx < subParts.length - 1);
    }
    subParts.push(parts[subParts.length]);
    fs.mkdirSync(subParts.join("/"));
    while (subParts.length < parts.length) {
        subParts.push(parts[subParts.length]);
        fs.mkdirSync(subParts.join("/"));
    }
}
export function makeEmptyFileSync(path: string): void {
    let directory = path.split("/").filter((_, idx, arr) => idx < arr.length - 1).join("/");
    if (directory && !fs.existsSync(directory)) makeDirectoryRecursivelySync(directory);
    fs.writeFileSync(path, "");
}
```
### What and Why?
This makes me an empty file, synchronously. If the folder location doesn't exist, make it exist.

Once I've made this file I can happily start to append data to it.


These 2 functions will take an array of values and append them to a csv.

The first function is more of a control flow, the second function actually does the work.

Read the comments carefully to see what's going on.
```typescript
import {makeEmptyFileSync} from "./file-system-helper";
const path = require("path"); // don't need to install, comes with node
const fs = require("fs");     // same here

let csvPath = path.resolve(`${__dirname}/../volume/files/myResults.csv`);
// this path.resolve makes sure it goes in the right place
// __dirname is globally available in node, you don't need to define it, and it is whereever the file running this code exists. 


function prepareAndMoveToCSV(values: object[], completed: boolean = false, firstBatch: boolean = false) {
                                    // these paramters have default values
    if (firstBatch) {
        console.log(`Starting CSV stream`);
        console.time(`Stream To CSV`);
        makeEmptyFileSync(csvPath);
    }
    if (values && values.length) {
        addValuesToCsv(values);
    }
    if (completed) {
        addValuesToCsv([], completed);
        // don't want to send values twice, send empty array
    }
}

function addValuesToCsv (values: object[], completed: boolean = false) {
    if (values && values.length) { // only if there are any values to append
    
        //remember, all values arrive as an array of objects.
       
        let valuesToAppend = values
            .map((row, idx, arr) => {
                // if this is the last value ever to be added to csv, don't add a new line.
                // otherwise add a new line.
                let lineEnding = (completed && idx === arr.length - 1) ? "" : "\n";
                
                // this maps each object(row) to a row of a csv
                // it takes all the values of the object and makes them into a string
                // converting nulls to blank spaces
                // join them together with a comma (it's a csv after all)
                // and adding the line ending.
                return `${Object.values(row).map(column => column === null ? "" : column).join(", ")}${lineEnding}`;
            }).join("");
            // then join all the rows together into a long string that's your csv
            
            //now write to the file
            fs.appendFile(csvPath, valuesToAppend, (err) => {
                if (err) {
                    console.error(`error in appending file`);
                    console.error(err);
                    return;
                }
                // all went well if you get to here  
            });
    }
    if (completed) {
        console.timeEnd(`Stream To CSV`);
        // warning! This will stop the timer before the CSV is finished, as fs.appendFile is an async operation.
        // If this is important, can use fs.appendFileSync, or manage this another way. 
        
        // do something, like
        // loadFileIntoMySQL();
    }
}
```