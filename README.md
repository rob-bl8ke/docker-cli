# Docker CLI

## Running locally

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

# Building via Docker

Its useful to have two different Docker files. One for development (`Dockerfile.dev`) and one for production (`Dockerfile`).

‼️Notice that in `Dockerfile.dev` a specific version of node is targeted to avoid bugs.

To target the `Dockerfile.dev` build, add an `-f` flag (for file) and run as below...

```
docker build -f Dockerfile.dev .
```

Here's the initial file, let's examine it:

```docker
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["npm", "run", "dev"]
```

Note that when you run `COPY . .` in `Dockerfi So the build will take some time. This isn't great. For now, one can simply remove node_modules before running the command. Try this:

```powershell
Remove-Item -Path .\node_modules\ -Force -Recurse

# Now run again...
docker build -f Dockerfile.dev .
```
Notice how quickly the build runs!

Let's run the container.

```
docker run <image id>
```

The container runs successfully, but the site cannot be reached using `http://localhost:3000` and this is because of the isolated nature of the internal network within the container. So try...

```
docker run -p 3000:3000 <image id>
```