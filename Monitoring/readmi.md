# Ollo node monitoring tool
## Community dashboard by L0vd.com: [Dashboard link](https://l0vd.notion.site/Projects-5510908598c245bda48313372271cd84)

To monitor you node your should install and configure:
* [InfluxDB](https://www.influxdata.com/products/influxdb/)
* [Grafana](https://grafana.com/)

Advantages  of using our free service:
* Our monitoring service is working on dedicated server (24/7 online)
* No need to install database  (InfluxDB)
* No need to install and configure  Grafana Dashboard
* On Grafana dashboard you will find all necessary metrics of your node (we use this monitoring service by ourselves, so we've configured dashboard properly)

# One line installation:
```
. <(wget -qO- https://raw.githubusercontent.com/L0vd/Ollo/main/Monitoring/ollo-monitoring-install.sh)
```

# OR Manual installation of telegraf and monitoring script

Install telegraf
```
sudo apt update
sudo apt -y install curl jq bc

# install telegraf
sudo cat <<EOF | sudo tee /etc/apt/sources.list.d/influxdata.list
deb https://repos.influxdata.com/ubuntu bionic stable
EOF
sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -

sudo apt update
sudo apt -y install telegraf

sudo systemctl enable --now telegraf
sudo systemctl is-enabled telegraf

# make the telegraf user sudo and adm to be able to execute scripts as ollo user
sudo adduser telegraf sudo
sudo adduser telegraf adm
sudo -- bash -c 'echo "telegraf ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers'
```
You can check telegraf service status:
```
sudo systemctl status telegraf
```
Status can be not ok with default Telegraf's config. Next steps will fix it.

Clone this project repo and copy variable script template
```
git clone https://github.com/L0vd/rebus-monitoring.git
cd ollo-monitoring
nano variables.sh
```

Insert your parameters to **variables.sh**:
* full path to ollo binary to COS_BIN_NAME ( check ```which ollod```)
* node PRC port to COS_PORT_RPC ( check in file ```path_to_ollo_node_config/config/config.toml```)
* node validator address to COS_VALOPER ( like ```ollovaloper********```)

Save changes in variables.sh and enable execution permissions:

```
chmod +x monitor.sh variables.sh
```

Edit telegraf configuration
```
sudo mv /etc/telegraf/telegraf.conf /etc/telegraf/telegraf.conf.orig
sudo mv telegraf.conf /etc/telegraf/telegraf.conf
sudo nano /etc/telegraf/telegraf.conf
```
Set you name to identify yourself in grafana dashboard and check correctness of the path to your monitor.sh file and username your validator runs at. Correct these two settings in the configuration file:
```
# Global Agent Configuration
[agent]
  hostname = "YOUR_MONIKER/SERVER_NAME" # set this to a name you want to identify your node in the grafana dashboard
...
...
[[inputs.exec]]
  commands = ["sudo su -c /root/ollo-monitoring/monitor.sh -s /bin/bash root"] # change path to your monitor.sh file and username to the one that validator runs at (e.g. root)
  interval = "15s"
  timeout = "5s"
  data_format = "influx"
  data_type = "integer"
```

## Dashboard interface 

Dashboard has main cosmos-based node information and common system metrics. There is a description for each metric.

Go to our comunity dashboard and select you node from the server list: 
## [Dashboard link](http://95.216.2.219:3000/d/rebus/rebus-monitoring-by-l0vd?orgId=1&refresh=30s)


![Screenshot_1](https://user-images.githubusercontent.com/43213686/169405751-8ff53124-e128-4078-8d68-229a18ea4e25.png)
![Screenshot_2](https://user-images.githubusercontent.com/43213686/169405777-eb9965a5-9fe8-4ecf-944b-4482c41c019b.png)



### Mon health
Complex parameter can show problem concerning receiving metrics from node. Normal value is "OK"

### Sync status
Node catching_up parameter

### Block height
Latest blockheight of node 

### Time since latest block
Time interval in seconds between taking the metric and node latest block time. Value greater 15s may indicate some kind of synchronization problem.

### Peers
Number of connected peers 

### Jailed status
Validator jailed status. 

### Missed blocks
Number of missed blocks in 10000 blocks running window. If the validator misses more than 500 blocks, it will end up in jail.

### Bonded status
Validator stake bonded info

### Voting power
Validator voting power. If the value of this parameter is zero, your node isn't in the active pool of validators 

### Delegated tokens
Number of delegated tokens

### Version
Version of ollod binary

### Vali Rank
Your node stake rank 

### Active validator numbers
Total number of active validators

### Other common system metrics: CPU/RAM/FS load, etc.
No comments needed)
