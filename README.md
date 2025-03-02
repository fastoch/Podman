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

To restart a pod: `podman pod start <pod_name>`

## Our first podman pod

Let's create a pod and name it 'webapp': `podman pod create --name webapp` (requires your podman machine to be started)  
![image](https://github.com/user-attachments/assets/583d766b-f4f2-4db7-b0ef-054a4b2e2f97)  

You can notice that the pod is already running something, despite you haven't told it to run anything...  
That's because every pod, by default, runs an internal **infra container**.  
This infra container is a lightweight container used to **coordinate the shared kernel namespace** of a pod.  

### The infra container

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

The nice thing about containers in a pod is that they don't have to have anything special to communicate with each other.  
They can basically ping each other using `localhost`.  

Let's run a third container in our pod:
`podman run --rm --pod webapp redis redis-cli -h localhost SET mykey "Hello from Redis"`
- The `--rm` flag is to make sure the container is deleted as soon as it's exited
- this container will run Redis (Remote dictionary server, functions as a distributed key-value database)
- we use `redis-cli` with `localhost` as the target host to send a redis command
- our redis command writes a key name 'mykey' which will hold the value "Hello from Redis"

Now we can run another command to read the key:  
`podman -run --rm --pod webapp redis redis-cli -h localhost GET mykey`  

![image](https://github.com/user-attachments/assets/8e60c7f7-6244-4396-b20e-ae2e4a8a8441)  

## Attaching a shell to a container running inside a pod

`podman run -it -d --pod webapp --name api alpine sh`  
- This will run a **third** container named 'api' inside our 'webapp' pod.
- This container will be built from the alpine image (lightweight Linux OS)
- The `-it` flag and `sh` allow us to attach a shell to the 'api' container
- The `-d` flag is for running the container in 'detached' mode so we don't leave the terminal

We can then `exec` into this container via `podman exec -it api sh`, which allows us to use the shell attached to it.  
![image](https://github.com/user-attachments/assets/2c286b78-c254-4c36-9a69-b9cfee2587ef)  

**Note**: we use the old shell because Alpine doesn't come with bash  

We can then install redis in our Alpine container, and send a Redis PING to localhost that comes back with PONG:  
![image](https://github.com/user-attachments/assets/d8530874-64de-4c84-85ff-a195f981b945)

---

# If you're using or plan to use K8s

## Deploy containers locally and generate a K8s-ready .yaml file

Let's say that we want to create a separate new application with nginx and postgres:
```bash
podman run -d --name web nginx
podman run -d --name db postgres
```

We can now run the `podman generate kube` command and save the output to a .yaml file:  
`podman generate kube web db | save deployment.yaml`  

Here are the contents of the .yaml file:
```yaml
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-5.4.0

# NOTE: If you generated this yaml from an unprivileged and rootless podman container on an SELinux
# enabled system, check the podman generate kube man page for steps to follow to ensure that your pod/container
# has the right permissions to access the volumes added.
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2025-03-02T15:37:09Z"
  labels:
    app: web-pod
  name: web-pod
spec:
  containers:
  - args:
    - nginx
    - -g
    - daemon off;
    image: docker.io/library/nginx:latest
    name: web
  - args:
    - postgres
    image: docker.io/library/postgres:latest
    name: db
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: 6c18e36cf4f0483b6d643ee066240c0fc85c8281c0454eaca699b56d5c9af67e-pvc
  volumes:
  - name: 6c18e36cf4f0483b6d643ee066240c0fc85c8281c0454eaca699b56d5c9af67e-pvc
    persistentVolumeClaim:
      claimName: 6c18e36cf4f0483b6d643ee066240c0fc85c8281c0454eaca699b56d5c9af67e
```

## Use the .yaml file to deploy the containers locally inside a pod

We can now remove our 2 containers with `podman rm -f web db`.  

And we can use our .yaml file to easily recreate these containers and have them run inside a pod: `podman play kube deployment.yaml`  
![image](https://github.com/user-attachments/assets/2ec78888-1db2-4953-bfec-7aff0d164508)  

We now have 2 pods running 3 containers each:  
![image](https://github.com/user-attachments/assets/a1d304f7-7093-4274-8d2c-4718babe1bbd)  

We can also get the names of containers running inside each pod:  
![image](https://github.com/user-attachments/assets/293b002f-48c1-4d80-8786-b0bf1f5de32e)

## Deploy the pod to a K8s cluster (from local dev environment to production)

### Review and modify the .yaml file

First, we need to modify our `deployment.yaml` file:
- Remove the `creationTimestamp` field from the metadata section. This field is automatically generated by K8s when the pod is created
- Update the `volume` and `persistentVolumeClaim` names to be more meaningful and easier to manage
  - Replace the long hexadecimal string with a descriptive name, such as "postgres-data"
- Add resource requests and limits for both containers to ensure proper scheduling and resource allocation
- Add environment variables for the PostgreSQL container to set up the database properly

Here's the updated .yaml file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web-pod
  name: web-pod
spec:
  containers:
  - name: web
    image: docker.io/library/nginx:latest
    args:
    - nginx
    - -g
    - daemon off;
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 256Mi
  - name: db
    image: docker.io/library/postgres:latest
    env:
    - name: POSTGRES_DB
      value: mydb
    - name: POSTGRES_USER
      valueFrom:
        secretKeyRef:
          name: postgres-secrets
          key: username
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: postgres-secrets
          key: password
    resources:
      requests:
        cpu: 200m
        memory: 256Mi
      limits:
        cpu: 1
        memory: 512Mi
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: postgres-data
  volumes:
  - name: postgres-data
    persistentVolumeClaim:
      claimName: postgres-data-claim
```

Before applying this pod configuration, you need to create a Secret named `postgres-secrets` with the database username and password.  
You can do this using the following command:
```bash
kubectl create secret generic postgres-secrets \
  --from-literal=username=myuser \
  --from-literal=password=mypassword
```

Remember to create both the `Secret` and the `PersistentVolumeClaim` before applying the pod configuration.

### Deploy the pod to a K8s cluster

- Once our .yaml file is ready, let's make sure minikube is on with `minikube status`
- make sure the cluster is empty: `kubectl get pods`
- deploy the pod: `kubectl apply -f deployment.yaml`

In order to help differentiate our .yaml files, we should give them different names, such as `dev_deployment.yaml` and `prod_deployment.yaml`  

To follow the logs of a specific container in the pod: `kubectl logs <podName> -c <containerName> --follow`


@11/12
