### Inserting into MySQL in batches

This involves making a query string with all of the values you've got, then running that insert statement.

```typescript
function insertToMySQL(applicableRows: object[]) {
    let queryString = createQueryString(applicableRows);
    let conn = getMySQLConnection();
    conn.query(queryString, (err, qResult) => {
        if (err) {
            console.error(`An error occurred inserting values: ${err}`);
            console.error(`queryString`);
            console.error(queryString);
            return process.exit(1);
        }
        console.info(`Results correctly inserted. Details:`);
        console.info(qResult);
        conn.end();
        return process.exit(0);
    });
}

function createQueryString(applicableRows: object[]): string {
    let insertHeader = `insert into myschema.mytable (column1, column2, column3) VALUES`;
    let values = applicableRows.map(row => {
        let string = `(`;
        string += Object.values(row).map((value) => {
            if (value == null || value === "") return 'NULL';
            if (!Number.isNaN(+value)) return +value;           
                // if it can be converted to a number, must be a number
                
                // be sure to check the number possibility first.
                // If these results came from another database or google sheets,
                // numbers may have been returned as strings.
                // Check if you can make them numbers before assuming it's a string    
                
            if (typeof value === "string") return `"${value}"`;  // wrap any strings string in quotes.
            return value;
        }).join(', ');
        string += `)`;
        return string; // this will look like:  (1, "my string", NULL)
    }).join(', ');  // this will look like:  (1, "my 1st string", NULL), (2, "my 2nd string", NULL), (3, "my 3rd string", NULL)
    
    // optional merge statement if you want to merge new data in without duplication.  
    let mergeStatement = ` ON DUPLICATE KEY UPDATE
                          column1 = VALUES(RELEASE_DATE),
                          column2 = VALUES(GAME_NAME),
                          column3 = VALUES(PRODUCT_TYPE)`;
    return `${insertHeader} ${values} ${mergeStatement}`;
}
```