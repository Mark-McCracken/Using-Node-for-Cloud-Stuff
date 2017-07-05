# Setting Things Up
One of the most daunting things about starting a new language or project is understanding how to set it up, so hopefully this folder will help out with that.

<hr>

### Javascript and Node

Different Browsers have slightly different javascript engines. For more info on versions, see [javascript versions](../General%20Javascript%20Concepts/javascript-versions.md).

Node (or node.js) is a Javascript Runtime that executes the Chrome Browser's V8 engine, but can run outside the browser (ie. on a server).
This capability of running outside the browser means it can do things your browser can't, like accessing the local file system. (Browsers cannot do this for obvious security reasons).

The benefit of this is that a lot of javascript written for the web will work similarly on the server. Lots of web APIs are written in Javascript to make interactions with the browser easy, so therefore these APIs are available to node as well.

You can install node locally on your computer from the organisation website, [node.org](https://nodejs.org/en/). Version 8.1.3 is the current version, seems to be working fine for me.

There's also a Docker build which isn't really any more complicated once you understand it a bit. See the [Docker](./Docker.md) section.

<hr>

### TypeScript

All of the examples I've written use [TypeScript](https://www.typescriptlang.org).

Typescript is a superset of Javascript. This means it can do anything Javascript can do plus a bit more. It was invented by Microsoft (Yeah, I didn't know they ever made good things either!) and is open source.

The main benefit is in the name: Types.
They keep your code a lot easier to understand when reading/reviewing it,
and any decent editor will warn you if you've tried to do something silly,
like assign the value "mark" to a variable age that you've specified should be a number.
Obviously that's a simple example, but when you start to work with complex data types, it can be very easy to notice that you've returned a single item, when you maybe meant to return an array of items. Typescript helps reduce that problem.

On top of this, it needs to be compiled, taking whatever you've created in a typescript file, and outputting that as a javascript file.

The compiler set up will go in a file called `tsconfig.json` in your project's root directory.

Hopefully once you've seen it once, that's enough to not really need to touch it again.

See details in the [tsconfig.json](./tsconfig.json.md) section.

<hr>

Virtually all projects you'll want to make will need a [package.json](./package.json%20and%20installing.md) and a [tsconfig.json](./tsconfig.json.md), so be sure to check those out.

### Environment Variables
This is a concept made fairly straightforward by the package dotenv

run `npm install dotenv` (make sure you've set up your package.json first)

create a file in your root directory called `.env`

Put the contents into it

```yaml
DB2_USER=myusername
DB2_PW=paSsw0rD
GCLOUD_PROJECT=mygooglecloudproject
```

as early as possible in your script, add the line
```javascript
require("dotenv").config();
```
You can now access these variables using
```javascript
process.env.DB2_USER
```
or whatever you named them