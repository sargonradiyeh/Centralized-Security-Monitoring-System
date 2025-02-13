# Checkmk

# Setting Up Checkmk Server

```bash
wget https://download.checkmk.com/checkmk/2.3.0p18/check-mk-raw-2.3.0p18_0.jammy_amd64.deb
sudo apt install ./check-mk-raw-2.3.0p18_0.jammy_amd64.deb
sudo omd create mkmonitoring
sudo omd start
```

- Make sure you copy paste the generated password and save it somewhere
- Proceed to login to [192.168.153.135/mkmonitoring](http://192.168.153.135/mkmonitoring/) or [http://ubuntu-virtual-machine/](http://ubuntu-virtual-machine/)

# Setting Up Checkmk Agent

- From the web interface, go to Setup—>Agent—>Linux and download what file you need, copy it over to the device where we want to monitor ElasticSearch
- Run the setup of the agent
- Go back to the web interface go to Setup—>Hosts and put a Host name, IPv4 address (192.168.153.128) and make it Configured API integrations and Checkmk agent
- Click save and run service configuration, go through list to decide what you want to monitor on the device
- Click on the changes and click active on the selected sites to confirm changes
- Everything should be monitored now

# Setting Up ElasticSearch monitoring

- First and foremost move elasticsearch.pem to /etc/ssl/certs/ and then run update-db
- Go to Setup—>Agents—>Other Integrations and click Elasticsearch which is under applications
- Click add rule and tick Username and Password and SSL certificate verification
- Add to Host names to query “192.168.153.128”
- Username is elastic
- Password is ……
- Protocol HTTPS
- Port 9200
- Go to conditions and add explicit hosts and choose the ELK server device with the agent on it.
- Go back to the host and click run service discovery
- Add all Elasticsearch services and save the changes like previously
- Everything should appear now
