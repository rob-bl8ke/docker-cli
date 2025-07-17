# Docker CLI

In order to exercise the following you'll need to have node.js installed. Particularly, the [Udemy course walk-through](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437046#overview) uses version 8.11.3.

You've found that you can get it to work (with deprecations) and vulnerabilities using 22.14.0

Here is the process for installing the version using `nvm`.

```bash
node -v
nvm
nvm list
nvm install v22.14.0
node -v
```

Run this command to install all your dependencies.
```bash
npm install
```

```bash
# Starts up a development server. For development use only.
npm run start

# Runs tests associated with the project
npm run test

# Builds a production version of the application
npm run build
```
