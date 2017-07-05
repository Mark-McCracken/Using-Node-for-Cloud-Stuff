### Reading a CSV

The methodology is essentially the opposite of writing, instead of join values together with commas and line breaks, we split a big string out.

Here's the code to do just that:
```typescript
const fs = require("fs");
const path = require(`path`);

let filePath = path.resolve(`${__dirname}/locale.csv`); //replace this with your file location

let contents = fs.readFileSync(filePath, 'utf8');
// must specify encoding!
// Otherwise you'll get a buffer stream of raw data,
// which involves an extra step to decode

// Could also do this async


let rows = contents.split("\n");

let columns = rows[0].split(",");

let rowsWithoutHeaderOrLast = rows.filter((row, idx, array) => {
    return idx !== 0 // don't want first row, that's the header
        && (row || idx === array.length - 1);
    // if the row has anything in it, or isn't the last row, keep it.
      // This means keep all the rows, but for the last one, only keep it if it's not empty
      // ie. in case your last line is just a line break.  
});

let values: object[] = rowsWithoutHeaderOrLast.map(row => {
    let value = {};
    // create an empty object
    let rowValues = row.split(",");
    // split the row out into it's values

    // for each column name (columns is an array of values)
    // get the name of the column, and what position it's in
    columns.forEach((columnName, idx) => {
        
        // use those to make a new property on the value,
        // assign it the value from the row in the same position
        value[columnName] = rowValues[idx];
    });
    return value;
    // don't forget to return, or all values will be undefined
});

console.log(`values`);
console.log(values);
```

You could do this asynchronously too if you desired.

You could also do this in a streaming fashion, but that may be more complicated, as rows may not come in nice chunks as you expect line by line.

Obviously if you have a very large csv with several GB of data, that will all need to be read into memory for this to work, which might crash your application.

In that case, you'll need to use a stream.

```typescript
const fs = require("fs");
const path = require(`path`);

let filePath = path.resolve(`${__dirname}/big-file.csv`); //replace this with your file location

let readStream = fs.createReadStream(filePath, 'utf8');

readStream.on('data', d => {
    console.log(`datachunk:`);
    console.log(d);
    // do something with data chunk
});
```

Depending on what you want to do with that data, this may be fine, or it may be complicated.

If you want to pipe this out to a writeable stream, fire on.

If you want to manually process this bit by bit, be prepared for what might happen.

![CSV not coming out in rows](./csv%20chunk%20not%20preserve%20in%20rows.png)

The data chunk here ends and starts in the middle of a row,
so be prepared to work some magic to join them together. 