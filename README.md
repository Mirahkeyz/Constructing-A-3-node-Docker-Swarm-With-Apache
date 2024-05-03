# Constructing-A-3-node-Docker-Swarm-With-Apache

Scenario: Your company is new to using containers and has been having issues with containerized workloads failing for their global shipping application. The company is worried about additional potential data loss as they have no way to reschedule containers that are failing.

The company has one week to update their infrastructure and leverage the use of Docker Swarm so that data can be restored for the containers that are no longer in a healthy state. Because they are not familiar with Docker Swarm, they will need a step-by-step guide on setting up a Swarm for their containerized workloads to assist with container orchestration.

‌The solution should include the use of a global service since the company’s global shipping application is having issues. Your Swarm should be able to launch the containers using a global service.

# Foundational:

As the junior engineer, you are responsible for only configuring Docker Swarm and providing your team with a step-by-step guide of setting up the Swarm. You should assume that there are new team members that may not have Docker installed on their devices. Your swarm creation should include the following steps via your remote ssh terminal in VSCode:

Installing docker on all hosts

Verifying that docker is in an active state

Change the hostname on each node to identify the master node and the two worker nodes

Validating the worker nodes all share the same security group as the master node

Be sure to add your SSH key to each node

Running the necessary containers using the CLI

Creating the Swarm using one master node and two worker nodes

Show your team members the status of the docker nodes

The first thing we need to do is spin up 3 EC2 instances that will end up being our 3 nodes in our Docker Swarm.

Head over to the EC2 console and click on Launch Instances. I will be naming the first node DockerMasterNode. For the OS we will be choosing Ubuntu.

![Snipe 1](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/9e883504-1402-4fa5-84f7-e0f5757abf5d)

For this project I will be creating a new key pair called DockerSwarm.

In the Network settings, click on Edit and Enable Auto-Assign public IP.

I also will be creating a new security group called DockerSwarmSG. I am going to leave the default security rules for now.

Go ahead and click on Launch instance.

Now we will create our 2 worker nodes. Click on launch instances again.

I am going to name this one DockerWorkerNode1. The OS is going to be the same for all nodes (Ubuntu). Do not forget to Enable Auto-assign public IP for node1 and node2. For Keypair select DockerSwarm and click on Select existing security group and choose DockerSwarmSG.

Go ahead and create node2 using the same steps.

We should now have 3 nodes running. One master and 2 worker nodes.

![Snipe 2](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/9aa57be9-e1e6-4fa3-8d61-a210b3e9401f)

Lets go to our security group and edit the rules inside of it. For the rules, I will be following Brett Fisher’s Docker Swarm Firewall Ports.

![Snipe 3](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/4676fbec-7bbb-4b41-ba09-22472b4d5d34)

Now we need to SSH into the master node. Head over to VS Code then navigate to your .SSH config file and create an entry with the MasterNode hostname (IP address), User (ubuntu) and IdentityFile (path to keypair).

![Snipe 4](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/9ae53954-a10c-4826-b003-1b87cd8ee16e)

Save the file then click on the Remote SSH icon on the lower left hand side of VS Code (You will have to install the Remote-SSH plugin in VS Code). Then click on connect to host.

Once you are logged into the MasterNode run the following command to install Docker:

```
curl -fsSL https://get.docker.com -o get-docker.sh 
sh get-docker.sh
```

![Snipe 5](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/f81907f0-579c-4953-a20f-5d82894c6613)

Now let’s set a password for the ubuntu user:

sudo su root

passwd ubuntu

To make thing easier to manage, we need to change the hostname to MasterNode.

sudo hostnamectl set-hostname MasterNode

sudo reboot

Before we can SSH to either of the worker nodes from our MasterNode, we need to modify the security group to allow traffic from the VPC CIDR.

Time to start adding the SSH keys. In the MasterNode terminal run cd .ssh and create a file with the same name as the keypair (DockerSwarm.pem).

Edit the file and copy and paste the contents of the DockerSwarm.pem file that was created earlier (during instance creation) into it. Then save the file.

Now create a new file called config with the following information:

![Snipe 7](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/05ce8dd4-349a-4920-9db7-77744ca89274)

The Host is going to be the name of the EC2 that we gave to each worker node. Hostname is going to be the private IP of the EC2 instance (node) and IdentityFile is the path to the keypair.

A good way to find the right Identity path is to run the following command once you are inside the directory where the .pem file is located:

$ readlink -f DockerSwarm.pem

If done correctly we should be able to SSH into WorkerNode1 from the MasterNode.

$ ssh WorkerNode1

![Snipe 8](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/f70586e8-10ab-4d0a-82fb-85ab24a9fc0e)

We have to set the Ubuntu user password for each node prior to changing the hostname.

Now we can change the hostname for node 1:

```
hostnamectl set-hostname WorkerNode1
sudo reboot
```

Please install Docker on WorkerNode1 then proceed to setup node2.

![Snipe 9](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/252c0df5-5523-4fce-a55e-9d971faaa07b)

Repeat the same steps we performed for WorkerNode1 to setup WorkerNode2.

After setting up both nodes, go back to the MasterNode’s terminal.

It’s time to setup the Swarm!!!

In the MasterNode terminal type in the following command:

$ sudo docker swarm init

![Snipe 10](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/2e0d3aa8-e2c7-463f-a55f-dd2336788076)

Here we need to copy the docker swarm join command in each of the worker node terminals. Please go ahead and do that.

Now let’s check the status of the Swarm we just created (run this command from the MasterNode terminal):

$ sudo docker node ls

![Snipe 11](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/b5e693dd-b61c-4635-a829-ad09b514b313)

Now we need to fulfill the last task which is to be able to show team members the status of the docker nodes.

We can do this by installing docker visualizer on the MasterNode.

The first step is to clone the GitHub repo:

$ sudo git clone https://github.com/dockersamples/docker-swarm-visualizer

Now we have to cd into docker-swarm-visualizer then run the following command:

$ sudo docker-compose up -d

Docker-Compose is not installed therefore the command above failed. We have to install it first:

```
sudo apt install python3-pip
sudo pip3 install docker-compose
```

![Snipe 12](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/36e6061d-9c98-4e6c-9646-ab1fb7e6d8a0)

The command below runs the visualizer in a Swarm:

```
sudo docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer
```

![Snipe 13](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/b715f6f8-23df-4917-96b7-7c44ef77b9a0)

(FYI - I had to allow all traffic in the security group in order to get the visualizer to open in the browser)

This concludes all of the tasks required by the foundational level.

# Advanced:

As the lead engineer, you’ve reviewed the Swarm configuration and you are now ready to deploy your services on the Swarm. Consider the following steps as you deploy the service:

Using only the CLI, SSH into the master node host and run the command to create your service using an Official image of your choice out of the following company preferred images:

nginx

apache

redis

python/alpine

ubuntu

Initially only launch 1 replica.

Run the commands to verify the service has been created and is running

Since it is a global application that needs to scale, Using the CLI, run the necessary commands to scale the service to 3 replicas.

Verify that the service has scaled

We will start by installing Apache with only 1 replica. We can do this by omitting the replicas syntax within the command:

```
sudo docker service create \
  --name my-apache-website \
  httpd:latest
```

![Snipe 14](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/d1f16770-ce8c-49f6-b421-28573c58cdf6)

There are 2 ways that we can view where the service was created. One way is through the CLI using this command:

$ sudo docker service ps my-apache-website --filter "node=WorkerNode1"

![Snipe 15](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/ad3036af-75b1-46d8-b301-92e175a06f6e)

Another way is to use the visualizer.

![Snipe 16](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/637f68cd-9032-434e-83a6-ddb58d076c49)

The second to last task is asking for us to scale the service by adding 3 more replicas. We can accomplish this with the following:

$ docker service scale my-apache-website=4

![Snipe 17](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/12201beb-1519-4bb2-8939-439e19ee8c89)

The last task is to verify that the service has scaled successfully. We can go check that out in the visualizer.

![Snipe 18](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/c1fb2824-fc74-4bcb-9d38-e059a439ec52)

We just fulfilled all the tasks. I will be running one more command to get Apache running so that we can access it via browser:

$ sudo docker service update --publish-add published=8081,target=80 my-apache-website

![Snipe 19](https://github.com/Mirahkeyz/Constructing-A-3-node-Docker-Swarm-With-Apache/assets/134533695/38c35b88-a284-44dd-88ae-b220073e6a3f)

































































We now have a list of the required tasks. Let’s get started!
