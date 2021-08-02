#Building DefCon

##Setup

### Docker and Docker Compose

The Defcon build requires the installation of Docker and Docker-Compose.

#### Docker

The following is an extract of the [Official Docker Engine installation](https://docs.docker.com/engine/install/ubuntu/)

_Assuming host machine is linux based._

1. Setup Docker repositories.
```shell
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

2. Add Dockers official GPG key:

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

3. Use the following command to add the 'stable' repository.

```shell
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

4. Install Docker Engine.

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### Docker Compose

The following is an extract of the [Official Docker Compose installation](https://docs.docker.com/compose/install/)

1. Download the current stable release of Docker Compose:

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

2. Apply executable permissions to the binary.

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

### Docker Containers

#### Reddcoin Docker repository

The Reddcoin docker repository resides on Github. Clone the repo to a suitable location on your local machine.

```shell
git clone https://github.com/reddcoin-project/reddcoin-docker
```

1. Prepare the local installation to build the docker environment.  
All commands take place in the root folder of reddcoin-docker.  
The docker_compose.yml file in the root path will be over-written with the DefCon specific configuration.

```shell
cd ./reddcoin-docker
cp ./Defcon/docker_compose.yml ./
```

2. Clone other required projects from github into the root project

##### Docker files for electrumX 

```shell
git clone -b reddcoin https://github.com/reddcoin-project/docker-electrumx
```

While for the most part the build process is automated, the initialization of each service should be done individually.

#### Reddcoin core nodes

There are 3 services configured for the launching of reddcoin core. [ reddcoin-server1, reddcoin-server2, reddcoin-server3 ]

##### Pre-requisites:

1. Download the testnet bootstrap

```shell
cd ./bootstrap
wget https://download.reddcoin.com/bin/bootstrap/testnet-blockchain-Jul-16-2021.zip
```

2. Build the reddcoin services.  
return to the root of the project and start the service.


```shell
cd ../
docker-compose up reddcoin-server1
```

This will build the service, run some configurations, prepare the data directories and then start the service.  
At this point the server should be running fine, and should start synchronising with the testnet blockchain.  

Issue a ctrl+c to stop the service

Repeat step 2. the above for [reddcoin-server2, reddcoin-server3]  
Wait for synchonisation to start before issing cntl+c

```shell
docker-compose up reddcoin-server2
ctrl c

docker-compose up reddcoin-server3
ctrl c
```

At this point each of the server should be able to run fine, but to improve internode communications and discovery we can add each of the other configured nodes to service reddcoin.conf configuration.  

Server1
```shell
echo addnode=reddcoin-server2 >> .reddcoin1/reddcoin.conf
echo addnode=reddcoin-server3 >> .reddcoin1/reddcoin.conf
```

Server2
```shell
echo addnode=reddcoin-server1 >> .reddcoin1/reddcoin.conf
echo addnode=reddcoin-server3 >> .reddcoin1/reddcoin.conf
```

Server3
```shell
echo addnode=reddcoin-server1 >> .reddcoin1/reddcoin.conf
echo addnode=reddcoin-server2 >> .reddcoin1/reddcoin.conf
```

Now start all servers and run them in the background

```shell
docker-compose up -d reddcoin-server1
docker-compose up -d reddcoin-server2
docker-compose up -d reddcoin-server3

```

#### ElectrumX

We are using a modified docker image for testnet

The repository should have already been cloned from an [earlier step](#Docker-files-for-electrumX) 

1. Start the ElectrumX service (firstly in the foreground)   
This will pull the necessary images to the local container and start the electrumx service

```shell
docker-compose up reddcoin-electrumx
```

