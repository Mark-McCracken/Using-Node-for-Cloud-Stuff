# Working with Avro files

#### What's an avro file?

This is first question you should ask as it's not immediately obvious, and what someone explains what it is, it still isn't clear.

A few definitions:
- `Serialise` or `Encode`:      Transform a javascript object into a format that can be stored and moved and saved between programs
- `Deserialise` or `Decode`:    Take a file, or a readable input stream of serialised data, and convert that into a javascript object in memory.

### What does that mean?

Well for Javascript, there is the JavaScript Object Notation (JSON) format.
This is an object that can be used in your javascript program, that might look like this
```typescript
const human = {
    name: "mark",
    age: 25,
    male: true,
    speak: function() {
        console.log(`${this.name.toUpperCase()} YELLED HELLO`);
    }
}
```
We can serialise most data like this, except for functions, which are actually a callable object in javascript.

We can use `JSON.stringify(human)` to get an output like `'{"name":"mark","age":25,"male":true}'`.

With the exception of the function which has not been serialised as it can't be, all of the data has been serialised.

**`JSON.stringify(object)`** is a `serialisation` function, that will take your javascript object and encode it in JSON format.

We can then use 
```typescript
JSON.parse(`${someLongJsonStringOrContentsOrApiResponse}`);
```
to decode this, and get back a javascript object that we can manipulate again like any other object.

### What does this have to do with Avro?

Well, it's actually exactly the same process, but on steroids, for a few reasons:
- It's much smaller in size
- It's much faster
- It seems much harder!

#### How is it smaller in size?

##### 2 Reasons:
###### 1:
Let's say you want to encode an array of objects in javascript. (Typescript here)
```typescript
interface item {
    name: string;
    quantity: number;
}
const items: item[] = [
    {
        name: "desk",
        quantity: 1
    },
    {
        name: "chair",
        quantity: 4
    }
]
```
When you encode this in JSON format, you get
```typescript
'[{"name":"desk","quantity":1},{"name":"chair","quantity":4}]'
```
But the words name and quantity are repeated.

Not so with Avro. It has a schema definition to tell you exactly the shape of the objects in the file.

There are 2 options.
- Have a file with the schema written at the top, known as the header. 
- Have a file without the schema at the top, and have the schema stored separately.

We're going with the first one option.


###### 2:
It's encoded to byte code. This means whenever you serialise it, it will not be human readable like that JSON object we encoded earlier.

It actually looks like this:
```text
Objavro.schema˙{"name":"Person","type":"record","fields":[{"name":"name","type":"string"},{"name":"age","type":["null","int"],"default":null},{"name":"gender","type":{"name":"Gender","type":"enum","symbols":["FEMALE","MALE"]}},{"name":"address","type":{"name":"Address","type":"record","fields":[{"name":"zipcode","type":"int"}]}}]}avro.codecnull •læ∏ÜÒ∏Ÿd›4…%óÙrîÓ	
oYBuz˛ ‰
xKMgdHyLw  ‡MPPsYun ÷
XTrqŒög  æynx  öxPFZ  ≤bRHCLEwdglb∫ˆ(UVcDVhxpyCziyBSiRasp  j•læ∏ÜÒ∏Ÿd›4…%ó
```
Nice, right?

So what have we got here. Well the header is stored at the top of the file, meaning we can actually read that bit. But the data obviously appears as jibberish to mere mortals like us.

This can be approximately 80% smaller in size compared to JSON (obviously varies wildly with the amount and kind of data).

<hr>

## What makes it harder?
Well, with that JSON serialisation, you can just give it an object and it's serialised, job done.

Not so fast with avro, You need a schema to define how that file should be serialised, to full explain what the contents of the file will look like.

It's a bit like if you had to instead say 
```typescript
let schema = {
    name: 'items',
    type: 'record',
    namespace: 'my.company',
    fields: [
        {name: "name", type: ['string', 'null']},
        {name: "quantity", type: ['double', 'long', 'null']}
    ]
};
let objectToSerialise = {
    name: "table",
    quantity: 1
};
console.log(JSON.stringify(schema, objectToSerialse));
```
But what happens if you try to feed in the wrong type? Errors obviously!



## Why on earth go to the pain of doing this?

Notice that the fake schema above has curious types.
If you're familiar with Javascript you'll know there's no such type as doubles or long, only number.
But if you're familiar with databases, this might look super intuitive what those mean*.
So avro can provide what people call "Rich Data Structures".
Doesn't mean an awful lot to me, just that it's well structured and 
you're not gonna end up with some sort of data item that doesn't conform to the schema, as it couldn't have gotten in there with conforming to the schema.

*Assuming you're so nerdy you know the difference between a double and a real

You might have to move data from a location where internet bandwidth is prohibitively expensive,
or you might want to store data in BigQuery in the most optimal format.

Whatever you think of the format, solutions architects think it's amazing and we should all use it.

Developers seem to have real headaches getting it to do what they want,
probably because the format is relatively new, and the documentation can sometimes be sparse.
This was probably the most difficult thing in this book for me to do,
my first avro file took 6 hours of trial and error and googling and reading and debugging and crying in the corner to create.
Admittedly 5 of those hours were crying in the corner.* 
However now that I've managed it once, it seems like less of a bad idea for developers.

Hopefully if you've read this far, you'll understand it a little more than you would from googling otherwise.



<hr>

*jokes aside, it really was 6 hours. Lack of working examples was difficult, hopefully this book will help.