# iot-swarm-iac

A Docker Consumer Edition Swarm in Azure

__PreRequisites__

Requires the use of [direnv](https://direnv.net/).
Requires the use of [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
Requires the use of [Docker](https://www.docker.com/get-started).

### Environment Settings

Environment Settings are stored in the .envrc file.  When installing the swarm this file will be automatically created that then keeps locally the Service Principal information.


Tips
---

```bash
# SSH to the Manager Node
ResourceGroup="demo-swarm"
IPName="$ResourceGroup-nat-ip"

NatLB="$(az network public-ip show --resource-group $ResourceGroup --name $IPName --query ipAddress -otsv)"

ssh docker@$NatLB -p 50000
```

```bash
# Transfer Keys to the Manager to enable SSH to workers
scp -P 50000 ~/.ssh/id_rsa ~/.ssh/id_rsa.pub docker@$NatLB:/home/docker/.ssh
```

```bash
# Deploying and Updating Docker Stacks

docker stack deploy -c <path_to_your_compose> <stackname>
docker stack up deploy -c <path_to_your_compose> <stackname>
```


Swarm Visualizer
---

```bash
# Startup the Visualizer Service
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  dockersamples/visualizer
```

Portainer
---

```bash
curl -L https://downloads.portainer.io/portainer-agent-stack.yml -o portainer-agent-stack.yml
docker stack deploy --compose-file=portainer-agent-stack.yml portainer

docker service create \
    --name portainer \
    --publish 9000:9000 \
    --replicas=1 \
    --constraint 'node.role == manager' \
    --mount type=bind,src=$(pwd)/data,dst=/data \
    portainer/portainer
```

Persistant Storage
---

Configure the CloudStor Plugin on all the servers
```bash
STORAGE_NAME=""
STORAGE_KEY=""

docker plugin set cloudstor:azure \
  AZURE_STORAGE_ACCOUNT=$STORAGE_NAME
  AZURE_STORAGE_ACCOUNT_KEY=$STORAGE_KEY
docker plugin enable cloudstor:azure
```

```bash
# Startup Voting App
wget https://raw.githubusercontent.com/danielscholl/docker-for-azure/master/apps/docker-compose-votingapp.yml && \
  docker stack deploy \
    -c docker-compose-votingapp.yml \
    votingapp
```

#### Voting App

```bash
# Startup Voting App
URL="https://raw.githubusercontent.com/danielscholl/docker-for-azure/master/apps/docker-compose-votingapp.yml"
wget $URL && \
  docker stack deploy \
    -c docker-compose-votingapp.yml \
    votingapp
```
*  Vote: http://swarmLB:5002/
*  Voting Results: http://swarmLB:5003
