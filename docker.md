## Docker Cheat sheet

Note: Use this only for development. This configuration might lead to security or other issues in production.

### Running Docker

- simple run

  `docker run -it <image_id> bash`

- run with gpu

  `docker run -u ubuntu --gpus all --ipc=host -v $PWD:/home/ubuntu/<dir> [-v more:dirs] -p 8888:8888 [-p more:ports] --name <cont_name> -it <image_id> bash`

  Here, 

  - "-u ubuntu" lets you sign in as a $user. In this case, 'ubuntu' is the user. Replace with your own $user. Or to login as the current user do: -u $(id -u):$(id -g).
  - "-it" lets you use the container interactively. Else it will just run in the background.
  - -v $PWD:/home/ubuntu mounts your current working directory to /home/ubuntu and you will be able to access the files in your $PWD inside the container. This is what I do. Otherwise you will have to copy files to and from the docker container which will unnecessarily waste storage. Also, you will be able to access the files created inside the container from your host OS even after the container is killed/removed. To mount some other volume, do: -v /home/user/some/path:/home/container/path
  - "--gpus all" will expose the gpus on the host to the container. 
  - "--ipc=host" will avoid shared memory issues during multiprocessing.

  This is the most common configuration I use. You can skip any option that you dont need.

- run without gpu

  `docker run -u ubuntu -v $PWD:/home/ubuntu -it new_image_id bash`

- to use jupyter

  `docker run -u ubuntu --gpus all --ipc=host -v $PWD:/home/ubuntu -p 8888:8888 -it new_image_id bash`

  - Start Jupyter inside container:

    `jupyter notebook --ip=0.0.0.0 --port=8888`

    You will get a local host ip like so: http://127.0.0.1:8888/?token=abcdefgh

  - Find docker container ip address

    `docker inspect -f "{{ .NetworkSettings.IPAddress }}" <container id>`

  - You will have to replace the local host ip address with the docker containers ip address.

    > This is a hack. Let me know if you have better solution.

- if display is needed

  `docker run -u ubuntu --gpus all --ipc=host -v $PWD:/home/ubuntu -it -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix:ro -p 8888:8888 img_id bash`

  Display might be needed for viewing outputs like images, plots etc. But I dont remember I ever using it. Kept it here for reference.

- enter a running container as root, to install something

  `docker exec -u ubuntu -it container_id /bin/bash`

### Containers

- list containers

  `docker container ls -a`

- remove containers

  `docker container rm <container id>`

- restart last closed container

  docker start -a -i \`docker ps -q -l\`

  `docker start -a -i <container id>`

- enter a running container from another terminal
  
  `docker exec -u ubuntu -it container_id bash`
  

### Images

- List Images

  `docker image ls`

- Remove dangling images

  `docker image prune --filter="dangling=true"`

## Docker setup steps:

1. login as root:

   `docker run --gpus all -it new_image_id bash`
2. create user

   `useradd -rm -d /home/ubuntu -s /bin/bash -g root -G sudo -u 1000 ubuntu -p "$(openssl passwd -1 ubuntu)"`
3. install sudo, vim

   `apt update`
   
   `apt upgrade`
   
   `apt install -y sudo vim wget curl locate git gcc libglib2.0-0 libsm6 libxext6 libxrender-dev cmake libboost-all-dev libblas-dev liblapack-dev`
   
   `python -m pip install opencv-python dlib`
   
4. Edit bashrc, vimrc
5. In another terminal do

   `docker ps`
   
   and note down the container id
6. exit docker terminal
7. in host, save the container as new image

   `docker commit <container_id> dockerhub_username/repo_name:tag`
8. start docker new image as new user

   `docker run -u ubuntu --gpus all -it new_image_id bash`
9. Install whatever packages you want like conda, python, pip etc
10. repeat steps 4,5,6 and save as new image.
11. now your docker image is ready. enjoy!
12. Backup docker images

    `docker save dockerhub_username/repo_name:tag | gzip > ringnet_v2.tar.gz`
13. Find docker image descendants

    https://gist.github.com/altaurog/21ea7afe578a523e3dfe8d8a746f1e7d
    
    `python docker_descendants.py <img_id>`
14. Find docker container ip address

    `docker inspect -f "{{ .NetworkSettings.IPAddress }}" <container id>`
15. Find ip address of all running containers

    `docker inspect --format "{{.Name}} - {{.NetworkSettings.Networks.app_default.IPAddress}}" $(docker ps -q)`


### Install Install

1. Download Script

   `curl -fsSL https://get.docker.com -o get-docker.sh`

   `sudo sh get-docker.sh`

2. Create the docker group if it does not exist

   `sudo groupadd docker`

3. Add your user to the docker group.

   `sudo usermod -aG docker $USER`

4. Run the following command or Logout and login again and run (that doesn't work you may need to reboot your machine first)

   `newgrp docker`

5. Check if docker can be run without root

   `docker run hello-world`

6. Reboot if still got error

   `reboot`
   
### Alternative Install docker

1. Install some stuff

   `sudo apt update && sudo apt install -y \
   apt-transport-https \
   ca-certificates \
   curl \
   gnupg-agent \
   software-properties-common`

2. Download key

   `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`

3. Add repo

   `sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"`

4. Install

   `sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io`

5. Give permission

   `sudo chmod 666 /var/run/docker.sock`

6. Add user and group

   `sudo groupadd docker`
   `sudo usermod -aG docker $USER`

7. Run

   `docker run hello-world`


### Troubleshoot

1. If docker doesnt work for some reason, try restarting it.

   `sudo systemctl enable docker`
   `sudo systemctl start docker` # or sudo service docker start

2. Permission Denied

   If docker login gives permission denied try either:
   
   `sudo chmod 666 /var/run/docker.sock` or
   
   `delete ~/.docker/config.json`

3. Restart docker

   `sudo systemctl start docker`

