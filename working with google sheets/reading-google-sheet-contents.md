## Reading google sheets data

After you've followed the set up, and the authorisation code has all run,
you're free to run your function.

You'll need the spreadhseet ID that you want to edit.

If this is the url for the spreadsheet: (not real url)
```
https://docs.google.com/spreadsheets/d/1if3GVm1zE8xUGQxPdeguGu7_4roKwUKfe6-oOLHUCAM/edit#gid=0
```
then this bit would be the id you're after: `1if3GVm1zE8xUGQxPdeguGu7_4roKwUKfe6-oOLHUCAM`

````typescript
function myFunctionToDoGoogleSheetsStuff() {
    const sheets = google.sheets('v4'); // should have imported google already
        const config = {
            auth: auth,
            spreadsheetId: '1if3GVm1zE8xUGQxPdeguGu7_4roKwUKfe6-oOLHUCAM',
            range: 'A2:I', // this will get all values from column A to I, 
        };                 // from row 2 to the end of the sheet
        
        // sheets refers to the entire workbook, including different tabs.
        // spreadsheets means the tabs within it. You can switch to different tabs, this will land you on the default
        // The values property object is what you want. The only other option is copyTo
         
        sheets.spreadsheets.values.get(config, (err, response) => {
            if (err) return console.error('The API returned an error: ' + err);
        
            const rows = response.values;
            // response.values is a 2 dimensional array that contains the
            // actual content of the cells.
            // response could manipulate the width/height/color etc...
            // But we're in the data business, get me those numbers!
           
            let datePattern = new RegExp(/^\d{2}\/\d{2}\/\d{4}$/g);
            
            // Might for example, only want rows where the 1st column exists and 3rd column is a date.
            // You can specify in the config how a date should look.
            
            let applicableRows = rows.filter(row => row[0] && datePattern.test(row[2]));
            if (applicableRows.length == 0) return console.info('No data found.');
    
            console.log(`applicable rows`);
            console.log(applicableRows);
            // insertToMySQL(applicableRows);
            // or do whatever it is you like doing with your google sheets data.
        });    
}
````

You can then take this data, and make a readable stream from it,
batch insert into MySQL (probably won't need to worry about running out of memory for once!),
or transform it to a CSV,
email it to someone, or do whatever it is you like doing with your numbers.

The Documentation for the google sheets api is quite extensive, and you should take a look [here](https://developers.google.com/apis-explorer/#p/sheets/v4/).
