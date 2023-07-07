# Get Started with Docker and Visual Studio Code

## Part One - Create a Container App

Source:

<https://docs.microsoft.com/en-us/visualstudio/docker/tutorials/docker-tutorial>

### Prerequisites

- Visual Studio Code
- Docker VS Code Extension
  - <https://code.visualstudio.com/docs/containers/overview>
- Docker Desktop
- Docker Hub account

### Permissions Error On First Running Docker Commands

```bash
sking@win10-simon:/mnt/d/Git$ docker run -d -p 80:80 docker/getting-started
docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/create": dial unix /var/run/docker.sock: connect: permission denied.
```

- Manage Docker as a non-root user
  - <https://docs.docker.com/engine/install/linux-postinstall/>

```bash
# Steps to follow from Docker KB article (see link above)

sudo groupadd docker

# Didn't expect this but hey ho.
groupadd: group 'docker' already exists

sudo usermod -aG docker $USER

newgrp docker

# Verify that you can run 'docker' commands without 'sudo'.
docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete
Digest: sha256:53f1bbee2f52c39e41682ee1d388285290c5c8a76cc92b42687eecf38e0af3f0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub. 
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### Proceed Through the Tutorial

#### Error Opening Container in Browser

Browsing to <http://localhost:3000> returns the following.

```plaintext
The connection was reset

The connection to the server was reset while the page was loading.

    The site could be temporarily unavailable or too busy. Try again in a few moments.
    If you are unable to load any pages, check your computer’s network connection.
    If your computer or network is protected by a firewall or proxy, make sure that Firefox is permitted to access the Web.
```

```bash
sking@win10-simon:/mnt/d/Docker/getting-started-master$ curl -vv http://localhost:3000
*   Trying 127.0.0.1:3000...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 3000 (#0)
> GET / HTTP/1.1
> Host: localhost:3000
> User-Agent: curl/7.68.0
> Accept: */*
>
* Empty reply from server
* Connection #0 to host localhost left intact
curl: (52) Empty reply from server
```

Tried troubleshooting with this GitHub issue, but no joy: <https://github.com/docker/for-win/issues/3214>

#### Sorted

My bad: I didn't read the instructions properly, and I was building the container from the wrong `Dockerfile` file.

I was building my container from the root of the cloned (downloaded) repository when I should have been running the command in the `/app` directory where the newly created `Dockerfile` was placed.  Doh!

Other than that self-inflicted gotcha, this was a nice, straightforward tutorial.

On to the next one!

----

## Part Two - Persist Data in your App

Source:

<https://docs.microsoft.com/en-us/visualstudio/docker/tutorials/tutorial-persist-data-layer-docker-app-with-vscode>

### Persist Data Using Named Volumes

```bash
# Create a named volume called 'todo-db'.
docker volume create todo-db

# Stop the container that the named volume will be attached to.
docker ps
docker stop <container-id>

# Start the relevant container, specifiying the volume to mount and the location in the file system.
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started-v3

# Look at the volume's properties e.g. location.
docker volume inspect todo-db
```

### Persist Data Using Bind Mounts

_With bind mounts, you control the exact mountpoint on the host. This approach persists data, but is often used to provide more data into containers. You can use a bind mount to mount source code into the container to let it see code changes, respond, and let you see the changes right away._

```bash
# Run your container to support a development workflow.
# NOTE: Run this command from within the /app folder!
docker run -dp 3000:3000 -w /app -v ${PWD}:/app node:12-alpine sh -c "yarn install && yarn run dev"

# Look at the app in the browser.
http://localhost:3000/

# Edit one of the app's files e.g. change a text value in the src/static/js/app.js file.
# Save your change(s)

# Refresh the browser and confirm the text has been changed.
```

### Cache Dependencies

This seems like an important concept, but I didn't get the same results as (presented in) the tutorial.

I didn't see any performance difference between the first build and the subsequent build which should have been (much) quicker due to using the cached dependencies.

I may have taken a misstep somewhere along the way when following the instructions, but I did go back over them again to try to rule out "user error".

Note to self: look out for more examples of this concept in other study materials.

### Multi-stage builds

Again, this looks like it could be an important concept but the examples in the tutorial are a bit sketchy (imo) so I'll hopefully find more helpful examples in other study materials.

----

## Part Three - Deploy your Docker App to Azure

Source:

<https://docs.microsoft.com/en-us/visualstudio/docker/tutorials/tutorial-deploy-docker-app-azure>

## Prerequisites for Part Three

- Azure account with active subscription
- The Azure Account extension for VS Code

### Error Creating ACI Context Via Command Palette

I encountered an authentication error (which I forgot to copy - doh!) when I was following the step to create an ACI context via the Command Palette in VS Code.

_Select View > Command Palette. Enter Docker Contexts: Create Azure Container Instance Context._

As a workaround, I was able to create my context via the terminal as per <https://docs.docker.com/cloud/aci-integration/#run-docker-containers-on-aci>

```bash
sking@win10-simon:/mnt/d/Docker/getting-started-master$ docker login azure
login succeeded

sking@win10-simon:/mnt/d/Docker/getting-started-master$ docker context create aci sktestacicontext
Using only available subscription : My Test Subscription (<subscription-id>)
? Select a resource group rg-simon-test-docker (uksouth)

Successfully created aci context "sktestacicontext"
```

## Run Containers in the Cloud

```bash
docker context use sktestacicontext

docker run  -dp 3000:3000 sking/getting-started-v3
```

### Error Attempting to Run Container in Cloud

```bash
docker context use sktestacicontext
sktestacicontext

docker run  -dp 3000:3000 getting-started-v3
[+] Running 0/1
 ⠿ Group youthful-bhabha  Error                                                                                                                        1.4s
containerinstance.ContainerGroupsClient#CreateOrUpdate: Failure sending request: StatusCode=400 -- Original Error: Code="InaccessibleImage" Message="The image 'getting-started-v3' in container group 'youthful-bhabha' is not accessible. Please check the image and registry credential."
```

OK...  I think there's an important detail missing from the tutorial.

Basically, the ACI context does not interact with the local images as it expects to find them in the Docker Hub account.

This article helped me to figure this out: <https://www.docker.com/blog/running-a-container-in-aci-with-docker-desktop-edge/>

So in this command from the Microsoft tutorial, `docker run  -dp 3000:3000 <username>/getting-started`, the `<username>` value should be for the Docker Hub and **not** for the Azure account.

When I run the following command, I'm able to successfully deploy an Azure container instance called "sktest" to my test Azure subscription.

```bash
# I sourced this command from https://www.docker.com/blog/how-to-deploy-containers-to-azure-aci-using-docker-cli-and-compose/.
docker --context sktestacicontext run -d --name sktest -p 3000:3000 <my-docker-hub-id>/getting-started-v3

docker context list 
NAME                 TYPE                DESCRIPTION                               DOCKER ENDPOINT                             KUBERNETES ENDPOINT   ORCHESTRATOR
default              moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                                       swarm                       
desktop-linux        moby                                                          npipe:////./pipe/dockerDesktopLinuxEngine                                                    v
sktestacicontext *   aci                 rg-simon-test-docker@uksouth

docker ps
CONTAINER ID        IMAGE                          COMMAND             STATUS              PORTS
sktest              sking2974/getting-started-v3                       Running             20.26.45.220:3000->3000/tcp      

# Tidy up by removing Azure container instance.
docker stop sktest

docker ps -a
CONTAINER ID        IMAGE                          COMMAND             STATUS              PORTS
sktest              sking2974/getting-started-v3                       Terminated          0.0.0.0:3000->3000/tcp

docker rm sktest

docker ps -a
CONTAINER ID        IMAGE               COMMAND             STATUS              PORTS
```

So...

I'm happy that I was able to deploy a container to Azure by working in my ACI context under the Docker CLI, but I'm not sure why I wasn't able to interact with the ACI context via Docker Desktop under VS Code.

Hopefully, this will be clarified - and hopefully resolved - as I move on to other study resources.  Yeeha!

----

## Part Four - Advanced - Create Multi-container Apps with MySQL and Docker Compose

Source:

<https://docs.microsoft.com/en-us/visualstudio/docker/tutorials/tutorial-multi-container-app-mysql>

----
