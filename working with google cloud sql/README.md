# Working With CloudSQL

#### What's CloudSQL?
It's JUST a relational database.
But instead of asking your enterprise team to manage it and updates and downtime and backups etc...
Google will happily do all that for you.
You've got 2 choices of SQL flavor, Postgres and MySQL. If you're picking one for the first time, do some research for 10 minutes on the pros and cons of each.

For the entirety of this book, I'll be using MySQL (as it's the most popular) on CloudSQL, so if I say CloudSQL, I really mean a MySQL instance managed by Google.

<hr>

#### How do I connect to it?

Well google can tell you better than I can, so have a look [here](https://cloud.google.com/sql/docs/mysql/how-to). Tons of ways to connect. Pick your favourite!


Or follow this guide, which I copied from the App Engine section a while ago, so this may be a little old. I think CloudSQL Proxy might be a better solution now.

But if you don't like those tutorials... see below.

<hr>

Create the folder `config`, which I usually place in my root directory, and add 3 files:
- `cloud-sql-connection.ts` - this is where you'll export the function you want to import later on. Other files are for setup.
- `cloudsql-credential-config.json` - this is where you can store credentials for connecting.
- `config.js` - This is a file I copied from google. I've never edited it, but it does a decent job.

<hr>

#### What do I put in them?

<hr>

`cloud-sql-connection.ts`
```typescript
require("dotenv").config();
import * as mysql from "mysql";
import * as config from './config';


export const getMySQLConnection = () => {
    let con =  mysql.createConnection({
        database: config.get('GCLOUD_PROJECT'),
        host: config.get('MYSQL_HOST'),
        user: process.env.MYSQL_USER,       // I replaced this with dotenv to load credentials
        password: process.env.MYSQL_PW,     // I replaced this too
        timezone: 'BST',                    // I added this.
        connectTimeout: 5 * 60 * 1000       // I added this. Value is Milliseconds. So I multiplied to get a value that's more obviously 5 minutes.
    });
    return con;
};
```

##### Why did you replace those?
I probably will want to share details like this file on my organisation's github, as everyone working there will know this information.

But I'm still not keen on handing out my personal password, or committing any sort of password whatsoever to github.

So I can still keep these details in `.env` file and ensure that's not committed using a `.gitignore` file.

I will import that function elsewhere to get a MySQL connection.

Timezone will default to something else, perhaps UTC. Set it however you please. You can also set the default configuration for how timestamps and dates are returned here.

Feel free to change the timeout to however long you think it'll take to do whatever you're doing.

Be wary of timeout issues.
If you try to run a query against mysql and get a connection timeout error, this may not be due to timeout at all.
It will give exactly the same error if your ip address is not whitelisted to access the database.
This was an issue I encountered with cloud functions.
The default timeout for a cloud function to run is 60 seconds.
Obviously this connection was set to 5 minutes, but the entire function was timing out after 60 seconds.
When I set the MySQL connection timeout to 20 seconds, the cloudsql function exited after 20 seconds,
suggesting that it was not connecting correctly, not that some query was taking a long time.
I whitelisted the MySQL instance for all addresses, and this function then worked.

I highly recommend not whitelisting your database to be connectable from all IP Address, for obvious security reasons.

<hr>

`cloudsql-credential-config.json`
```json
{
  "GCLOUD_PROJECT": "your-cloud-project",
  "DATA_BACKEND": "cloudsql",
  "MYSQL_HOST": "your.instance.ip.address",
  "MYSQL_PORT": "3306"
}
```

##### You didn't use the port number
Doesn't seem to matter for some reason. 3306 is the default MySQL port number. Guess it just knows.

##### Note:
When compiling your code, this file has a `.json` extension, therefore the typescript compiler will not copy it by default.
Set this up in your package.json prestart script
```json
{
  ...
  "scripts": {
    "prestart": "tsc && cp config/cloudsql-credential-config.json dist/config",
    "start": "node dist/your-compiled-file.js"
  },
  ...
}
```

<hr>

`config.js`
```javascript
// Copyright 2015-2016, Google, Inc.
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//    http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

'use strict';

// Hierarchical node.js configuration with command-line arguments, environment
// variables, and files.
let nconf = module.exports = require('nconf');
let path = require('path');

nconf
  // 1. Command-line arguments
  .argv()
  // 2. Environment variables
  .env([
    'DATA_BACKEND',
    'GCLOUD_PROJECT',
    'MONGO_URL',
    'MONGO_COLLECTION',
    'MYSQL_USER',
    'MYSQL_PASSWORD',
    'MYSQL_HOST',
    'PORT'
  ])
  // 3. Config file
  .file({ file: path.join(__dirname, 'cloudsql-credential-config.json') })
  // 4. Defaults
  .defaults({
    // dataBackend can be 'datastore', 'cloudsql', or 'mongodb'. Be sure to
    // configure the appropriate settings for each storage engine below.
    // If you are unsure, use datastore as it requires no additional
    // configuration.
    DATA_BACKEND: 'cloudsql',

    // This is the id of your project in the Google Cloud Developers Console.
    GCLOUD_PROJECT: '',

    MYSQL_USER: '',
    MYSQL_PASSWORD: '',
    MYSQL_HOST: '',
    INSTANCE_CONNECTION_NAME: '',
    // Port the HTTP server
    PORT: 5001
  });

// Check for required settings
checkConfig('GCLOUD_PROJECT');

if (nconf.get('DATA_BACKEND') === 'cloudsql') {
  checkConfig('MYSQL_HOST');
} else if (nconf.get('DATA_BACKEND') === 'mongodb') {
  checkConfig('MONGO_URL');
  checkConfig('MONGO_COLLECTION');
}

function checkConfig (setting) {
  if (!nconf.get(setting)) {
    throw new Error('You must set the ' + setting + ' environment variable or' +
      ' add it to config.json!');
  }
}

```

When you try to make a connection and run a script, if you haven't got the MYSQL_HOST variable set up, the application will exit saying you haven't set that setting.

##### Note:
When compiling your code, this file has a `.js` extension, therefore the typescript compiler will not compile it, unless you have set the option in your tsconfig.json, `"allowJs": true`

If you set that option, be sure to also add dist to exclude: 
```json
{
  ...
  "exclude": [
    "node_modules",
    "dist"
  ]
  ...
}
```

Or if you don't like that method, add some more to your prestart script:
```json
{
  ...
  "scripts": {
    "prestart": "tsc && cp config/cloudsql-credential-config.json dist/config && cp config/config.js dist/config",
    "start": "node dist/your-compiled-file.js"
  },
  ...
}
```

<hr>

You're all good to go!