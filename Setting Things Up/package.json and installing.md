### Package.json and Installing libraries
All packages can be install from npm, short for Node Package Manager.
This includes for front-end packages as well, such as react/angular (no use on a server), backend packages like things that help you manipulate the file system (no use on a browser), and packages that can work on front end and backend, such as [moment](https://momentjs.com), which helps you manipulated dates and times, which could obviously be useful in both a browser or server.

When you install node, npm comes installed with it. Likewise for Docker.

It's best to create a package.json as soon as you create an empty project.

This is an example from the project [Export-DB2-to-Avro-in-GCS](https://github.gamesys.co.uk/BI-Cognos/Export-DB2-to-Avro-in-GCS):
```json
{
  "name": "dw-to-gcs-avro",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "prestart": "tsc",
    "start": "node dist/db2-gcs-avro.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@google-cloud/storage": "^1.1.1",
    "avro-js": "^1.8.2",
    "avsc": "^5.0.4",
    "dotenv": "^4.0.0",
    "flogger-ts": "^0.3.1",
    "ibm_db": "^2.0.0"
  },
  "devDependencies": {
    "@types/node": "^8.0.7",
    "typescript": "^2.4.1"
  }
}

```

This package is used for quite a few things:
- keeping a list of dependencies so that others can download the necessary dependencies easily. (You don't typically store the dependencies with the project, as they can be large, and with this method, other people can download them for themselves)
- running common tasks
- describing the project.

Do this by navigating to your project's root directory and running `npm init -y`.
This creates an (almost) empty package.json file. `-y` flag is to say yes to all questions it will ask you about setup and accept all defaults, which you probably don't need to change.

You can install a package by navigating in your terminal to the project's root directory, and running `npm install package-name`.
You can use `npm i package-name` for short.

There are 3 kinds of install you can do:
- Project Dependency
    - Something the project needs to run in production. (default if you don't specify)
    - If you don't have a package.json, get one before trying to do this. 
    - If you have a package.json set up already and are using npm v 5+, you can just run `npm install package-name` and that will add the package automatically to your project's dependencies in the `package.json` file as default.
    - If you're using an older version of npm (don't), then you'll need either one of the flags in order to record the details in your package.json.
    - `npm install --save package-name` or `npm install -S package name`
    - If you run an install without a package.json, it will still install the files, but won't register them as there's no package.json
- Project Dev-Dependency
    - Something that you as a developer need to make the project, but isn't needed to actually run the project. For example, Typescript helps you develop the project, but to run the project you just need the compiled output, so don't need that as a project dependency
    - Use `npm install --save-dev package-name` or `npm install -D package-name`
    - Same rules as above for package.json

Both of these methods will store the package in the folder `node_modules/` in the root directory.
- Global Install
    - This is for when you want a command line tool to run in your terminal.

<hr>

### package-lock.json
If using npm version 5+ (comes with node version 8+), this file will be auto generated. Don't edit it manually. It just keeps a record of exactly where it got all of the files that you installed, so that if you try to reinstall all the packages for your project, it will get EXACTLY the same code from somewhere. Useful for docker, so that what you run on your local machine will be identical to a docker container.

<hr>

### Scripts

This section contains scripts you might want to commonly run.

npm can be used as a task runner for your project, to run tests, or start the project.

You can also use pre or post hooks, to do stuff before other things.

For example, when I run `npm start` from my terminal, that will automatically run `npm prestart` first of all, which in this instance will compile all the files first of all. (`tsc` is the command to run the typescript compiler. This will probably come with your editor, or you can globally install typescript to make it work).
For new scripts that you add in outside the defaults, you'll need to run them using `npm run scriptname`, eg. `npm run myScriptThatNpmWouldNotPickUpWithoutTheRunKeywordBeforeIt`

<hr>

### Stuff you can ignore most of the time

name, version, description, keywords, author, licence.

Only change these if you're thinking of publishing this package in some sense, to github, or npm.

Keywords is used for search purposes. The rest is pretty obvious.
