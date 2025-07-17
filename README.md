# Docker CLI

[Follow the Udemy course on which this example is based on](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437040#overview)

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

# Supporting changes through volumes

#### Windows issues
💥 Important: If you're on Windows you'll have to use WSL 2 to get this running. Windows doesn't play nice with volumes because of the change in the file system. Here are some important links to help you out if you don't have WSL 2 installed:
- [Notion Notes on setting up and using WSL 2](https://www.notion.so/Using-Windows-Subsystem-for-Linux-v2-WSL-2-f28951d3411d448dbc5fb1c919c199d3) is based on the following Udemy courses:
- [The Complete WSL 2 Course for Web Development & Hacking](https://www.udemy.com/course/the-complete-wsl2-course/?couponCode=MT150725G1)
- [WSL 2, Docker, Kali Linux and Windows Terminal - get started](https://www.udemy.com/course/wsl-2-docker-and-windows-terminal/)

The course [explains a few options for Windows users](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/18799500#overview), but, to get this to use volumes effectively one must move your code to WSL2.

> [Click here](./docs/multiple-github-accounts-wsl.md) for information as to how to handle multiple GitHub accounts in WSL 2. Worthwhile to take a look at [how line-endings are treated and the correct settings to apply for Git](./docs/git-concerns.md).

### Beginning with Volumes
Making changes to the project source code at this point will not show up in the running container. In order to get these changes to reflect, a different approach is required.

Currently we copy all our source code into the Docker image. A snapshot is created and when a container is spun up, the snapshot is used.

To avoid copying over the public and src directory, we need to make use of a feature in Docker... volumes. A volume will map a folder outside to a folder inside the container.


Effectively this command will attempt to mount the present working directory to the /app directory in the container.

```bash
# Ensure this works... (get the present working directory)
pwd

# Map the pwd into the /app folder in the container...
docker run -p 3000:3000 -v $(pwd):/app "image-d"
```

---

- [You are currently here](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437066#overview) - Work from this video on Linux or a MacBook to exercise this...
---