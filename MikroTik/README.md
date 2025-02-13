# MikroTik

# Setting Up Winbox

- Download the Windows 64-bit Winbox from [https://mikrotik.com/download](https://mikrotik.com/download)
- To run winbox on a linux machine, wine is needed (the linux based winbox is buggy so download the windows version then use wine to open it on linux)

```bash
sudo apt install wine
sudo dpkg --add-architecture i386 && apt-get update &&\napt-get install wine32:i386
sudo wine Winbox/winbox64.exe #or wherever you put the winbox app
```

# Setting Up MikroTik router VM

- Go to [https://mikrotik.com/download](https://mikrotik.com/download) and go to the Cloud Hosted Router section and choose the long term 6.49.13 version (or whatever version it is in the future) and click download for the VMDK file
- Go to VMware and open the device by clicking File—>Open then go to where the VMDK is stored and click it
- Login with admin and no password, you will be prompted to add a new password

# Setting up Netflow on Router

- Open winbox:

```bash
sudo wine Winbox/winbox64.exe #or wherever you put the winbox app
```

- A view like this should appear, fill in the IP of the MikroTik router and username and password (make sure the VM is running):

![image.png](image%201.png)

- You should now be seeing this (with only ether1):

![image.png](image%202.png)

- To setup Netflow click on IP—>Traffic Flow then click Targets on the side then click +, a screen like this should appear:

![image.png](image%203.png)

- Fill in the details for Filebeat similar to the screenshot, the destination address is that of where the ELK server is and the port is that of Netflow as shown in the netflow.yml in the Filebeat modules folder

![image.png](image%204.png)

- Make sure it’s enabled and shown in the Traffic Flow Targets

![image.png](image%205.png)

- Make sure it’s enabled on all interfaces as well or at the very least the ether interface that’s one the same network as the ELK server (ether1 in my case)

# Setting up Syslog

- Open winbox:

```bash
sudo wine Winbox/winbox64.exe #or wherever you put the winbox app
```

- A view like this should appear, fill in the IP of the MikroTik router and username and password (make sure the VM is running):

![image.png](image%201.png)

- You should now be seeing this (with only ether1):

![image.png](image%202.png)

- To setup Syslog click on System—>Logging, a screen like this should appear:

![image.png](image%206.png)

- Notice syslog2filebeat is actually syslog2logstash but It was named correctly inititally,  this action is what sends the rules

![image.png](image%207.png)

- Click the + button to add a new action and re-create the following screenshot, replacing only the remote address based on the ELK server

![image.png](image%208.png)

- In the Rules section proceed to add rules with the Topics as shown above with the associated action (in this case syslog2filebeat), make sure they are enabled
- You can now check you’re ELK server for the logs called mikrotik-syslogs in the discover section (Depending on how what you named it in the logstash .conf file)
