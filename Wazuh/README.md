# Wazuh

Wazuh Server Setup and Rule Modification Process and Slack Integration

# Install Wazuh

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -o
```

- Make sure to save login credentials provided at the end of the setup, necessary to login
- Open Web URL and insert username and password

# Enable all needed services

```bash
sudo systemctl enable wazuh-manager
sudo systemctl enable wazuh-indexer
sudo systemctl enable wazuh-dashboard
sudo systemctl enable filebeat
```

# Add Wazuh Agent Elasticsearch Device

After logging in:

- Go to Server Management —>Endpoints Summary —>Deploy new agent
- Follow the steps, the should be self explanatory, they require you to run a command on the desired device, in our case the ELK Device.

Sample Code to run on device that we want an agent on:

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.0-1_amd64.deb && sudo WAZUH_MANAGER='192.168.153.135' WAZUH_AGENT_NAME='ELK-LINUX' dpkg -i ./wazuh-agent_4.9.0-1_amd64.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

# Setting up the Wazuh Agent to monitor logs

- To allow the ossec user to read the elastic server logs (to check for authentication failures) we must allow all users to be able to read it (not sure if this is the ideal solution for security reasons…)

```bash
sudo chmod +644 /var/log/elasticsearch/elk-kali.log
```

- Then proceed to the ossec.conf file to make the agent monitor any changes in this log, add the <localfile>… </localfile> section where ossec_config is and where other <localfile> configs are found as well like journald’s localfile config (needed for authentication errors on the ubuntu machine itself, not the ELK server)

```bash
<ossec_config>
 ..
 ..
 <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/elasticsearch/elk-kali.log</location>
 </localfile>
  ..
  ..
</ossec_config>  
```

- Validate config and restart agent service if everything is successful

```bash
sudo /var/ossec/bin/ossec-control validate
sudo systemctl restart wazuh-agent
```

# Confirming Presence of Logs

- Now go to back to the wazuh web app and check for logs coming from elk-kali.log
- To test for logs, go to [https://192.168.153.128:9200/](https://192.168.153.128:9200/) on the ELK server (elasticsearch server link) and do a failed login attempt.
- then go to threat hunting on your wazuh server and click refresh.
- An alert should appear with the location being /var/log/elasticsearch/elk-kali.log

# Creating new rule for authentication failures

- On the wazuh web app go to Server Management—>Rules and click Manage rule files
- Proceed to search up local_rules.xml and click the edit button on it
- Copy paste these rules:

```bash
<group name="slack,">

  <rule id="100002" level="13">
    <if_sid>5503</if_sid>
    <description>User Authentication Failure: Ubuntu</description>
    <group>authentication_failed,syslog,pam,</group>
  </rule>
  
  <rule id="100003" level="13">
    <if_sid>2501</if_sid>
    <description>User Authentication Failure: Elasticsearch</description>
    <group>authentication_failed,syslog,access_control,</group>
  </rule>

</group>
```

- These rules trigger if the previous rule IDs (ex rule id 5503) trigger, you can check what rule IDs trigger when inputting wrong passwords on ubuntu itself or on the ELK server
- I made all these rules start with “slack,” for their group name and each one has a bit more unique group descriptions like “syslog,pam,” etc…
- I added a custom description for this rule rather than the default full log to be displayed, this will now appear in the alert ex: rule.description: User Authentication Failure: Elasticsearch
- I set all the rules to level 13 which is part of what wazuh considers “High” severity
- Proceed to save and check if the rule works
- Allow Wazuh Manager to restart
- Check results by testing a failed login again

# Setting up Slack Integration

- Proceed back to ossec config file on the Wazuh Server

```bash
sudo gedit /var/ossec/etc/ossec.conf &
```

- Add the slack integration with a custom webhook, follow this tutorial to create a webhook: [https://api.slack.com/messaging/webhooks](https://api.slack.com/messaging/webhooks)

```bash
  </remote>
  ..
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/....</hook_url> <!-- Replace with your Slack hook URL -->
    <alert_format>json</alert_format>
  </integration>
  ..
  <!-- Policy monitoring -->
```

- Restart Wazuh-manager and check if alerts are now being sent

```bash
sudo systemctl restart wazuh-manager
```

- You should now be able to see slack alerts all levels of warning on the slack channel you created
