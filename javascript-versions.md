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


### TypeScript

Since TypeScript is compiled before being given to a browser to run, the code you write doesn't need to be fully feature compatible with every browser, but it will still work! You can use features from es2017 and even es2018, long before those features will actually reach a browser or even a node version. This means typescript can move a lot faster than javascript for new stuff. Like Enums for example, which are a foreign concept to javascript.

Typescript is currently on version 2.4.1.