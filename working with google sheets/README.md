### Working With Google Sheets

#### Why?

Because not everyone is able to/wants to/allowed to
use the structured data sources that data engineers or analysts or administrators are used to deal with.

Think back to the last time you can recall someone setting up a database.
It's not (normally*) a quick and easy thing to do.

*At least it used to be. Nowadays on GCP, they've somehow got it down to about 5 clicks. Wizards...

But whipping up a new google sheet takes a matter of seconds.
It's easy, familiar, and spreadsheets have been around for eons.
The collaborative technology is also pretty awesome.

However sometimes we can end up with a very important google sheet that
hundreds of people can monitor and edit.
With data this important, it probably could be joined onto other data sources to get more value out of it.
You could join it to other sources in Tableau, but that's not always as easy as it sounds,
and then you have to consume it in tableau, and what if you don't even have tableau, or your other data sources can't connect to tableau?
(They almost certainly can, pretty much the only things tableau can't connect to are God, an DB2 using a mac.)

But it's stuck in a google sheet. 

... for now ...

Lets sort that out.

### What Do I need?

`npm i googleapis google-auth-library`

Those 2 libaries. No points for guessing what they're for, one lets you get at a google sheet or a YouTube API or whatever you enable,
and one helps you authorise stuff.
Lastly, another built in package for node, readline.
Lets you read from a readable stream, one line at a time (such as process.stdin, where you type into the console);

### How does it authorise stuff?

Well essentially, the first time this runs, you have to sit there and babysit it.
When you run the process for the first time, you'll need some credentials initially,
but you'll still need to manually approve it through your browser when the time comes.
It will provide you a url in your terminal, that you'll copy into your browser,
that'll let you authorise, and give you a code to paste into your terminal.
So in total, 2 credential files needed, plus approval.
Sounds like a lot, but this can then run in automation forever,
so we'd like to think it should be pretty secure the first time and going forward.

Get yourself some credentials using this [API wizard](https://console.developers.google.com/flows/enableapi?apiid=sheets.googleapis.com)

Choose a project, Enable the API, go to
https://console.developers.google.com/apis/credentials/wizard?api=sheets.googleapis.com&project=YOUR-PROJECT-NAME-HERE

Then go to your google cloud project, select APIs, credentials, create credentials, and pick OAuth Client ID

![Client ID Screenshot](./OAuth%20Client%20ID%20screenshot%20option.png)

Use the wizard, pick other, name it:

![Client ID Selection](./Client%20ID%20Selection.png);

Click create, you'll be redirected, Ignore that client ID and secret that Pops up,

click Download key on the right

![Download Key](./Download%20key.png)

Rename this file to `client_secret.json`, save it in your project in the folder `volume/credentials/`



#### Then what?
The following Boilerplate (code that stays the same everywhere that's required for setup) should run in your process


````typescript
const fs = require("fs");
const path = require("path");
const readline = require('readline');

const google = require('googleapis');
const googleAuth = require('google-auth-library');


let SCOPES = ['https://www.googleapis.com/auth/spreadsheets.readonly'];
// this is where you specify the level of access you need.
// If you ever need to change the access level,
// be sure to delete the credentials file stored in volume/credentials/credentials files.
// (That file won't exist yet if you're running for the first time.
// For edit access, you'd need
//   'https://www.googleapis.com/auth/spreadsheets'  instead

let TOKEN_DIR = path.resolve(`${__dirname}/../volume/credentials/`);
let TOKEN_PATH = `${TOKEN_DIR}/sheets-token.json`;
// this is what we'll use long term, which doesn't exist yet

const clientSecretPath = `${TOKEN_DIR}/client_secret.json`;
// this is what we need already, before for the first time

fs.readFile('client_secret.json', function processClientSecrets(err, content) {
    if (err) {
        console.error('Error loading client secret file: ' + err);
        return process.exit(1);
    }
    // Authorize a client with the loaded credentials, then call the
    // Google Sheets API.
    authorize(JSON.parse(content), myFunctionToDoGoogleSheetsStuff);
});

function authorize(credentials, callback) {
    var clientSecret = credentials.installed.client_secret;
    var clientId = credentials.installed.client_id;
    var redirectUrl = credentials.installed.redirect_uris[0];
    var auth = new googleAuth();
    var oauth2Client = new auth.OAuth2(clientId, clientSecret, redirectUrl);

    // Check if we have previously stored a token.
    fs.readFile(TOKEN_PATH, function(err, token) {
        if (err) {
            getNewToken(oauth2Client, callback);
        } else {
            oauth2Client.credentials = JSON.parse(token);
            callback(oauth2Client);
        }
    });
}
function getNewToken(oauth2Client, callback) {
    var authUrl = oauth2Client.generateAuthUrl({
        access_type: 'offline',
        scope: SCOPES
    });
    console.log('Authorize this app by visiting this url: ', authUrl); 
    // You'll see this printed out
    
    // At this point, you need to:
    // copy that url, paste into web browser,
    // sign in, approve usage, copy code, and
    // come back here, paste in into the command line.
    
    // If you're doing this on docker, you will need to run your image in interactive mode.
    // Add the flags -it after docker run
    
    var rl = readline.createInterface({
        input: process.stdin,  // you'll type in the input
        output: process.stdout // and see the output
    });
    rl.question('Enter the code from that page here: ', function(code) { // the code you enter will be passed here
        rl.close();
        oauth2Client.getToken(code, function(err, token) {
            if (err) {
                console.log('Error while trying to retrieve access token', err);
                return;
            }
            oauth2Client.credentials = token;
            storeToken(token); //gets saved for the future, and can refresh itself.
            callback(oauth2Client);
        });
    });
}
function storeToken(token) {
    try {
        fs.mkdirSync(TOKEN_DIR);
    } catch (err) {
        if (err.code != 'EEXIST') {
            throw err;
        }
    }
    fs.writeFileSync(TOKEN_PATH, JSON.stringify(token));
    console.info('Token stored to ' + TOKEN_PATH);
}

function myFunctionToDoGoogleSheetsStuff() {
    // do stuff.
}
````

That's a lot of boilerplate... but it's all necessary.
Hopefully if you've read the rest of this book,
it shouldn't be too much of a stretch to see what's happening.

However this is a lot of code to look at that you may not want to look at all the time.

There's no reason you couldn't ship the 3 functions, `authorise`, `getToken` and `storeToken` to another file, and call them from here.
Just make sure they have access to the correct variables for path names.