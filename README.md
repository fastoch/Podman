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

---

### About Nushell

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

## Running our first podman container

- `podman run -d -p 8080:80 docker.io/library/nginx`
  - open a browser and head over to localhost:8080, you should see the Nginx web server running
  - the `-d` flag is for running the container 'detached' mode, so we can keep using the terminal
 
Run `podman ps` to show a list of running containers. And `podman ps -a` to show all containers.  
Stop a container via `podman stop <containerName>`.  

- `podman top <containerName>` shows the current processes running inside a container and how much resources they're consuming 

---

# An important differentiator 

While Docker needs its Docker socket to run, which is running as root, Podman is **daemonless** and **rootless**.  
If you run `ls -l /var/run/docker.sock | select user`, you can verify that the user for the docker socket is root.  

If you run podman containers and look for processes mentioning 'podman' via `ps | find podman`, you'll see there are none.  
On the opposite, Docker needs a bunch of processes to run containers.  

---

# Podman's best selling point

As its name indicates, Podman lets you gather multiple containers in **pods** and have them run together, which is something Docker can't do.  

To access these pods through the CLI, podman comes with the `podman pod` command.  
The `podman pod ps` command can be used to list information about pods we have created.

## Our first podman pod

Let's create a pod and name it 'webapp': `podman pod create --name webapp` (requires your podman machine to be started)  
![image](https://github.com/user-attachments/assets/583d766b-f4f2-4db7-b0ef-054a4b2e2f97)  

You can notice that the pod is already running something, despite you haven't told it to run anything...  
That's because every pod, by default, runs an internal **infra container**.  
This infra container is a lightweight container used to **coordinate the shared kernel namespace** of a pod.  

## The infra container

The infra container serves several important purposes:
- It holds the namespaces associated with the pod, allowing Podman to connect other containers to the pod.
- It enables starting and stopping containers within the pod while keeping the pod running.
- It manages pod-level attributes such as port bindings, cgroup-parent values, and kernel namespaces

It's important to note that once a pod is created, the attributes assigned to the infra container cannot be changed.  
This means that if you need to modify things like port bindings, you would need to recreate the pod with the new desired config.

## Feeding the pod

Let's pour some stuff into our new pod: `podman run -d --pod webapp --name redis redis`  
This cmd will run a redis container named 'redis' inside our 'webapp' pod.  

As you can see, our pod is now running 2 containers:  
![image](https://github.com/user-attachments/assets/8af189d7-a882-4aa7-8789-e78f2bd41119)  

We can also run `podman pod inspect webapp` to see more detailed information:  
![image](https://github.com/user-attachments/assets/1189daf3-d692-476b-820a-e16ca66fe9ab)

## Communication between containers within a pod

The nice thing about containers in a pod is that they don't have    
`podman run --rm`
- The `--rm` flag is 


@7/12
