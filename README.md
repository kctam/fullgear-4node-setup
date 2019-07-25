# A Fabric Network deployed on 4 Nodes
A Fabric Network of 3 Orderers with Kafka, 2 Organizations each of which has 2 peers, deployed on 4 nodes.
## Node Setup
We need total four nodes, designated Node 1-4. With the following setup.

| Node | Zookeeper | Kafka | Orderer | Peer | CLI |
| --- | --- | --- | --- | --- | --- |
| 1 | zookeeper0 | kafka0 | orderer0.example.com | peer0.org1.example.com|cli |
| 2 | zookeeper1 | kafka1 | orderer1.example.com | peer1.org1.example.com|cli |
| 3 | zookeeper2 | kafka2 | orderer2.example.com | peer0.org2.example.com|cli |
| 4 | | kafka3 | | peer1.org2.example.com |cli|

## Steps

### Step 1: Launch Four Nodes
The setup is tested with Ubuntu 18.04 LTS and on AWS EC2 t2.small instances. It should also work in other cloud instances.
For demo purpose simply open all ports in security group (or equivalent).

Keep the public IP address of the four nodes.

### Step 2: Install everything required in a Hyperledger Fabric Node
That includes the [prerequisite](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html) and the [fabric software](https://hyperledger-fabric.readthedocs.io/en/latest/install.html). Release 1.4.1 is tested in this setup. If a single cloud provider is used,
we can create a machine image after installing all the software. Next time when we launch the four nodes we don't need to redo it again.

Here is a sample how to do this on AWS: [Setup a Hyperledger Fabric host and Create a Machine Image](https://medium.com/@kctheservant/setup-a-hyperledger-fabric-host-and-create-a-machine-image-682859fd58ba)

### Step 3: Prepare material in localhost
Clone this repository in `fabric-samples`

Modify the `.env`, with the public IP address for each node.

```
NODE1=
NODE2=
NODE3=
NODE4=
```

### Step 4: Copy the whole directory to the four nodes
```
cd fabric-samples
tar cf fullgear-4node-setup.tar fullgear-4node-setup/
```
And scp to each node.
```
scp -i <key_name> fullgear-4node-setup.tar ubuntu@[node_address]:/home/ubuntu/fabric-samples/
```

### Step 5: Bring up containers in each node
On each node,
```
cd fabric-samples
tar xf fullgear-4node-setup.tar
cd fullgear-4node-setup
docker-compose -f node<n>.yaml up -d
```
After it is done on four nodes, do a `docker ps` and check whether all containers are up and running.

Note that there are chances that kafka container exits. If so, just perform a docker-compose command again on those nodes, until all containers are up and running.

### Step 6: Create and join channel
In this setup I only have one channel *mychannel*. We will create the **mychannel.block** first using cli in Node 1. 

```
# Node 1
docker exec cli peer channel create -o orderer0.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx
docker exec cli peer channel join -b mychannel.block
```
Then copy **mychannel.block** to other nodes. I am using localhost and scp.
```
# Node 1
docker cp org1-cli:/opt/gopath/src/github.com/hyperledger/fabric/peer/mychannel.block .

# localhost
scp -i ~/Downloads/aws.pem ubuntu@[Node1]:/home/ubuntu/fabric-samples/fullgear-4node-setup/mychannel.block .
scp -i ~/Downloads/aws.pem mychannel.block ubuntu@[Node2&3&4]:/home/ubuntu/fabric-samples/fullgear-4node-setup/

# Node 2, 3 and 4
docker cp mychannel.block cli:/opt/gopath/src/github.com/hyperledger/fabric/peer/
```

Now **mychannel.block** is in all cli in each node. We will join other peers to *mychannel*

```
# Node 2, 3 and 4
docker exec cli peer channel join -b mychannel.block
```

### Step 7: Test your chaincode

Now the *mychannel* is ready, and you can start testing your own chaincode.
