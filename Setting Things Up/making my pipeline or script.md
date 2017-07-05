# Making my script or pipeline

Pipeline sounds much cooler.

There's a pretty large chunk of setup at the top of almost every file I've made.

You can copy this (please understand it first) for a setup with synchronous logging

```typescript
require('dotenv').config();
const fs = require("fs");
const path = require('path');

// might not need this unless you're making your own readable stream.
const Stream = require("stream");

import {currentDateString,
        ReusableLog, LoggingConfig,
        Colors, ConsoleAllColor,
        redirectLoggingToFilesSync,
        createReusableLoggerSync} from "flogger-ts";
import {makeEmptyFileSync} from "./fs-helper";
const currentDate = currentDateString();
const generatePathLocation = (suffix: string) => {
    if (suffix === "log") return path.resolve(`${__dirname}/../volume/logs/${currentDate}.log`);
    return path.resolve(`${__dirname}/../volume/logs/${currentDate}.${suffix}.log`);
};
const logPath = generatePathLocation('log');
const infoPath = generatePathLocation('info');
const errorPath = generatePathLocation('error');
const loggingConfig: LoggingConfig = {
    log:   { location: logPath,   printToTerminal: true },
    info:  { location: infoPath,  printToTerminal: true },
    error: { location: errorPath, printToTerminal: true }
};
redirectLoggingToFilesSync(loggingConfig);
const reusableLogsLocation = path.resolve(`${__dirname}/../volume/logs/reusable_logs`);
const reusableLog = createReusableLoggerSync(reusableLogsLocation);
declare const console: Console & ConsoleAllColor;

// import packages relevant to this script, eg:

import {getDB2Connection} from "./config/dw_config";                // for DB2
import {getMySQLConnection} from "./config/cloud-sql-connection";   // for MySQL
const avro = require("avsc");                                       // for Avro
const storage = require("@google-cloud/storage");                   // for GCS
const google = require('googleapis');                               // for google services, inc sheets
const googleAuth = require('google-auth-library');                  // for google services, inc sheets



// now get to work

DB2Conn = ...
```

#### That's a bit long.

Deal with it. But you can make that flogger-ts import shorter, by using
```typescript
import * as flogger from "flogger-ts"
```
Then every time you want to use one of those methods you need to prefix it with flogger, like
```typescript
flogger.redirectLoggingToFilesSync(loggingConfig);
```



<hr>


# Async Setup

```typescript
let path = require('path');
import {currentDateString, currentTimestampString,
    ReusableLog, LoggingConfig, Colors, ConsoleAllColor,
    redirectLoggingToFiles,
    ReusableLogger,
    removeEmptyLogFiles} from "flogger-ts";

// note these methods don't have the word sync at the end. They're all async.
// Anything written to log files might not be in order. Careful how you go...

let currentDate: string = currentDateString();
let generatePathLocation = (suffix: string) => {
    if (suffix === "log") return path.resolve(`${__dirname}/volume/logs/async-logs/${currentDate}.log`);
    return path.resolve(`${__dirname}/volume/logs/async-logs/${currentDate}.${suffix}.log`);
};
let logPath = generatePathLocation('log');
let infoPath = generatePathLocation('info');
let errorPath = generatePathLocation('error');
let loggingConfig: LoggingConfig = {
    log:   { location: logPath,   printToTerminal: true },
    info:  { location: infoPath,  printToTerminal: true },
    error: { location: errorPath, printToTerminal: true }
};
redirectLoggingToFiles(loggingConfig);

// provide types for colored Logging
declare const console: Console & ConsoleAllColor; //even works for warn

let reusableLogLocation = path.resolve(`${__dirname}/volume/logs/async-logs/reusable_logs`);
let reusableLogger = new ReusableLogger(reusableLogLocation);
```


### Why would I want an async setup?

Logging is not a free operation.

If you have a stream of incoming data, and you're logging it all synchronously, it might not be a lot to log, but this will stop the next operation from proceeding. Even logging to the terminal is not entirely free.

Don't beliive me? Try decoding a very large file and printing it to the terminal as you go.
Then try it again without the logging. Should be a lot faster.

This is increased when you want to log to a file as well.

Asynchronous logging still takes time, but it doesn't stop your program from adding the next item to process to the event queue.

People often say Javascript is single-threaded. What they mean is you don't have to deal with the concept of concurrency directly, by managing what's considered as important and threads of work being done. You just have x amount of stuff to do and it's done when it's done.

This is that's not technically true, javascript can do advanced stuff, you just don't see it happening. Intelligent people have huge debates about this, but lets not get hung up on it.
 
 ### What does it mean for me?
 
If you want to go the way of async, put any logging all in one console.log/info/error/warn call, and it will all come out together, like this:
```typescript
console.error(`error received running query: ${query}, got error: ${JSON.stringify(err)}, this is bad`);
```
Will yeild
```bash
error received running query: select sum(member_id) from members, got error: {"error":"Arithmetic Overflow","sqlstate":25024,sqlcode:24524}, this is bad
```

But if you're using async logging and instead do this:
```typescript
console.error(`error received running query`);
console.error(query);
console.error(`got error:`);
console.error(err);
console.error(`this is bad`);
```
This might come out in the wrong order.
Worse, if you're running many async operations like processing lots of rows, these logs could be interlinked with logs from another row, making them almost indistinguishable.

#### Why did you use `JSON.stringify(error)` ?

If you add an object into a string and try to print it using this code:
```typescript
console.error(`error received running query: ${query}, got error: ${err}, this is bad`);
```
it will come like this
```bash
error received running query: select sum(member_id) from members, got error: [object Object], this is bad
```
Which obviously you can't read. Use JSON.stringify to serialise it.