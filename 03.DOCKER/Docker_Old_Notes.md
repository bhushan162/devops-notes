# NOTES

## Docker vs VMs

| Docker  | VMs |
| --- | --- |
| Shares the Host OS Kernel | Requires a full Guest OS per VM |
| Very lightweight (MBs) | Heavy (GBs) |
| **Boot Time**Seconds | **Boot Time minutes** |
|  |  |
|  |  |

## Docker Commands

```bash
#basic commeands
docker pull <Image_name>                                                                 #pull Docker image 
docker images                                                                            #see all images
docker run <Image_name>                                                                  #run any image 
docker ps                                                                                # see all running conatiners 
docker ps -a                                                                             # see running and stoped conatiners
docker run -p 6000:6379 <Image_name>                                                     # here we can give Host Port: Conatiner port
docker log <name>                                                                        # logs of container 

#give name to conatiner 
docker run -d -p 6001:6379 --name redis-older radis 
			# -d deteched head -- dont show logs 
			#  -- name give name for container
			# radis-older - that we are giving conatimner name and radis --> image name from that conatiner will build 
			
docker start <ID/name> 
docker stop <ID/name>
	
# want to go inside container 
docker exec -it <ID/name> /bin/bash

#---------------------------------------------------delete---------------------------------------------------------------------------

docker rm <ID/name>                                                                    # remove conatiner
docker rmi <name>                                                                      # docker remove image 

#delete all conatiner 
docker rm -f $(docker ps -a -q)
```

## Docker Compose

- whenever we need vreate multiple images then we can use single file make up all conatiner and setup envorement
- 

```bash
docker compose up -d 
docker compose stop
```

# Dokerfile

- Dockerfile we need to create the image of our project

### dockerfile

```docker
FROM node
ENV mongo_DB_USERNAME = admin
			mongo_DB_PWD =password 
			
RUN mkdir -p/home/app
COPY ./home/app

CMD["node","server.js"]

```

### Docker file command command

```bash
#--------------------------build image --------------------------------------------------------
docker build -T my-app:1.0                             #This command run in directory where dockerfile exit along with composefile

```

/#

# Volumes

- used for persistance data

## Type of volume

### TYPE 1

```bash
docker run -v /home/mount/data:vae/lib/mysql/data
							#host             # conatiner
```

### TYPE 2

```bash
docker run -v /var/lob/mysql/data
								#conatiner 
#here we dont need to specify the host path it wil take care by container 
```

### TYPE 3 (named volume)

```bash
docker run -v name:/var/lob/mysql/data
## herer we dont give exact place from host but we give refrence namne
# this is most used one  
```

# Use cases

- create ubuntu image and enter into it in one command
    
    `docker run -it --name platform_sandbox ubuntu:latest bash`
    
    **What this does under the hood:**
    
    - `docker run`: Tells the Docker engine to create and start a container.
    - `it`: Interactive mode + TTY (keeps the terminal open so you can type commands inside it).
    - `-name platform_sandbox`: Names your container so we can find it later.
    - `ubuntu:latest`: Pulls the official, minimal enterprise Ubuntu image.
    - `bash`: Launches the Bash shell immediately inside that container.
- You write a simple Python script. You create a Dockerfile to package it. You build the image, run docker images, and see that your image size is 950 MB for a script that is only 5 KB.
    - When you write FROM python:3.11 at the top of your Dockerfile, Docker doesn't just download Python. It pulls down a full, heavy Debian/Ubuntu Linux Operating System layer containing hundreds of system tools (like compilers, package managers, and text editors) that your Python script will never actually use.
    - To drop your image size from **950MB to under 50MB**, you must use an **Alpine Linux** or **Distroless** base image. Alpine is a security-oriented, incredibly lightweight Linux distribution that is only **5MB** in size.
    
    #### The Rookie Way (950 MB):
    
    ```bash
    FROM python:3.11
    COPY . /app
    CMD ["python", "/app/sys_check.py"]
    ```
    
    #### The Enterprise Way (45 MB)
    
    ```bash
    # 1. Use the tiny alpine variant
    FROM python:3.11-alpine  
    
    # 2. Set a working directory
    WORKDIR /app             
    
    # 3. Copy only necessary files
    COPY sys_check.py .      
    
    # 4. Execute safely
    CMD ["python", "sys_check.py"]
    ```