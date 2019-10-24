# fabric-operation

Operation scripts for creating and managing Hyperledger Fabric networks.

## Prerequisites
* Install Docker and Docker Compose as described [here](https://docs.docker.com/compose/install/).
* For Mac, enable kubernetes as described [here](https://docs.docker.com/docker-for-mac/#kubernetes).

## Start CA server and generate crypto data
```
cd ./ca
./start-ca.sh
./bootstrap.sh
./stop-ca.sh
```
These steps generates crypto data for a sample network operator, i.e., _netop1.com_.  The sample network is defined in the config file [netop1.env](./config/netop1.env).  Each of the above scripts accepts an optional argument that specifies which config file to load, e.g., `./start-ca.sh org1` would load the network config from file [org1.env](./config/org1.env).

After the network config is defined, the following steps can be used to generate crypto data from CA servers:
* Configure and start CA server and TLS CA server using the script [start-ca.sh](./ca/start-ca.sh).
* Generate crypto data for the specified network, using the script [bootstrap.sh](./ca/bootstrap.sh).
* Shutdown CA servers and cleanup work files, using the script [stop-ca.sh](./ca/stop-ca.sh).

## Sample crypto data
Sample crypto data generated by the above steps are show in [netop1.com](./netop1.com).

It uses the same folder structure as the result from _cryptogen_ that is demonstrated by [fabric-samples](https://github.com/hyperledger/fabric-samples). However, by using CA server, our scripts are more flexible and adaptable for production environment.

## Optional: start sample Fabric network, and verify the crypto data
You may manually configure a Fabric network to verify the correctness of the generated crypto data according to the steps described in [docker-netop1](./docker-netop1). This is useful only if you have changed the CA scripts or upgraded the CA servers.

The following steps will generate the MSP definisions and Fabric network configurations, and so you will not have to manually configure the docker containers.

## Generate MSP definition and genesis block
To create artifacts for a fabric network managed by using docker-compose:
```
cd ./msp
./bootstrap.sh
```
This command, [bootstrap.sh](./msp/bootstrap.sh), creates the MSP definition and network profiles, [configtx.yaml](./netop1.com/artifacts/configtx.yaml) for the sample network, and uses the network profiles to create orderer genesis block and transaction for creating a test channel, _mychannel_, as specified in the onfig file [netop1.env](./config/netop1.env).  The resulting artifacts are stored in folder [artifacts](./netop1.com/artifacts).

Similar to previous commands, you can provide an optional argument to this script to load different network config, e.g., `./bootstrap.sh org1`.

To create artifacts for a fabric network managed by using Kubernetes, i.e., docker-desktop on Mac, use the following command option:
```
cd ./msp
./bootstrap.sh netop1 k8s
```
## Configure and start Fabric network
If you created msp artifacts for docker-compose in the previous step, use the following commands to start and test the network using docker-compose:
```
cd ./network
./start-docker.sh
./docker-test.sh
./stop-docker.sh
```
These commands does the following:
* Generate docker-compose yaml files for the Fabric network, and then start the fabric network, using the script [start-docker.sh](./network/start-docker.sh).  The yaml files are stored in folder [network](./netop1.com/network).
* Use the script [docker-test.sh](./network/docker-test.sh) to create a test channel, join a peer to the channel, install a test [chaincode](./chaincode/chaincode_example02/go), and invoke chaincode transactions.
* Use the script [stop-docker.sh](./network/stop-docker.sh) to stop the network and cleanup the docker containers and images.

If you created msp artifacts for kubernetes in the previous step, use the following commands to start and test the network using `docker-desktop` on Mac:
```
cd ./network
./start-k8s.sh
./k8s-test.sh
./stop-k8s.sh
```
* The script [start-k8s.sh](./network/start-k8s.sh) generates yaml files for Kubernetes persistent volumes and orderer and peer pods, starts the fabric network using the `docker-desktop` on Mac. The yaml files are stored under folder [network](./netop1.com/network). 
* The script [k8s-test.sh](./network/k8s-test.sh) collects artifacts required by the tests and stores the artifacts in folder [k8s](./netop1.com/k8s), and then creates and starts a `cli` pod to run tests for channel creation and chaincode invocations.
* The script [stop-k8s.sh](./network/stop-k8s.sh) shutdown all kubernetes pods, deletes persistent volumes, and cleans up the persistent data.

## TODO
Will soon enhance these scripts to support Kubernetes on popular cloud service providers.