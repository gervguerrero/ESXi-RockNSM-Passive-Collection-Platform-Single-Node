# ⚙️ ESXi-RockNSM-Passive-Collection-Platform-Single-Node 🔵

This showcases a project similar to what I have built and maintained in my professional work as a Junior Security Engineer while in the military. This project is meant to be used on a network as an additional security analysis tool for cyber threat hunting.

This platform is based off a bare-metal installation of RockNSM using a Gigamon Network TAP collecting network traffic for threat hunting analysis.

[RockNSM](https://rocknsm.io/) is an open source network security monitoring platform built on CentOS that uses the Elasticsearch stack (ELK) with Zeek and suricata.

## High Level Overview 

Shown below, we can see an overview of the toolset collecting raw network traffic from a client's network via a Gigamon TAP.

![Single Node](https://github.com/gervguerrero/ESXi-RockNSM-Passive-Collection-Platform-Single-Node/assets/140366635/edd35056-dcb3-4419-9b1a-668db78869b5)

The way this is used, is that the Gigamon TAP is placed between two network devices in the client's network to passively collect traffic and creates a bit for bit copy of it.

It will then send that bit for bit copy via a Tool Port (Tool Ports send data in Gigamon TAP devices) to the physical monitoring ingestion port "P2P1" of the PowerEdge R440 that houses the RockNSM software. RockNSM will process that data through Zeek, Suricata, and it's Elasticsearch (ELK) Stack to make it ready for analysis.

Security Analysts using their workstations will query that data using the Kibana web page and perform analysis to find cyber threats on the client's network. 

This is all done passively that does not involve directly connecting the client's network, nor does it involve installing and deploying agents or custom software to the client's network. This passive collection model works in favor of network owners or administrators that are hesitant to deploy Endpoint Detection & Response tools, or giving responding security teams privileged access.

## Network Traffic Data Flow

![image](https://github.com/gervguerrero/ESXi-RockNSM-Passive-Collection-Platform-Single-Node/assets/140366635/234af374-9f1e-4f28-996e-7a645219c90a)

This picture above represents RockNSM's services and how data is processed. It starts upon ingestion of network traffic into the NIC, where a service called AF_Packet splits the data into 3 streams for each service:

- Zeek
- Suricata
- Stenographer

**Zeek** is a metadata protocol analyzer and labels network traffic based on its actions and behavior, rather than its port or protocol names. 

**Suricata** is a multi-threaded IDS/IPS tool with metadata analysis features that can be customizable rules similar to snort.

**Stenographer** is a full PCAP tool that writes to disk quickly, and can provide the PCAP to analysts when queried.



While Stenographer writes the data capture to PCAP, Zeek and Suricata generate their service specific logs and analyze the traffic. 

Zeek will send it's logs through **FSF (File Scanning Framework). FSF is a recursive file scanning solution to provide static file analysis on file types of interest. It uses a client-server model and can watch for new extracted files.

Those logs along with the logs Zeek and Suricata generated are then sent to **Kafka**, which is a fault  tolerant message queue used to transfer data from Zeek to the start of the ELK Stack - Logstash. 

**Kafka** has the ability to hold large amounts of data by storing data to disk which is non-volatile unlike most message queues which store data in RAM which is. This ability keeps Logstash from being overflown with data while Zeek and Suricata continue to push data. From Kafka, the data is pushed to the beginning of the ELK Stack (Elasticsearch).

### ELK Stack (Elasticsearch)

**Logstash** is a data processing pipeline that will enrich data by adding tags to make it searchable before sent to Elasticsearch.

**Elasticsearch** will index and store the data making it ready to be queried. It is a powerful search/query engine that is used in common applications. 

**Kibana** is the visualization application of the ELK stack and allows for visual exploration and real time analysis of your data in Elasticsearch. Security Analysts will use Kibana to build tables and visuals to perform threat hunting analysis with their queries. 

Below is a simple DNS visual dashboard in Kibana that a Security Analyst would use:

![image](https://github.com/gervguerrero/ESXi-RockNSM-Passive-Collection-Platform-Single-Node/assets/140366635/a5862dcf-dd4e-48a9-9fe3-c89a7c060801)

🔵 **To see how a Security Analyst would use the ELK (Elasticsearch) Stack to find anomalies and threats in networks, see my page:** [LLMNR-NBT-NS-Poisoning-DETECTION
](https://github.com/gervguerrero/LLMNR-NBT-NS-Poisoning-DETECTION). 🔵


## Hardware List
**pfSense Netgate SG-3100**
- Router, Firewall rules, VLAN, NTP/DNS server

**Dell R440/R640**
- Analyzes and Stores the traffic

**Gigamon TAP GigaVUE HB1**
- Ingests the Client's traffic Passively with fail-open power over ethernet properties 

**Cisco Switch 3850X**
- Layer 3 Switch, VLAN capability providing basic connectivity 


**Resources:**

https://rocknsm.io/

https://docs.rocknsm.io/

https://github.com/rocknsm

https://www.gigamon.com/
