## New to Javascript?

everyone has to start somewhere... Quite literally.
If someone invented this language, they weren't born with the ability to write it. They learned. You can too. Node was only invented in 2009, so someone who says they're great at node has probably only been doing it a few years.

That being said, despite the youth of Node compared to other programming tools for the same jobs that have traditionally been used like Java, Node's strong point comes from it's simplicity, and growth and community. By simplicity, I mean it can be fairly straightforward to do tasks that would otherwise be highly complex. Checkout this code for a node web server in a few lines:
```javascript
const fs = require('fs');
const http = require('http');
let server = http.createServer(function(req, res) {
    res.writeHead(200, {'content-type':'text/plain'});
    fs.createReadStream(`ed_sheeran_at_glastonbury.m4v`).pipe(res);
});
server.listen(8888);
```
Fairly basic web server, it just sends you a video you don't want to watch. But this kind of setup would be orders of magnitude more complex using apache.

Node's growth rate means that by mid-2018, it will have surpassed java in popularity (number of developers).

The internet contains an unending supply of tutorials, free and paid, that are worth your time!

I normally recommend codecademy, but it won't do you much use in for node other than basic object manipulation.
If you want the basics of the language, do the course, shouldn't take more than a couple of hours to whip through.

However for Node and general Javascript, I really recommend 2 resources.

[Node School](https://nodeschool.io/#workshoppers) is a series of interactive tutorials that you can install on your command line.

[You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) is an amazing series of books you can read for free here, or on [safari books](https://www.safaribooksonline.com/home/).


I can't recommend Safari books highly enough either, it has resources for almost anything.

However, be cautious with publish dates on books. For Javascript, aim for 2015 or more recent, as you'll want the latest es6 features.