### Importing a CSV into MySQL in Batch

If you have a huge dataset you want to get into mysql, first thoughts might be along the lines of:
 - read the dataset
 - create insert statements
 - run them in batches
 
Sounds fine in theory, but there might be bottleneck of creatings those insert statements where a lot of processing is needed.

You can read the file very very quickly, and you can run an insert statement fast enough, but the processing might take time,
and the connection setup for every query is a lot of overhead to run thousands of times, where a SQL server will need to check that you have permissions to insert into a table, every time you do it.

#### How to get around this?

We can simply take the whole file, and send it to the database with a query for how to insert it, and we're done*. 

What does the syntax for this look like?

```typescript
const path = require("path");
const fs = require("fs");
import { getMySQLConnection } from "./config/cloud-sql-connection"

let csvPath = path.resolve(`${__dirname}/../volume/files/big-file.csv`);
let MySQLConn = getMySQLConnection();

let tableInsertStatement = `LOAD DATA LOCAL INFILE "${csvPath}"
INTO TABLE ${myTableName}
CHARACTER SET utf8
FIELDS TERMINATED BY ","
LINES TERMINATED BY "\\n" 
`;

MySQLConn.query(tableInsertStatement, (err, results) => {
    if (err) {
        console.error(`Results unable to be inserted, got error: ${err}`);
        
        // if you're using the flogger-ts package:  
        let errObj: ReusableLog = {
            error: err,
            query: tableInsertStatement
        };
        reusableLog(errObj);
        MySQLConn.end();
        return process.exit(1);
    }
    console.log(`results:`);
    console.log(results);
    try {
        fs.unlinkSync(csvPath); //delete csv, don't need it now.
    } catch (e) {
        console.error(`Unable to delete csv file at ${csvPath}`);
        console.error(err);
    }
    MySQLConn.end();
});
```

#### What if I want blank columns to be considered as NULL for some columns in my table?

We can make that happen... with some playing around.
Essentially we need to write a slightly longer SQL statement to specify what should happen when we come across a column that's empty.
 
We specify the columns that we're inserting, give them a name, and specify they should be null if the value in the csv is whatever value you give it.

```typescript
let tableInsertStatement = `LOAD DATA LOCAL INFILE "${myFilePath}"
               INTO TABLE ${myTable}
               CHARACTER SET utf8
               FIELDS TERMINATED BY ","
               LINES TERMINATED BY "\n" 
               
               (@v0, @v1, @v2, @v3, @v4, @v5, @v6, @v7, @v8, @v9)
               SET MEMBER_ID = nullif(@v0, ''),
                   LOCALE_ID = nullif(@v1, ''),
                   DEVICE = nullif(@v2, ''),
                   OTHER_COL_1 = nullif(@v3, ''),
                   OTHER_COL_2 = nullif(@v4, ''),
                   OTHER_COL_3 = nullif(@v5, ''),
                   OTHER_COL_4 = nullif(@v6, ''),
                   OTHER_COL_5 = nullif(@v7, ''),
                                      
                   
                   MODIFIED = nullif(@v9, '')`
                   

```

#### What did you do with column @v8?

Well that one was a timestamp, and it can't be null. This is a pain. How do we know which columns can and can't be null?

Here's one I wrote earlier.

```typescript
let mySQLconn = getMySQLConnection();
let tableInsertFromCsv = `LOAD DATA LOCAL INFILE "${csvPath}"
           INTO TABLE ${tables.mysql}
           CHARACTER SET utf8
           FIELDS TERMINATED BY ","
           LINES TERMINATED BY "\\n" 
           `;
mySQLconn.query(`show columns from ${tables.mysql}`, (err, results) => {
    // find all the column data for the table I want to insert into
    
    if (err) {
        console.error(`error getting table schema for ${tables.mysql}`);
        console.error(err);
        return process.exit(1);
    }
    
    // add on that (@v0, @v1, @v2, ...) string, for the correct number of columns.
    // Assuming we're matching exactly what's in the csv with the destination table.
    tableInsertFromCsv += `(${results.map((_, idx) => `@v${idx}`).join(", ")}) SET `;
    
    tableInsertFromCsv += results
    // for column, if that column can be null, then append the code that makes the column null, with the correct index  
        .map((item, idx) => item.Null === "YES" ? `${item.Field} = nullif(@v${idx}, '')` : "")
        // one of these would look like "Member_ID = nullif(@v0, '')", or else just ""
        .filter(item => item !== "")
        // filter out any items that are just "".
        // we don't need them, and they'll cause a syntax error when we try to join them together,
        // like Member_ID = nullif(@v0, ''), , Device = nullif(@v2, '')
        // that comma in the middle is wrong so remove it
        .join(", ");
        // make me one big string out of all these for each result.
        
        

    console.log(`Starting to load into mysql`);
    mySQLconn.query(tableInsertFromCsv, function (err, result) {
        if (err) {
            console.error(`Results unable to be inserted, got error ${JSON.stringify(err)}`);
            
            // if using flogger-ts ...  
            let errObj: ReusableLog = {
                error: err,
                query: tableInsertFromCsv
            };
            reusableLog(errObj);
            
            
            mySQLconn.end();
            console.log(`Failed`);
            return process.exit(1);
        }
        console.log(`results:`);
        console.log(result);
        try {
            fs.unlinkSync(csvPath);
        } catch (e) {
            console.error(`Unable to delete csv file at ${csvPath}`);
            console.error(err);
        }
        console.log(`finished`);
        mySQLconn.end();
    });
});
```