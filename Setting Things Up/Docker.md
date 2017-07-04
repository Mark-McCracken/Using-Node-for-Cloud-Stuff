# Using Docker

Docker containers are all the rage.

The technology represents a major shift in the way software can be developed, without so much need for devops.
Infrastructure teams just look after the core server hosting the docker service,
but developers can specify whatever they want in a container.

Want to use node? Cool.

Prefer Python? Fine.

Like Go? Good.
 
Got experience with Swift & iOS? Use it.

Your code can be whatever language you can find a base image for, and runs in it's own almost sandboxed container.

As long as it does what you say it does (don't skip tests!!), it's all good in the hood.

So how do we get node up and running on a docker?

### What is a docker?
Tricky to explain for the first time, but essentially a docker host is a server that runs the docker daemon.
This basically manages containers and everything that docker does.

You would log onto this server, and create an image file.

#### What's an image file?
This is a recipe for a container
#### What's a container?
Stop interrupting me.

A container is what will actually run.
It consists of all the code required to execute whatever it is that you've coded,
from the basic operating system, to the packages you want to install, to your actual code.

But here's the catch. A container only runs for as long as it's needed, then it dies.

In some cases, as long as it's needed happens to be forever, like a web server,
so this will run indefinitely. But in some other cases, this will be a process you want to run,
which unless it was written by Neil Stone, will hopefully not take forever, and will end.

When it ends, the container dies, along with any files in it.*

*Except for Volumes.
If you attach a volume to a container, this is exactly the same concept as plugging a USB stick into a laptop.
If the laptop dies, still got that USB with whatever you put on it.
Anything that was on that stick (or volume here), you can read it when your laptop (container) starts.
Laptop dies (container ends), stick still has all the data (volume remains with whatever changes your container made to it).


So back to your prior question, 
#### What's an image file?
This is basically your code all wrapped up and ready to be launched as a container at an instant's notice.

You build your files into an image. Kind of like compiling them.

You run that image, and a container is spawned to do your bidding:
- web server
- data transformation
- create alchemy campaign
- web scraping
- Manipulate FTSE to make Millions in BitCoin

Whatever you're into.

#### Ok, so how do I make one of these images?

Use a dockerfile. Make a new file in your project's root directory called `Dockerfile`

This is where you can specify how to build your image for the stuff you need.

The format is yaml, so comments are anything after `#` to the end of that line.

Here's one I made earlier:
```yaml
# use the node build. This will be pulled from the docker-hub.
FROM node:latest
# :latest can be replaced with other maintained versions,
# but this means I'll get 8.1.3 of node atm.
# The latest build will obviously change though as new version of node are release,
# so be cautious using latest if you're using an APIs in node marked as anything other than stable.

ENV http_proxy="<insert http proxy address for outbound http requests>"
ENV https_proxy="<same but for https requests>"
ENV someOtherEnvironmentVariable=value
# these value will be available in the process's environment variables.
# I prefer not to store credentials in here, because if we do, and for the sake of convenience,
# we want to share this on say, github,
# WE CANNOT COMMIT CREDENTIALS TO GIT. EVER.
# So I use dotenv package.


# make a directory to hold the stuff. (-p flag makes parent directories as necessary)
RUN mkdir -p /usr/src/app

# move into this directory. Same as cd in your terminal
WORKDIR /usr/src/app

# copy your package.json into that location
COPY package.json /usr/src/app

# this will install all packages required by your package.json
RUN npm install

# copy anything else from your local folder to the image.
COPY . /usr/src/app/

# run npm start when the image starts
CMD ["npm", "start"]
```
A few notes on why:
- putting things in the folder usr/src/app
   - No idea. Found this on the node setup example from their docs. Stuck with it.
- why not just move over all the files straight away?
   - docker makes use of everything being on one docker host, so saves memory by building things in layers.
   - Having 10 docker containers running with the node environment, does not necessarily mean I will be running something 10 times the size.
- why copy package.json then other files? Why not all at once?
   - odds are, when developing a project, your packages don't change all that often. Not nearly as often as your source code.
   - Docker builds the image in layers, so every command in this dockerfile, is essentially run on top of the previous command's image.
   The previous command has it's own image that we can build on.
   This is how a docker host can be more efficient than a single VM.
   - We just want the package.json, because the next command will install all the packages.
   - The command after that copies in the files.
   If our packages.json has not changed since the last build,
   docker can basically pick up from the middle fo this dockerfile,
   as the image for everything up to `RUN npm install` already exists.
   Don't need to wait to download packages again.
- won't copying `.` (current directory, meaning root) copy the node_modules too? Meaning we'd have 2 sets?
   - Not if you do the smart thing and use a ...
   
### .dockerignore
Imagine a .gitignore files but for docker. You now understand .dockerignore files*.

*as much as you understoof .gitignore files. If you didn't know much, see [here](./git.md).

Bit less worring about credentials though, we will actually need them to run our code after all.

Here's one I made earlier
```yaml
node_modules/
```
(finding that file was more trouble than preparing it in advance saved me).