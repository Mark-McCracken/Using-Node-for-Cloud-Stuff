# tsconfig.json
This is where you set up your typescript compiler.

Here's an example from the project [Export-DB2-to-Avro-in-GCS](https://github.gamesys.co.uk/BI-Cognos/Export-DB2-to-Avro-in-GCS):

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "outDir": "dist",
    "declaration": false,
    "noImplicitAny": false,
    "noLib": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "lib": ["es2016", "es2017"],
    "allowJs": true,
    "target": "es2015"
  },
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```
This will probably look like the default version, but a few things I've added in are:
#### emitDecoratorMetadata
  - Enables typescript to emit information about decorators, an advanced javascript proposal. I haven't used any decorators in this project, but have in others. They're pretty useful when using well constructed frameworks. Other than that, I never write decorators myself as they're quite complicated.
#### experimentalDecorators
  - Goes with the option above. Don't know when you'd have one but not the other... Think it's something to do with dev dependencies that use the Metadata for that first option, but actual code that's produced for this option.
  
#### outDir

Set this to the location you want your files to be compiled to. I normally use dist. Without this option, files will be compiled and outputted to the same place as their typescript equivalent.

#### lib: ["es2016", "es2017"]

This enables features from es2016 and es2017 versions of javascript when compiling. If you use a new feature that your compilation target (es2015 in this case) does not have, typescript will magically polyfill* them for you. [See Javascript versions](../general-javascript-concepts/javascript-versions.md) for more info.

*(make a large function that works in place of the new version but does the same thing)

#### allowJs

Set this to also allow typescript to compile javscript files. For example, you might have some es6 javascript file you don't want to convert to a typescript file, for whatever reason. This will allow the typescript file to compile it, perhaps to work in a old browser with an older version of javascript. Not really a big deal with versions in Node, since you're not running your code on someone else's browser, it's your server and there's only one version of it. However, in any example projects with MySQL, I've allowed this, because I copied a file from a google tutorial for the setup (`config/config.js`) that I don't want to change, and I want typescript to copy that into the dist folder for me as well as any typescript files, when I compile the project, rather than me having to copy it manually.

Note, when using this, you'll almost certainly want to...

#### exclude dist

I don't want to compile files in the dist directory, those are the files I just compiled! Might end up in an infinite compilation loop. Hardly ideal...
Also exclude node modules, you don't want to comile the libraries you downlaod. This will probably come as a default if you generate a tsconfig.json automatically.
