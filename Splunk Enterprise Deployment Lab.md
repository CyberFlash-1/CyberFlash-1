 $${{\color{Lime}\huge{\textsf{Splunk Enterprise Lab\ }}}}\$$


$${{\color{Blue}\huge{\textsf{Overview }}}}\$$

---
In this lab I set up a splunk enviroment consisiting of:
| VM                   | OS              | Role(s)                                                                                                        |
| -------------------- | ----------------|--------------------------------------------------------------------------------------------------------------- |
| Virtual Machine 1    | RHEL 10         | **Universal Forwarder (UF)**                                                                                   |
| Virtual Machine 2    | RHEL 10         | **Search Head (SH)**                                                                                           |
| Virtual Machine 3    | RHEL 10         | **Deployment Server**+ **Monitoring Console**+ **License Manager**                                            |
| Virtual Machine 4    | RHEL 10         | **Cluster Manager**                                                                                            |
| Virtual Machine 5    | RHEL 10         | **Indexer**                                                                                                    |
| Virtual Machine 6    | RHEL 10         | **Indexer**                                                                                                    |
| Virtual Machine 7    | RHEL 10         | **Indexer**                                                                                                    |;



$${{\color{Yellow}\huge{\textsf{Architecture }}}}\$$
<div align="center">
<img width="800" height="600" alt="deployment" src="https://github.com/user-attachments/assets/5fb759df-167a-4c74-94a2-c01eb014a974" /></a>
</div>
  </br>
  </div>
  
<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 1: Download Splunk\ }}}}\$
 
</div>

```
wget -O splunk.tgz 'https://download.splunk.com/products/splunk/releases/9.1.2/linux/splunk-9.1.2-Linux-x86_64.tgz'
tar -xvzf splunk.tgz -C /opt
```

### Create the splunk User from the ROOT User Account 

```
sudo su
useradd splunk
sudo chown -R splunk:splunk /opt/splunk    #### This give the splunk user ownership/permissions for the splunk directory #####
sudo su - splunk
cd /opt/splunk/bin
/opt/splunk/bin/splunk start --accept-license
./splunk stop
```

<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 2: Enable boot-start\ }}}}\$

</div>

```
cd /opt/splunk/bin
./splunk enable boot-start -systemd-managed 1 -create-polkit-rules 1 -user splunk
#### revert back to the splunk user and start splunk ####
sudo su - splunk
cd /opt/splunk/bin
./splunk start
```
<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 3: Enabling Clustering for the Cluster Manager - CM \ }}}}\$

</div>

```
[clustering]
mode=master
pass4SymmKey = <string>
search_factor=2
replication_factor=3
```
<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 4: Enabling Clustering for the Peers(Indexers) \ }}}}\$

</div>

```
[replication_port://9887]

[clustering]
mode=peer
pass4SymmKey = <string> # same as the key:value for the cluster manager
manager_uri=https://CM:8089
```

<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 5: Enabling Indexer Discover on the CM\ }}}}\$

</div>

```
[indexer_discovery]
pass4SymmKey = my_secret # same string is to be used on the UF
polling_rate = 10
[optional]indexerWeightByDiskCapacity = <bool> # determines whether indexer discovery uses weighted load balancing. 
```


<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 6: Enabling Indexer Discover on the UF\ }}}}\$

</div>

```
[indexer_discovery:manager1]
pass4SymmKey = my_secret
manager_uri = https://10.152.31.202:8089

[tcpout:group1]
autoLBFrequency = 30
forceTimebasedAutoLB = true
indexerDiscovery = manager1
useACK=true

[tcpout]
defaultGroup = group1



```

<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 7: Creating the Indexes\ }}}}\$

</div>










<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 8: Configuring the deploymentclient.conf on the UF\ }}}}\$

</div>

```
[deployment-client]
clientName=$hostname

[target-broker:deploymentServer]
target_uri=https://DS:8089

Then run ./splunk show deploy-poll : This should be the IP address of the Deployment Server 

```





<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 9: Creating the serverclass on the Deployment Server \ }}}}\$

</div>


<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 10: Creating an app on the Deployment Server \ }}}}\$

</div>


<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 10: Creating the inputs on the Deployment Server \ }}}}\$

</div>


<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 11: Connecting the SH to the Cluster Manager  \ }}}}\$

</div>

```
splunk edit cluster-config -mode searchhead -manager_uri https://10.160.31.200:8089 -secret my_secret

splunk restart

```

<div align="center">
 
${{\color{Yellow}\large{\textsf{Step 12: Setting up the Monitoring Console  \ }}}}\$

</div>
