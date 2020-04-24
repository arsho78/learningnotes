## Docker Commands ##

```bash
docker run -a imageName
docker run -d imageName
docker run -it imageName command args 
docker run -p localhostPort:containerPort imageId|imageName
docker run -v localhostDir:containerDir imageName
docker run -v containerDir imageName  # bookmarking containerDir *ref[1]*
docker create imageName
docker start containerId
docker ps
docker ps -a
docker prune
docker stop containerId
docker kill containerId
docker attach containerId|containerName *ref[2]*
docker exec -it containerId command args
docker build -t dockerhubId/dockerProjectName:version(default value is 'lates') .
docker build -f Dockerfile.name .
docker commit -c 'CMD ["start command"]' containerId

# must be run in the directory as docker-compose.yml
docker-comopse up
docker-compose up --build
docker-compose up -d
docker-compose down
docker-compose ps     

minikube start *ref[3]*
minikube ip
kubectl apply -f object-config.yaml
kubectl apply -f k8sdir # apply all the config files in this folder
kubectl get pods|services|kinds
kubectl describe <objectType> <objectName>
kubectl delete -f object-config.yaml # delete an object in the cluster
kubectl set image <objectType>/<objectName> <containerName=newImageName>
kubectl create secret genric|tls|docker-registry <secret-name> --from-literal key=value
eval $(minikube docker-env) # switch docker to the one in VM
```

ref[1]: Bookmarking a container directory makes docker exclude this directory in volume, which means, instead of referencing this directory back to localhost file system, docker will let the container to create and manage this directory. In other words, this directory is created inside the container and not exposed to outside.

ref[2]: `attach` will automatically attach localhost terminal to the root process inside given container.

ref[3]: minikube 在本地系统中创建了一个 VM，所有的 service 和 pod 都运行于此 VM 中，它拥有自己的 ip 地址，因此不能使用 localhost 来访问。

on dockerhub.com, an image tagged with "alpine" means this image only includes required program and keeps itself as small as possible, for example "node:alpine" only includes node.js with minimum other programs.

## Multi stages builds ##

When building project for production purpose, since there is no more change in the source code in production environment, we can simply replace volumes with `COPY . .` during the build stage.

## Docker compose config file template ##

```
version: '3'
services:
	service_name:
		build:
			context: project_dir_on_localhost
			dockerfile: Dockerfile.name
		ports:
			- "localhostPort:containerPort"
		volumes:
			- bookmarkingDir
			- localhostDir:containerDir
		command: ["command", "arg1", "arg2"...]
		environment:
			- variableName=value
			- variableName  # will retrieve the value from localhost system environment
```

### Restart Policies ###

- `"no"`: **DON'T** restart the service for any reason. **Attention** the `"` is mandatory for this option
- `always`: always restart the service
- `on-failure`: if service failed, restart it; if it exits normally, don't restart it
- `unless-stopped`: restart the service except that it is stopped manually

## Kubernetes persistence volume acccess mode ##

- `READWRITEONCE`: can be read and written by a single node
- `READONLYMANY`: can only be read by many nodes
- `READWRITEMANY`: can be read and written by many nodes
