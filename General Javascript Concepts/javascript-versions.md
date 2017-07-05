### Javascript Versions

Javascript is really a name by which people actually mean an ECMAScript engine run in the browser.
ECMAScript is simply a language specification that outlines what should happen under different scenarios
(What happens when you try to cast a number to a string for example, or how inheritance should work).

Javascript versions are usually referred to as es<number> 

It was invented in 1997. Version 2 was better, but for a long time the language remained on es3.
There were debates in the community about the direction of version 4, and where the language should go, and in the end, it was scrapped altogether.
 
es5 was released in 2009, with 5.1 bringing improvements in 2011.

However now, the ECMAScript board have decided to release version yearly as features are ready (gone through draft proposals and implementations and are essentially battle tested), rather than waiting many years for large upgrades.

So version 6, es6, was also named es2015. Those mean exactly the same thing.

Confusingly, a lot of people still refer to es7 to mean es2016, and es8+ to refer to es2017, but hopefully this will dissipate, and we'll stick with year numbers.


### Why do I care?

ES2015 (or es6) brought loads of new features that make programming easier. If you know some javascript, learn these features.

es2016 added a very handy method to arrays, Array.includes(<item to search for>), so I usually add this to the lib array in my typescript compiler options

es2017 adds a handy feature Object.values(<object to get all values from keys>), which is useful sometimes, so I usually add this library to my typescript compiler options too.

Those versions do have a couple of other features, but they're less notable.

Example Usage of new features:

### Array.includes
```javascript
let values = [1, 2, 3, 4, 5];

values.includes(3); //true
values.includes(6); //false
```

### Object.values

```javascript
const human = {
    firstName: "mark",
    lastName: "mccracken"
};

const fullName = Object.values(human).join(".");  // mark.mccracken

const emailAddress = `${fullName}@miceanddice.com`; // mark.mccracken@miceanddice.com
```
This one not always immediately clear when it's useful, but saves a lot of code if you remember it when you need it!

<hr>

### TypeScript

Since TypeScript is compiled before being given to a browser to run, the code you write doesn't need to be fully feature compatible with every browser, but it will still work! You can use features from es2017 and even es2018, long before those features will actually reach a browser or even a node version. This means typescript can move a lot faster than javascript for new stuff. Like Enums for example, which are a foreign concept to javascript.

Typescript is currently on version 2.4.1 (as of 4th July 2017).

<hr>

### Package Versions and Semver

If you install a package, your package.json will note the version as `1.2.3`

It may be prefixed with `~` or `^`

#### What does this mean?

Most packages will use semantic versioning, where an increase in the first number is a major new version.

Typically a major new version means breaking changes to the API.
Some things might ahve been added, removed or changed from `1.x` to `2.x`, and you should proceed with caution when making a major upgrade.
This means reading the documentation to see what changed, and what changes you might need to make in your code if you wish to upgrade to the new version.
 
An increase in the second number is a minor version.
There will likely not be any breaking changes, this kind of  upgrade usually means a new feature has been added.
For example, increasing typescript from version 2.3 to 2.4 adds support for string enums. You can pretty confidently update your code and get new features knowing nothing will break.

An increase in the last number is a patch version. This is a bug fix, and you should almost always get it as soon as possible. Obviously nobody ever intends for bugs, but reports are filed and things get fixed before new features get released, so that fix gets releases as soon as possible.

#### What about those symbols?

Your package.json will probably say "package": "^1.2.3";

The `~` will bump to latest PATCH version, `^` will bump to latest MINOR version.
If you were to reinstall all of the packages
by running `npm install`, or more likely,
you upload your code to a git repository and someone else later downloads it,
and runs `npm install`, instead of getting the exact version you saved in your repo,
it will bump the version up to the latest patch.

An example:
- Imagine there are 3 very similar packages, `a`, `b` and `c`, that always get updated at the same time on the same versions.
- You install all 3 on version 1.2.3, and upload to github
Lets say your package.json looked like this when you installed: 
```json
{
  ...
  "dependencies": {
    "a": "1.2.3",
    "b": "~1.2.3",
    "c": "^1.2.3"
  },
  ...
}
```
- The next day, a new PATCH version of the package gets released (1.2.4)
- A week later, a new MINOR version gets released, 1.3.0
- The next day, a new PATCH verions gets released, 1.3.1
- A month later, a new MAJOR version gets released, 2.0.0

- Someone then downloads your repo with that package.json, and runs `npm install`.

What they will actually install is
- a: 1.2.3
- b: 1.2.4
- c: 1.3.1


a gets exactly what was specified.
b gets highest patch version up within the current minor version.
c gets highest patch version of highest minor version.

Nothing ever moves to version 2. Breaking changes might have happened and the project might not work. Hence, there is no syntax for this.

This package numbering can be tricky for reliable builds of code though, so the package-lock.json can help with this, if you want to deploy your code to docker for example.