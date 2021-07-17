## Elasticsearch Installation

- Create the **elastic** user:

```bash
sudo useradd elastic
```

- Set the maximum number of open files for *elastic* user:

```bash
# /etc/security/limits.conf

elastic - nofile 65536
```

- Set the maximum number of memory map areas a process may have:

```bash
# /etc/sysctl.conf

vm.max_map_count = 262144
```

- Reload the sysctl configuration:

```bash
sudo sysctl --load
```

- Download elasticsearch **7.2.1** binaries:

```bash
curl --remote-name https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-linux-x86_64.tar.gz

tar --extract --gzip --verbose --file elasticsearch-7.2.1-linux-x86_64.tar.gz

mv elasticsearch-7.2.1 elasticsearch
```

## Cluster 1

### Master Node

- Elasticsearch configuration:

```yml
# elasticsearch/config/elasticsearch.yml

# --- Cluster ---
cluster.name: c1

# --- Node ---
node.name: master-1
node.attr.zone: 1

# --- Network ---
network.host: [ _local_, _site_ ]

# --- Discovery ---
cluster.initial_master_nodes: [ "master-1" ]

# --- Roles ---
node.master: true
node.data: false
node.ingest: false
```

- JVM config:

```bash
# elasticsearch/config/jvm.options

# Xms Initial size of total heap space
# Xmx Maximum size of total heap space

-Xms768m
-Xmx768m
```

- Start Elasticsearch binary:

```bash
./bin/elasticsearch --daemonize --pidfile pid
```

- Check Elasticsearch:

```bash
less logs/c1.log

# or

curl http://localhost:9200/_cat/nodes?v
```

### Data Node

- Elasticsearch configuration:

```yml
# elasticsearch/config/elasticsearch.yml

# --- Cluster ---
cluster.name: c1

# --- Node ---

# data-1
node.name: data-1
node.attr.zone: 1
node.attr.temp: hot

# data-2
node.name: data-2
node.attr.zone: 1
node.attr.temp: warm

# --- Network ---
network.host: [ _local_, _site_ ]

# --- Discovery ---
discovery.seed_hosts: [ "<MASTER_NODE_PRIVATE_IP_ADDR>" ]
cluster.initial_master_nodes: [ "master-1" ]

# --- Roles ---
node.master: false
node.data: true
node.ingest: false
```

- JVM config:

```bash
# elasticsearch/config/jvm.options

# Xms Initial size of total heap space
# Xmx Maximum size of total heap space

-Xms1g # default
-Xmx1g # default
```

- Start Elasticsearch binary:

```bash
./bin/elasticsearch --daemonize --pidfile pid
```

- Check Elasticsearch:

```bash
less logs/c1.log

# or

curl http://localhost:9200/_cat/nodes?v
```

### Kibana Installation

- Download kibana **7.2.1** binaries:

```bash
curl --remote-name https://artifacts.elastic.co/downloads/kibana/kibana-7.2.1-linux-x86_64.tar.gz

tar --extract --gzip --verbose --file kibana-7.2.1-linux-x86_64.tar.gz

mv kibana-7.2.1 kibana
```

- Kibana configuration:

```yml
# kibana/config/kibana.yml

server.port: 80
server.host: "<MASTER_NODE_PRIVATE_IP_ADDR>"
```

- Start Elasticsearch binary:

```bash
sudo ./bin/kibana --allow-root
```

## Cluster 2

### Single Node

- Elasticsearch configuration:

```yml
# elasticsearch/config/elasticsearch.yml

# --- Cluster ---
cluster.name: c2

# --- Node ---
node.name: node-1

# --- Network ---
network.host: [ _local_, _site_ ]

# --- Discovery ---
cluster.initial_master_nodes: [ "node-1" ]

# --- Roles ---
node.master: true
node.data: true
node.ingest: true
```

- JVM config:

```bash
# elasticsearch/config/jvm.options

# Xms Initial size of total heap space
# Xmx Maximum size of total heap space

-Xms1g # default
-Xmx1g # default
```

- Start Elasticsearch binary:

```bash
./bin/elasticsearch --daemonize --pidfile pid
```

- Check Elasticsearch:

```bash
less logs/c2.log

# or

curl http://localhost:9200/_cat/nodes?v
```

### Kibana Installation

- Download kibana **7.2.1** binaries:

```bash
curl --remote-name https://artifacts.elastic.co/downloads/kibana/kibana-7.2.1-linux-x86_64.tar.gz

tar --extract --gzip --verbose --file kibana-7.2.1-linux-x86_64.tar.gz

mv kibana-7.2.1 kibana
```

- Kibana configuration:

```yml
# kibana/config/kibana.yml

server.port: 80
server.host: "<MASTER_NODE_PRIVATE_IP_ADDR>"
```

- Start Elasticsearch binary:

```bash
sudo ./bin/kibana --allow-root
```

## Infrastructure Topology

- Multi-node cluster:

```bash
+----------+-----------+-------------------+--------------+----------+
| Server   | Node Name | Attributes        | Roles        | JVM Heap |
+----------+-----------+-------------------+--------------+----------+
| master-1 | master-1  | zone=1            | master       | 768m     |
+----------+-----------+-------------------+--------------+----------+
| master-2 | master-2  | zone=2            | master       | 768m     |
+----------+-----------+-------------------+--------------+----------+
| master-3 | master-3  | zone=3            | master       | 768m     |
+----------+-----------+-------------------+--------------+----------+
| data-1   | data-1    | zone=1, temp=hot  | data, ingest | 2g       |
+----------+-----------+-------------------+--------------+----------+
| data-2   | data-2    | zone=2, temp=hot  | data, ingest | 2g       |
+----------+-----------+-------------------+--------------+----------+
| data-3   | data-3    | zone=3, temp=warm | data         | 2g       |
+----------+-----------+-------------------+--------------+----------+
```

## Creating Certificates

- Create a Certificate Authority file:

```bash
# elasticsearch/config/certs

/home/elastic/elasticsearch/bin/elasticsearch-certutil ca \
  --out config/certs/ca \
  --pass elastic_ca
```

- Create a certificate for master node:

```bash
# ex. Master 1

/home/elastic/elasticsearch/bin/elasticsearch-certutil cert \
  --ca config/certs/ca \
  --ca-pass elastic_ca \
  --name master-1 \
  --dns <MASTER_DNS_ADDRESS> \
  --ip <MASTER_PRIVATE_IP_ADDRESS> \
  --out config/certs/master-1 \
  --pass elastic_master_1
```

- Create certificates for data nodes:

```bash
# ex. Data 1

/home/elastic/elasticsearch/bin/elasticsearch-certutil cert \
  --ca config/certs/ca \
  --ca-pass elastic_ca \
  --name data-1 \
  --dns <DATA_1_DNS_ADDRESS> \
  --ip <DATA_1_PRIVATE_IP_ADDRESS> \
  --out config/certs/data-1 \
  --pass elastic_data_1
```

- Move certificates to respective nodes:

```bash
chown <YOUR_USER>:<YOUR_USER> [master-1, data-1, data-2, node-1]

scp [master-1, data-1, data-2, node-1] <NODE_PRIVATE_IP_ADDRESS>:/tmp

chown elastic:elastic [master-1, data-1, data-2, node-1]
```

## Encrypt the Transport Network

- Change certificate permissions file:

```bash
chmod 640 [master-1, data-1, data-2, node-1]
```

### Node Configurations:

- Elasticsearch config:

```yml
# elasticsearch/config/elasticsearch.yml

# --- X-Pack ---
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: full # or certificate
xpack.security.transport.ssl.keystore.path: certs/[master-1, data-1, data-2, node-1] # certificate name
xpack.security.transport.ssl.truststore.path: certs/[master-1, data-1, data-2, node-1] # certificate name
```

- Add pass-phrase to Elasticsearch keystore:

```bash
# keystore

echo "CERTIFICATE_PASSWORD_HERE" | \
/home/elastic/elasticsearch/bin/elasticsearch-keystore add --stdin \
  xpack.security.transport.ssl.keystore.secure_password

# truststore

echo "CERTIFICATE_PASSWORD_HERE" | \
/home/elastic/elasticsearch/bin/elasticsearch-keystore add --stdin \
  xpack.security.transport.ssl.truststore.secure_password
```

- Restart Elasticsearch:

```bash
pkill --pidfile pid; ./bin/elasticsearch --daemonize --pidfile pid
```

- Kibana config:

```yml
# kibana/config/kibana.yml

elasticsearch.username: "kibana"
elasticsearch.password: "<KIBANA_USER_PASSWORD>"
```

- Restart Kibana.

## Encrypt the Client Network

**IMPORTANT**: In a production environment, is highly recommend to use a global signed certificate, instead of a self-signed certificate.

### Node Configurations:

- Elasticsearch config:

```yml
# elasticsearch/config/elasticsearch.yml

# --- X-Pack ---
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/[master-1, data-1, data-2, node-1]
xpack.security.http.ssl.truststore.path: certs/[master-1, data-1, data-2, node-1]
```

- Add pass-phrase to Elasticsearch keystore:

```bash
# keystore

echo "CERTIFICATE_PASSWORD_HERE" | \
/home/elastic/elasticsearch/bin/elasticsearch-keystore add --stdin \
  xpack.security.http.ssl.keystore.secure_password

# truststore

echo "CERTIFICATE_PASSWORD_HERE" | \
/home/elastic/elasticsearch/bin/elasticsearch-keystore add --stdin \
  xpack.security.http.ssl.truststore.secure_password
```

- Restart Elasticsearch:

```bash
pkill --pidfile pid; ./bin/elasticsearch --daemonize --pidfile pid
```

- Kibana config:

```yml
# kibana/config/kibana.yml

elasticsearch.hosts: [ "https://localhost:9200" ]
elasticsearch.ssl.verificationMode: none
```

- Restart Kibana.

## Define Elasticsearch user passwords

- Redefine default passwords:

```bash
# elastic, apm_system, kibana, logstash_system, beats_system, remote_monitoring_user

/home/elastic/elasticsearch/bin/elasticsearch-setup-passwords interactive
```
