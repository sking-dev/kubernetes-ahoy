# Get Started with Docker Remote Containers on WSL 2

Source: <https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-containers>

## Prerequisites

- Windows 10, updated to version 2004, Build 18362 or later
- Install WSL 2
  - Install a Linux distribution e.g. Ubuntu 20.04 running under WSL 2
- Install Visual Studio Code
- Install Windows Terminal
  - Optional
    - I haven't done this as I plan to interact with WSL via the Terminal in VS Code
- Sign up for a Docker ID at Docker Hub
  - <https://hub.docker.com/>
  
## Docker Desktop for Windows

- Download installer from the Docker Hub
  - Latest version at time of writing 4.10.1
- Install as per installation instructions
  - Ensure that "Use WSL instead of Hyper-V (recommended)" is checked
- Log out of Windows to complete installation
- Log back in and Docker Desktop launches automatically (takes a little while...)
- Confirm WSL 2 is being used via Settings > General
  - Confirm your required distribution(s) will be able to access Docker via Settings > Resources > WSL Integration
- Open your WSL distribution via VS Code Terminal
  - Type `docker --version`
    - You should see something like
      - *Docker version 20.10.17, build 100c701*

## Do Some Stuff with Docker

I'm going to shift over to a different Microsoft tutorial at <https://docs.microsoft.com/en-us/visualstudio/docker/tutorials/docker-tutorial>

My notes on this tutorial can be found here.  TODO: Add Markdown link to relevant .MD file.
