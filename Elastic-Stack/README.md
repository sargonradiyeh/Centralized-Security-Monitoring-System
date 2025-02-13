# Elastic Stack

- General Reference: [https://www.youtube.com/watch?v=OYS0hzPDgp4](https://www.youtube.com/watch?v=OYS0hzPDgp4)

# Setting up Elasticsearch

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch
```

- Make sure to save the password for the superuser elastic, let’s go to the elasticsearch.yml to configure things now

```bash
cd /etc/elasticsearch
mousepad elasticsearch.yml &
```

- Start by changing the cluster.name, in our case its elk-kali
- Make network.host equal to the local IP of the ELK server which is : 192.168.153.128 in our case
- Make sure to copy this security section, we will proceed with making the certs required:

```bash
#----------------------- BEGIN SECURITY AUTO CONFIGURATION -----------------------
#
# The following settings, TLS certificates, and keys have been automatically      
# generated to configure Elasticsearch security features on 23-09-2024 10:07:25
#
# --------------------------------------------------------------------------------

# Enable security features
xpack.security.enabled: true

xpack.security.enrollment.enabled: true

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: true
  key: /etc/elasticsearch/elasticsearch.key
  certificate: /etc/elasticsearch/elasticsearch.crt
  certificate_authorities: /etc/elasticsearch/elasticsearch.crt

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  key: /etc/elasticsearch/elasticsearch.key
  certificate: /etc/elasticsearch/elasticsearch.crt
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
cluster.initial_master_nodes: ["kali"]

# Allow HTTP API connections from anywhere
# Connections are encrypted and require user authentication
http.host: 0.0.0.0

# Allow other nodes to join the cluster from anywhere
# Connections are encrypted and mutually authenticated
#transport.host: 0.0.0.0

#----------------------- END SECURITY AUTO CONFIGURATION -------------------------
```

- The device is now using SSL certificates for secure communication, now to setup the key and certificate as well as the certificate authority, run the following commands:

```bash
openssl genpkey -algorithm RSA -out elasticsearch.key -pkeyopt rsa_keygen_bits:2048
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/elasticsearch/elasticsearch.key -out /etc/elasticsearch/elasticsearch.crt -config /etc/elasticsearch/elasticsearch-cert.cnf #when it asks for a challenge password use the one for ELK server and the subject should be the servers IP ie: 192.168.153.128
sudo nano /etc/elasticsearch/elasticsearch-cert.cnf
#ADD THIS TO THE .CNF FILE#
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = LB
CN = 192.168.153.128 #WHATEVER THE ELK SERVER IP IS

[v3_req]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.153.128 #WHATEVER THE ELK SERVER IP IS
#CONTINUE WITH RUNNING THE COMMANDS AFTER THIS#
openssl x509 -req -days 365 -in elasticsearch.csr -signkey elasticsearch.key -out elasticsearch.crt
openssl x509 -in /etc/elasticsearch/elasticsearch.crt -out /etc/ssl/certs/elasticsearch.pem -outform PEM
sudo cp /etc/ssl/certs/elasticsearch.pem /usr/local/share/ca-certificates/
sudo update-ca-certificates
sudo mv elasticsearch.key /etc/elasticsearch/ #if not already there
sudo mv elasticsearch.crt /etc/elasticsearch/ #if not already there
sudo cp elasticsearch.crt /usr/local/share/ca-certificates/ #assuming you created them somewhere else like /etc/elasticsearch
sudo update-ca-certificates
sudo chown elasticsearch:elasticsearch /etc/elasticsearch/elasticsearch.crt /etc/elasticsearch/elasticsearch.key
sudo chmod 600 /etc/elasticsearch/elasticsearch.key
sudo systemctl restart elasticsearch
#TO TEST FUNCTIONALITY
curl -v -u elastic:(your_password) https://192.168.153.128:9200
```

- Enable elasticsearch for it to open on startup

```bash
sudo systemctl enable elasticsearch
```

# Setting up Kibana

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install kibana
```

- Before moving forward, lets create a service token for kibana to allow it to connect securely to elasticsearch

```bash
cd /usr/share/elasticsearch/bin
sudo ./elasticsearch-service-tokens create elastic/kibana kibana_token
```

- Make sure to copy paste and save this token, we’ll be using it in the config file, lets giving elasticsearch the permissions to see and use these tokens

```bash
cd /etc/elasticsearch
sudo chown -R elasticsearch:elasticsearch service_tokens
sudo systemctl restart elasticsearch
```

- Let’s go to the kibana.yml to configure things now:

```bash
cd /etc/kibana
mousepad kibana.yml &
```

- Set server.host: "0.0.0.0” and  elasticsearch.hosts: ["[https://192.168.153.128:9200](https://192.168.153.128:9200/)"]
- Set elasticsearch.serviceAccountToken: "(kibana_token’s value)”
- Make sure to copy this for the SSL setup

```bash
# =================== System: Elasticsearch (Optional) ===================
# These files are used to verify the identity of Kibana to Elasticsearch and are required when
# xpack.security.http.ssl.client_authentication in Elasticsearch is set to required.
#elasticsearch.ssl.certificate: /path/to/your/client.crt
#elasticsearch.ssl.key: /path/to/your/client.key

# Enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
elasticsearch.ssl.certificateAuthorities: [ "/etc/ssl/certs/elasticsearch.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
elasticsearch.ssl.verificationMode: full
```

- Give kibana ownership over its own folder

```bash
cd /etc/kibana
sudo chown -R kibana:kibana ./
```

- Enable kibana for it to open on startup

```bash
sudo systemctl enable kibana
```

# Setting up Filebeat

- Open up kibana at [http://192.168.153.128:5601](http://192.168.153.128:5601/) (for example) and go to Home—>Integrations and search up NetFlow Records, click the button on right that says “Also available in Beats” and click on NetFlow / IPFIX Records and now follow the instructions:

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.15.1-darwin-x86_64.tar.gz
tar xzvf filebeat-8.15.1-darwin-x86_64.tar.gz
cd filebeat-8.15.1-darwin-x86_64/
```

- Modify filebeat.yml:

```bash
cd /etc/filebeat
mousepad filebeat.yml &
```

- Modify as follows:

```bash
# ======================= Elasticsearch template setting =======================

setup.template.settings:
  index.number_of_shards: 1
  #index.codec: best_compression
  #_source.enabled: false

..
..
..
# =================================== Kibana ===================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "192.168.153.128:5601" #Replace with kibana's actual link

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:
..
..
..
# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["https://192.168.153.128:9200"] #Replace with elasticsearch's actual link

  # Performance preset - one of "balanced", "throughput", "scale",
  # "latency", or "custom".
  preset: balanced

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  username: "elastic"
  password: "(password)"
```

- Now enable Netflow on Filebeat and modify its config and setup

```bash
filebeat modules enable netflow
cd /etc/filebeat/modules.d
mousepad netflow.yml &
## PUT THIS IN THE .YML FILE ##
# Module: netflow
# Docs: https://www.elastic.co/guide/en/beats/filebeat/main/filebeat-module-netflow.html

- module: netflow
  log:
    enabled: true
    var:
      netflow_host: 192.168.153.128 # Whatever your ELK server's IP is
      netflow_port: 2055
      # internal_networks specifies which networks are considered internal or private
      # you can specify either a CIDR block or any of the special named ranges listed
      # at: https://www.elastic.co/guide/en/beats/filebeat/current/defining-processors.html#condition-network
      internal_networks:
        - private
## RUN THESE ON THE COMMAND LINE ##
filebeat setup
filebeat -e
```

- Assuming you’ve already completed the MikroTik setup , restart filebeat and click on Check Data on the Kibana page’s setup of Netflow, it shoud look like this:

[MikroTik](MikroTik%2011e8479b945080929612cdec795b9f88.md)

![image.png](image.png)

- In Home—>Discover,  filebeat-* logs should be appearing now

# Setting up Logstash

```bash
sudo apt-get install logstash
```

- Setup a logstash conf for Mikrotik syslogs

```bash
cd /etc/logstash/conf.d
```

- Create the conf file

```bash
mousepad miktrotik-syslog.conf
## ADD THIS TO THE FILE ##
input {
  syslog {
    port => 514
    host => "192.168.153.128"  # Listen on all network interfaces for incoming syslog messages
    type => "mikrotik-syslog"
  }
}

filter {
  if [type] == "mikrotik-syslog" {
    #Applying filters here to process the MikroTik log format.
    grok {
      match => { "message" => "%{SYSLOGBASE} %{GREEDYDATA:syslog_message}" }
    }
    date {
      match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://192.168.153.128:9200"]  # Replace with your ELK server IP
    index => "mikrotik-logs-%{+YYYY.MM.dd}"  
    user => "elastic"  
    password => "(your password)"  
    ssl => true  
    ssl_certificate_authorities => ["/etc/elasticsearch/elasticsearch.crt"] 
  }
  stdout { 
    codec => rubydebug  # Output to the console for debugging (optional)
  }
}
## RUN THESE COMMANDS ##
sudo systemctl restart logstash
```

- You should be seeing mikrotik-syslogs on the discover page of Kibana now if you setup everything correctly on MikroTik
