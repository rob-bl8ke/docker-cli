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


Effectively this command will attempt to map the present working directory to the app/ directory in the container.

```bash
# Ensure this works... (get the present working directory)
pwd

# Map the pwd into the /app folder in the container...
docker run -p 3000:3000 -v $(pwd):/app "image-d"
```

When you run this command you'll get a `sh: react-scripts: not found` or something similar. This is happening because we're missing an argument.

Let's understand what's going on. [This video explains](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437068#overview) that when we map the present working directory to app/ we are missing the node_modules folder. The mapped folder (on this computer) does not contain a node_modules folder because the `npm install` happened within the container and not on this computer. This is why we get this error message.

Fix this by modifying the terminal command as such...

```bash
# Map the pwd into the /app folder in the container...
# Put a bookmark on the node_modules folder
docker run -p 3000:3000 -v /app/node_modules -v $(pwd):/app "image-d"
```

Note that this flag does not have a colon because there is no mapping. With this terminal command we will successfully start the react app and can navigate to it in the browser (this includes WSL 2).

🙂 Also note that if we make a change to `App.js` and save the file it immediately shows up in the brower window. This is a function of the react engine but in order to get this to work correctly, react needs to see the changes in the folder.

# Building with Docker Compose

This is still a ridiculously long line to run to achieve this functionality. Docker Compose can greatly simplify things. Now this first attempt will fail because it looks for `Dockerfile`. We have `Docker.dev` and in any event, we will want to specifically target this file.

```docker
version: '3'
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```
We get an error like this:
``
failed to solve: failed to read dockerfile: open Dockerfile: no such file or directory
```

Let's fix it to target the file specifically.

```docker
version: '3'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

Now it should work as expected.

💥 A question you might be asking yourself is: "Do we still need the `COPY . .` step in the `Dockerfile.dev`? Strictly no, because we're now looking at files on your local file system. The instructor prefers to leave the instruction in because at some point in time in the future he might use the current file as a blueprint for setting up something in production, or decide to do away with docker-compose completely. Without `docker-compose.yml` it would not work on its own. You decide. 🙂

# Executing the tests

This should be straight forward now.

To run a specific command on the container, one appends the command at the end of the `docker run` command like so...
```
run -it f1dc56fadb3d18 npm run test
```
💥 Make sure you use the `-it` flag or you'll likely get some unexpected terminal input behavior.

### What about modifications to the tests?

Let's try and make a modification to the tests. Go and change some code in `App.test.js`. Copy the test and paste it in again (effectively duplicating it).

Hmmm.... why does this change not get picked up and trigger the tests to run again? This could be solved by:
- Attach to the existing container that is created and execute a command to start up the test suite inside a container that already has all the volume mapping set up.
- Adding another service and setting up volumes solely to run the test suite.

#### Option 1: Attaching to the existing container

Run `docker compose up` again and open up another terminal. In this other terminal run the following command...
```
exec -it 28efa7f7d1fa npm run test
```

The tests will run. If we duplicate the test and save the file the change will be picked up and the tests will run again (with both tests). Delete the duplicated test and save and the tests will be triggered and only a single tests will run.

❓Why does this happen?

But this is not the best solution because it has to be done manually and requires multiple steps.

#### Option 2: Adding another service

Let's create a second service to the `docker-compose.yml` file. Call it tests. It'll look almost the same as the first service but it won't contain the ports. It'll also override the default startup command.

```

```

Be sure to do a `docker compose up --build` (don't forget the build flag just in case). Notice now that if you make a change to your `App.test.js` it will trigger the tests to run again with the changes.

The downside is that we get all the output from the test suite from all the logging to docker compose. We don't have the ability to add any standard input/output to that container. Can't hit ENTER to get the tests to rerun etc.

But can we attach to the container?

