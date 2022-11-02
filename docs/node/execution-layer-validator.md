---
---
# Run a Gnosis Execution Layer Node

You can also run a Gnosis Execution Layer Node on the same machine as your Beacon Chain node(s). This is recommended for more experienced node runners, but is highly encouraged as it increases decentralization of the network. If you don't wish to do this, you can just ignore this section and connect your node through an existing JSON RPC endpoint. Using an existing RPC is the easiest method when you first setup your node. It is also encouraged to use an existing RPC during setup as it can take a while (up to 3 days, depending on your setup) for your execution layer node to fully sync. You will be encouraged to update your setup once the merge happens. If you have experience and want to setup your own GC node now, read on!

## Technical Requirements

GC Nodes can be run with Nethermind client and the following recommended minimum specs:

* OS: Ubuntu, Windows & MacOs
* CPU: 2 cores
* RAM: 8GB
* Disk: 500gb SSD
* Git installed `git --version`

## Setup on Nethermind

:::caution
A majority of Gnosis validators now run the Nethermind client.

Ethstats: [https://ethstats.gnosischain.com/](https://ethstats.gnosischain.com/)
:::

Nethermind Launcher is a self-contained app - you do not need to install .NET separately to run it.

📄 **Nethermind Docs**: [https://docs.nethermind.io/nethermind/first-steps-with-nethermind/getting-started](https://docs.nethermind.io/nethermind/first-steps-with-nethermind/getting-started)

📦 **Latest Packages:** [http://downloads.nethermind.io/](http://downloads.nethermind.io)

🛑 **To stop Nethermind:** `Control + c`

### Configuration

* [Supported Platforms](https://docs.nethermind.io/nethermind/first-steps-with-nethermind/supported-platforms) (macOS, Windows, Linux)
* [Platform-specific instructions for running the client](https://docs.nethermind.io/nethermind/ethereum-client/running-nethermind/running-the-client)
* The Nethermind Launcher will present several options on start. Use arrow keys to select options[ Available CLI methods ](https://docs.nethermind.io/nethermind/nethermind-utilities/cli)- CLI supports javascript.
* [Understanding logs](https://docs.nethermind.io/nethermind/first-steps-with-nethermind/getting-started#explaining-nethermind-logs)

:::note
When you start the Launcher choose the following options to sync a node:

* Start Nethermind: **Node**
* Select network: **xDai**
* Select sync: **Fast sync**
:::

### Setup Instructions (Linux)

#### 1) Set up directories and docker-compose.yml file

*Replace `<user>` with your username on your device*
```bash
    cd
    mkdir /home/<user>/nethermind
    mkdir /home/<user>/nethermind/nethermind_db
    mkdir /home/<user>/nethermind/keystore
    mkdir /home/<user>/nethermind/logs
```
Create docker-compose.yml:
```bash
nano /home/<user>/nethermind/docker-compose.yml
```
Add the following text:
```bash
version: "3.7"
services:

  nethermind:
    hostname: nethermind
    container_name: nethermind
    image: nethermind/nethermind:latest
    restart: always
    stop_grace_period: 1m
    networks:
      net:
        ipv4_address: 192.168.32.100
    ports:
      - "30303:30303/tcp"
      - "30303:30303/udp"
    volumes:
      - /home/<user>/nethermind/nethermind_db:/nethermind/nethermind_db
      - /home/<user>/nethermind/keystore:/nethermind/keystore
      - /home/<user>/nethermind/logs:/nethermind/logs
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - NETHERMIND_CONFIG=xdai
    logging:
      driver: "local"

networks:
  net:
    ipam:
      driver: default
      config:
        - subnet: "192.168.32.0/24"
```
:::note
Be mindful of spacing, yaml is very particular and the error messages for malformed yaml aren't always intuitive. Also, for those unfamiliar with nano, `Ctl-o` to save and `Ctl-x` to close :)
:::

#### 2) Start Nethermind

```bash
cd nethermind
docker-compose up
```
To check logs:
```bash
sudo docker ps -a
sudo docker-compose logs -f <container name>
```
Container name should in this case be 'nethermind'

:::note
This can take 1-3 days to sync, so in the meantime you can connect your Beacon chain node to a public RPC
:::

### Setup as Archive node

A Gnosis archive node execute heavy historical sync verifying all the transactions and keeping all the historical state. In Nethermind, the default configuration activates the pruning functionality.

:::caution
Make sure there's enough disk space to accommodate the archive data, the minimum amount of disk required to run the archive node is as of today ~1.9 TB.
:::

To start a node as an archive node you will need to disable pruning in the config file when running an archive node.

Set the following variable:

```bash
NETHERMIND_PRUNINGCONFIG_MODE: "None"
```

Alternatively, if you followed the [setup instructions](#setup-instructions-linux) with Docker, edit the docker compose file with the following:

```js title="docker-compose.yml"
version: "3.7"
services:

  nethermind:
    hostname: nethermind
    container_name: nethermind
    image: nethermind/nethermind:latest
    restart: always
    stop_grace_period: 1m
    networks:
      net:
        ipv4_address: 192.168.32.100
    ports:
      - "30303:30303/tcp"
      - "30303:30303/udp"
    volumes:
      - /home/<user>/nethermind/nethermind_db:/nethermind/nethermind_db
      - /home/<user>/nethermind/keystore:/nethermind/keystore
      - /home/<user>/nethermind/logs:/nethermind/logs
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      // highlight-next-line
      - NETHERMIND_CONFIG=xdai_archive
    logging:
      driver: "local"

networks:
  net:
    ipam:
      driver: default
      config:
        - subnet: "192.168.32.0/24"
```

Read more about the [Nethermind Sync Modes](https://docs.nethermind.io/nethermind/ethereum-client/sync-modes) in the official documentation.


## Node Maintenance

Once Nethermind has fully synced, you can point your validator nodes to it to use as an RPC to increase decentralization of the network. To do this or any other maintenence, you will have to take down your nodes, so make sure you closely follow the restart procedure closely to avoid slashing.

### Restart Procedure

 **Stop all running containers and remove them**
 ```bash
 docker stop <container name> && sudo docker remove <container name>
 sudo docker ps -a
 ```
 No containers should be running. If you have separate containers for the GC and GBC clients, make sure you stop and remove them both. Every container you were running for both your GC and GBC clients should be stopped and removed. This is very important - not shutting them down properly before setting up new clients after maintenance can cause your key pairs to be run on separate instances, leading to slashing.


**Make Changes**
Once containers are stopped and removed, you can make your changes. This can be adding new validators, or changing configuration settings such as your RPC.

**Restart the Containers**
Restart the containers using `docker-compose up` as you did initially when setting up. Make sure you check the logs to ensure your nodes are running without errors using `sudo docker-compose logs -f <container name>`

### Sharing machines for GC and GBC clients

As was mentioned on the doc page for running a beacon chain node, if you decide to run a Gnosis client (Nethermind) and a Gnosis Beacon Chain client (Lighthouse, Teku, Nimbus or Prysm) on the same machine, we recommend [running both in the same docker-compose.yml](https://docs.docker.com/compose/extends/) file to enable RPC access by container name.

If you have questions about config details for this type of setup, please [ask in our discord.](https://discord.gg/VQb3WzsywU)
