# Podman
Stop Using Docker. Use Free and Open Source Software (FOSS) instead.

## Resources
- https://www.youtube.com/watch?v=Z5uBcczJxUY&t=72s
- https://podman.io/docs/installation

---

# Why stop using Docker?

It's 2024 and there are different container runtimes now.  
Actually, the most popular K8s (Kubernetes) services are not running Docker, the backend runtime is something else.  

These alternatives follow the OCI (**Open Container Initiative**), which was created to **standardize container specifications** so you're 
not locked into one solution.  

A few years ago, Docker decided to monetize their CLI usage.  
For instance, when you use `docker pull` or `docker push` without specifying a registry, the Docker CLI automatically uses Docker Hub.  
And suddenly, companies using Docker Hub started hitting limit rates.  
Sure you can solve this by switching registries and use AWS CCR, Google GCR, GitHub Quay, and a bunch of others.  
But now, you also have to pay if using Docker Desktop for commercial purposes.  

---

# Podman is better

Podman does everything Docker does, has a simpler CLI, is faster, safer, and is free for both personal and commercial use.  

Podman is also **daemonless**, which provides: 
- enhanced security
- improved resource management
- greater flexibility

In addition to these advantages, Podman offers compatibility with Docker-based workflows, as it suppors OCI and Docker images.  

Being **daemonless** means that Podman operates without a central daemon or background service.  
Instead, it communicates directly with the Linux kernel and container runtime interface.  

Podman uses **runC** as its **default container runtime**.  
runC is an OCI (Open Container Initiative) compliant runtime that is used to launch containers.  
However, Podman is **flexible** and allows users to change the runtime if desired.  

---

# Starting to use a podman machine

Containers run natively on Linux systems, but running Podman on Windows requires WSL2 to be enabled.   
https://github.com/fastoch/WSL2  

## Init & start 

The first thing you want to do, if you're not running Linux, is creating a podman machine: `podman machine init`  
Once it's initiated, you can start it: `podman machine start`  

By default, podman machines are configured in **rootless** mode. 
If your containers require root permissions (e.g. ports < 1024), or if you run into compatibility issues with non-podman clients, 
you can switch using the following command: `podman machine set --rootful`  

## Useful podman commands

- `podman machine ls` provides the list of the podman machines we have created.  
And as a **nushell** hipster, I have to show off and detect columns: `podman machine ls | detect columns`

- To search for images: `podman search <imageName>`. For example: `podman search nginx`
- We can filter the results to only output official images: `podman search nginx --filter=is-official`

## Running our first podman container

- `podman run -dt -p 8080:80 docker.io/library/nginx`
  - open a browser and head over to localhost:8080, you should see the Nginx web server running
  - the `-dt` flag is for 'detached' mode with port mapping.
  - It would be `portman run -d` without port mapping.
 
Run `podman ps` to show a list of running containers. And `podman ps -a` to show all containers.

---

## About Nushell

- to install it on Windows, open a command prompt or PowerShell and run `winget install nushell`
- After installation, you can launch Nushell by typing `nu` in your command prompt or PowerShell

To set Nushell as your default shell in Windows Terminal:
- Open Windows Terminal
- Press Ctrl + , to open the Settings.
- Go to "Add a new profile" and select "New empty profile".
- Fill in the following details:
  - Name: Nushell (or any name you prefer)
  - Command line: Enter the path to the Nushell executable, to get it, run `get-command nu`
  - Go to the "Startup" option in the Settings menu.
  - Select Nushell as the "Default profile".
  - Click "Save" to apply the changes

---

@3/12
