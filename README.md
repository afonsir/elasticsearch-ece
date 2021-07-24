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

## Create Roles

- Create a role with read-only permissions to all indexes:

```json
POST _secutiry/role/sample_read_only
{
  "indices": [
    {
      "names": [ "sample-*" ],
      "privileges": [ "read", "write", "delete" ]
    }
  ]
}

GET _security/role/sample_read_only
```

## Create Users

- Create a user:

```json
POST _security/user/new_user
{
  "roles": [ "sample_read_only", "kibana_user" ],
  "full_name": "User Full Name",
  "email": "user@company.com",
  "password": "mypwd123"
}

GET _security/user/new_user
```

## Create Index

- Create an index:

```json
PUT index-name
{
  "aliases": {
    // ...
  },
  "mappings": {
    // ...
  },
  "settings": {
    // ...
  }
}
```

## CRUD Operations:

- Create **sample-1** index:

```json
PUT sample-1
```

- List the documents:

```json
GET sample-1/_search
```

- Create a document:

```json
POST sample-1/_doc
{
  "firstname": "John",
  "lastname": "Doe"
}
```

- Show document with metadata by its ID:

```json
GET sample-1/_doc/xCh2y3oBg4UTC8GCpQ3J
```

- Show only document data by its ID:

```json
GET sample-1/_source/xCh2y3oBg4UTC8GCpQ3J
```

- Update a document by its ID:

```json
POST sample-1/_update/xCh2y3oBg4UTC8GCpQ3J
{
  "doc": {
    "lastname": "Wilson",
    "middleinitial": "D"
  }
}
```

- Update a document by its ID through script:

```json
POST sample-1/_update/xCh2y3oBg4UTC8GCpQ3J
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.remove('middleinitial')"
  }
}
```

- Delete a document by its ID:

```json
DELETE sample-1/_doc/xCh2y3oBg4UTC8GCpQ3J
```

- Delete the **sample-1** index:

```json
DELETE sample-1
```

## Data Samples

- Accounts data:

```bash
curl --remote-name https://raw.githubusercontent.com/linuxacademy/content-elastic-certification/master/sample_data/accounts.json
```

- Logs data:

```bash
curl --remote-name https://raw.githubusercontent.com/linuxacademy/content-elastic-certification/master/sample_data/logs.json
```

- Shakespeare data:

```bash
curl --remote-name https://raw.githubusercontent.com/linuxacademy/content-elastic-certification/master/sample_data/shakespeare.json
```

## Bulk API

- Importing **[ accounts, shakespeare, logs ]** data:

```bash
curl \
  --user elastic \
  --insecure \
  --header 'Content-Type: application/x-ndjson' \
  --request POST \
  'https://localhost:9200/[ accounts, shakespeare, logs ]/_bulk?pretty' \
  --data-binary @[ accounts, shakespeare, logs ].json > [ accounts, shakespeare, logs ]-bulk.json
```

- Refresh manually the indices:

```bash
POST /bank/_refresh

POST /shakespeare/_refresh

POST /logs/_refresh
```

## Index Aliases

- To create or delete an alias (or multiple):

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "bank",
        "alias": "sample_data"
      }
    },
    {
      "add": {
        "index": "logs",
        "alias": "sample_data"
      }
    },
    {
      "add": {
        "index": "shakespeare",
        "alias": "sample_data"
      }
    }
  ]
}
```

- To create a filter alias:

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "shakespeare",
        "alias": "Henry_IV",
        "filter": {
          "term": {
            "play_name": "Henry IV"
          }
        }
      }
    }
  ]
}

GET Henry_IV/_search
```

## Index Template

- To create an index template:

```json
PUT _template/logs
{
  "aliases": {
    "logs_sample": {}
  },
  "mappings": {
    "properties": {
      "field_1": {
        "type": "keyword"
      }
    }
  },
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "index_patterns": [ "logs-*" ]
}
```

## Dynamic Mapping

- To create an index with dynamic mapping (also, it can be used in index template):

```json
PUT sample
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_to_keyword": {
          "match_mapping_type": "string",
          "unmatch": "*_text",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "longs_to_integers": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "strings_to_text": {
          "match_mapping_type": "string",
          "match": "*_text",
          "mapping": {
            "type": "text"
          }
        }
      }
    ]
  }
}
```

## Remote Reindexing

- Elasticsearch config:

```yml
# elasticsearch/config/elasticsearch.yml

reindex.remote.whitelist: "<SOURCE_NODE_1_PRIVATE_ID>:9200, <SOURCE_NODE_2_PRIVATE_ID>:9200, ..."
reindex.ssl.verification_mode: certificate
reindex.ssl.truststore.type: PKCS12
reindex.ssl.keystore.type: PKCS12
reindex.ssl.truststore.path: certs/node-1
reindex.ssl.keystore.path: certs/node-1
```

- Add pass-phrase to Elasticsearch keystore:

```bash
# keystore

echo "CERTIFICATE_PASSWORD_HERE" | \
/home/elastic/elasticsearch/bin/elasticsearch-keystore add --stdin \
  reindex.ssl.keystore.secure_password

# truststore

echo "CERTIFICATE_PASSWORD_HERE" | \
/home/elastic/elasticsearch/bin/elasticsearch-keystore add --stdin \
  reindex.ssl.truststore.secure_password
```

- Restart Elasticsearch:

```bash
pkill --pidfile pid; ./bin/elasticsearch --daemonize --pidfile pid
```

- To remotly reindex:

```json
POST _reindex
{
  // source index (remote)
  "source": {
    "remote": {
      // In this case, a remote cluster
      "host": "https://<MASTER_NODE_PRIVATE_IP>:9200",
      "username": "elastic",
      "password": "..."
    },
    "index": "bank",
    // query a subset of data
    "query": {
      "term": {
        "gender.keyword": {
          "value": "F"
        }
      }
    }
  },
  // destination index (local)
  "dest": {
    "index": "accounts_female"
  },
  // mutate data in-transit
  "script": {
    "lang": "painless",
    "source": "ctx._source.remove('gender')"
  }
}
```
