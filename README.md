# Docker Datacenter on AWS 

Docker Data Center Design:

Docker Data Center is made of two main components: Docker Universal Control Plane (UCP) and Docker Trusted Registry (DTR). UCP is made of the UCP controllers and UCP nodes. DTR is made of DTR replicas only. UCP needs to be installed first before DTR can be installed. The following is the recommended architecure and installation workflow:

![](images/design_1.png)

Requirements:

**AMI:**

*  Ubuntu 14.04
*  3.16+ Kernel
*  3.5 G Minimum RAM

**Setup:**

* Single VPC in Single Region
* 3 AZs with Private Subnets
* Public Subnet


**Packages:**

* Docker CS Engine 1.11+
* 

**Standard Intallation Shell Script for all Nodes:**


```
wget -qO- https://get.docker.com/ | sh
sudo usermod -aG docker ubuntu
```

**Standard Docker daemon.json:**

```
```

**User Parameters:**

1. UCPSAN: The user defined SAN/FQDN for UCP.
2. License: the json formatted license text.
3. DTRSAN: The user defined SAN/FQDN for DTR.
4. ClusterSize: The total number of UCP node ( excluding any UCP/DTR controller nodes)
5. KeyName: Name of an existing EC2 KeyPair
6. InstanceType: EC2 Instance Type



**Installation Phases:**

1. UCP Installation: First phase is the create the UCP cluster that's composed of the three controllers (we'll call them `controller`,`replica1`, `replica2`) and N number of nodes defined by an auto-scaling group. 

	a. Installing the first UCP Controller:

- Pass the UCPSAN user parameter as environment variable.

```
"export UCPSAN=",
    {
      "Ref": "UCPSAN"
    },
",
	
```

- Pass the license json file from user parameters to local node
	
```
"echo '",
    {
      "Ref": "License"
    },
"' >> /home/ubuntu/docker_subscription.lic \n",
	
```

- Run the following Docker run command:

```
		sudo docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v /home/ubuntu/docker_subscription.lic -e UCP_ADMIN_PASSWORD=ddconaws --name ucp docker/ucp:latest install --jsonlog --fresh-install --host-address $PRIVATE_IP --san $PRIVATE_IP --san $UCPSAN 

```
This will output a SHA fingerprint(UCP_FINGERPRINT) that needs to be passed to other resources *** 
    
   b. **TODO** Install the UCP Replica #1 and #2
   
   - **TODO** Pass the SHA fingerprint and export it as an env var (UCP_FINGERPRINT)
   - **TODO** Get private IP of `controller` and pass it as env var ( export UCP_URL=https://PRIVATE_IP:443)
   - **TODO** Pass the UCPSAN user parameter as environment variable(UCPSAN).

```
	sudo docker run --rm -it --name ucp -v -e UCP_ADMIN=admin -e UCP_ADMIN_PASSWORD=ddconaws /var/run/docker.sock:/var/run/docker.sock docker/ucp join --replica -D --san $UCPSAN

```


c. **TODO** Install the UCP Nodes using an autoscaling group.
	
   - **TODO** Pass the SHA fingerprint and export it as an env var (UCP_FINGERPRINT)
   - **TODO** Get private IP of `controller` and pass it as env var ( export UCP_URL=https://PRIVATE_IP:443)

```
   sudo docker run --rm -it --name ucp -v -e UCP_ADMIN=admin -e UCP_ADMIN_PASSWORD=ddconaws /var/run/docker.sock:/var/run/docker.sock docker/ucp join -D
   
```
	

2.**TODO** DTR Installation


