# Git

# **DO NOT COMMIT CREDENTIALS TO GIT**

There are tons of guides out there for git and how to use it, this isn't really the place for it. Checkout these conflunce pages I made instead: [How to Git and Github](https://confluence.gamesys.corp/display/Reporting/How+to+git+and+github), and [SSH keys and linking gitbash to github profile](https://confluence.gamesys.corp/display/Reporting/Creating+SSH+key+and+linking+Git+Bash+with+Github). Tiny note: if you're on a mac, macs have an extra config file in the directory `~/.ssh/` since mac os10.12, checkout github instructions for making that work or ask me. 

What I will say is, 
- don't commit credentials or environment variables.
- don't commit your node_modules folder, other people will clone your repo, and then use `npm install` to download those files from elsewhere, so no need to overload a git server to pull those files. They're not unique to your project anyway.
- probably don't commit log files, don't need those in git
- don't commit git-type stuff
- don't commit `dist/`, it's just the compiled code that someone else can compile, and it might potentially include credentials.
- **do** commit your .gitignore file. (probably)

A .gitignore file sits in the project root directory.

Mine typically looks something like this:
```yaml
.env
volume/credentials

node_modules/
dist/

.git/
.idea/
.DS_Store
```