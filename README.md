
# Running a Minecraft Server with Docker on Ubuntu
Managing your Mincraft server with docker is a little bit nicer than just using the old "screen -r Mincraft" trick. docker can do snaphoting and auto restarting, and will allow you to think of the Mincraft server as a service, not just a java file that is being executed. Heres how I built mine. You should not have to edit the docker files. Just make sure you replace minecraft jar file version inside the code blocks when specified.

obviosly the only difference if your not using Ubuntu is the installation of java and the installation of docker

## how to build it


### Install all the things
[Install Docker](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
Im hesitant to put this step in here because docker changes there crap alot. I recomment just following dockers install steps in the link above. But here is what i did.

```bash

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

sudo apt-key fingerprint 0EBFCD88

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
sudo apt-get update && sudo apt-get install docker-ce

```

Install java. Make sure you pick the lastest stable version.
```bash
sudo apt update && sudo apt install openjdk-8-jre

```

### Config of Mincraft Server

Download and run [Minecraft Server](https://minecraft.net/en-us/download/server) once manually. This will generate the eula.txt file. Make sure to update the minecraft_server<VERSION>.jar in the command below to your current version.

```bash
java -Xmx1024M -Xms1024M -jar minecraft_server<VERSION>.jar nogui 
```

Edit the eula.txt file. Change "eula=false" to "eula=true".
```bash
nano eula.txt
```
#### Adding minecraft operators
Before moving on i do remmend you grant players operator permissions before building a docker image out of it. becuase its kind annoying to do afterwards.
```bash
java -Xmx1024M -Xms1024M -jar minecraft_server<VERSION>.jar nogui 
```
You should see out put like this eventually
```
[06:12:34] [Server thread/INFO]: Starting minecraft server version 1.12.2
[06:12:34] [Server thread/INFO]: Loading properties
[06:12:34] [Server thread/INFO]: Default game type: SURVIVAL
[06:12:34] [Server thread/INFO]: Generating keypair
[06:12:35] [Server thread/INFO]: Starting Minecraft server on *:25565
[06:12:35] [Server thread/INFO]: Using epoll channel type
[06:12:35] [Server thread/INFO]: Preparing level "world"
[06:12:35] [Server thread/INFO]: Loaded 488 advancements
[06:12:35] [Server thread/INFO]: Preparing start region for level 0
[06:12:36] [Server thread/INFO]: Preparing spawn area: 0%
[06:12:37] [Server thread/INFO]: Preparing spawn area: 27%
[06:12:38] [Server thread/INFO]: Preparing spawn area: 76%
[06:12:39] [Server thread/INFO]: Done (3.961s)! For help, type "help" or "?"
```
Then issue the mincraft command

```bash
op <Your Players Name>
```




### Configure Docker 
Add a Dockerfile in the same directory as the the minecraft server files. 
```bash
nano Dockerfile
```

Past this in the Dockerfile. Make sure to update the minecraft_server<VERSION>.jar in the command below to your current version.
```bash
FROM java:8
WORKDIR /app/
ADD . /app/
EXPOSE 25565:25565
CMD java -Xmx1024M -Xms1024M -jar minecraft_server<VERSION>.jar nogui
```



Build Docker image
```bash
sudo docker build . --tag minecraft
```

Create a docker volume so the server data is persistant if the docker containers stops for any reason.
```bash

sudo docker volume create minecraftvol
```

Run your docker image. this will set the mincraft container to auto restart should anthing happen to your Ubuntu server, unless you stop the docker conatiner manually. 
```bash
sudo docker run -d -p 25565:25565 --restart unless-stopped --mount source=mincraftvol,target=/app minecraft
```

### Ready to Play
you can now connect to the server using the servers ipaddress or Domain name if you have mapped one.


### Some Issues I havent quite resolved.
I dont yet know how to attach the mincraft containers java terminal that is already running. This is important becasue it is only though this process that you can grant players the "operators" permissions so players can run cheats and stuff. currently I have to stop the container, then run the container without the '-d' flag to gain access to the java terminal which allows you issue mincraft commands in "god mode".

#### basic steps
list containers, take note of the Container ID. should be a random string of numbers and letters.
```bash
sudo docker container list
```

Stop the mincraft Container
```bash
sudo docker container stop <ContainerID>
```

To run the Container and get the java terminal.
```bash
sudo docker run -p 25565:25565 --mount source=mincraftvol,target=/app minecraft
```

*after your finished with the java terminal*


list containers again
```bash
sudo docker container list
```
Stop the mincraft Container
```bash
sudo docker container stop <ContainerID>
```

start the container again in normal awesome mode again.
```bash
sudo docker run -d -p 25565:25565 --restart unless-stopped --mount source=mincraftvol,target=/app minecraft
```


hope this helps someone!
