#  Centralized Log Aggregation and Monitoring using ELK Stack

>A logging setup on AWS where Filebeat collects Nginx logs from multiple EC2 instances, Logstash processes them, and the logs are stored in Elasticsearch. Everything can then be viewed through a single Kibana dashboard, with alerts configured to detect spikes in errors.

>It's not truly real-time, but the logs are processed quickly enough to give near-real-time visibility and help catch issues before they become bigger problems.

![AWS](https://img.shields.io/badge/AWS-ELK%20Stack-orange?style=flat-square&logo=amazon-aws)
![ELK](https://img.shields.io/badge/ELK-8.19.16-blue?style=flat-square&logo=elastic)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)
![Author](https://img.shields.io/badge/Author-Premkumar%20Patil-purple?style=flat-square)

---

## Project Preview

> **Kibana Nginx Monitoring Dashboard** showing real-time error rate, requests per minute, top client IPs, and HTTP response code breakdown from both application servers.
![alt text](images/120.png)
![alt text](images/137.png)
![alt text](images/138.png)

---

## 📖 Introduction

When something breaks in production, the logs you need are often spread across multiple servers. Figuring out what happened can mean SSHing into each server, searching through log files, and trying to piece everything together. It's slow, and a simple investigation can take much longer than it should.

That's the problem ELK is designed to solve - bringing logs from different servers into one place so you don't have to search through each machine whenever something goes wrong.

I built this project mainly to understand how that pipeline works end to end, rather than just reading about it. Filebeat collects the Nginx logs from each server, Logstash processes and tags them, Elasticsearch stores and indexes the data, and Kibana provides a central place to search and visualize everything. I also added alerts for spikes in error rates so the system can flag potential issues without someone having to constantly watch the dashboard.

---

## Project Overview

This project builds a complete centralized log aggregation system with 3 EC2 instances — one ELK server running all three Elastic components, and two application servers running Nginx and Filebeat — with Kibana dashboards and alerting for error detection.

| What | Details |
|---|---|
| Type | Centralized Log Aggregation and Monitoring |
| Role | Site Reliability Engineer |
| Servers | 3 EC2 instances (1 ELK + 2 App servers) |
| Stack | Elasticsearch + Logstash + Kibana + Filebeat + Nginx |
| OS | Ubuntu Server 26.04 LTS |
| Region | us-east-1 (N. Virginia) |
| Status | Completed |

---

## Architecture

```
App Server 1 (t3.small)          
  Nginx → /var/log/nginx/access.log    
  Filebeat → ships logs              
                    ↓
                    ↓ port 5044
                    ↓
            ELK Server (c7i-flex.large)
              Logstash  → parses + tags errors
              Elasticsearch → stores + indexes
              Kibana → dashboard + alerts
                    ↑
                    ↑ port 5044
                    ↑
App Server 2 (t3.small)
  Nginx → /var/log/nginx/access.log
  Filebeat → ships logs
```
![ELK STACK Architecture](images/Architecture.png)

---

## Project Structure

```
elk-stack-monitoring/
├── README.md
├── image/
│   ├── 01-ec2-instances-running.png
│   ├── 02-elk-security-group.png
│   ├── 03-app-security-group.png
│   ├── ... (all 21 screenshots)
│   └── 21-all-services-running.png
├── config/
│   ├── elasticsearch.yml
│   ├── logstash/
│   │   └── nginx-pipeline.conf
│   └── filebeat/
│       └── filebeat.yml
└── dashboard/
    └── nginx-monitoring-dashboard.ndjson
```
---
## Complete Data Flow

```
1. User hits Nginx on port 80
2. Nginx writes request to /var/log/nginx/access.log
3. Filebeat watches log file — detects new lines within seconds
4. Filebeat ships raw log lines to Logstash on port 5044
5. Logstash grok pattern parses each line:
   Extracts: client_ip, method, request, response_code, bytes
6. Logstash tags 4xx responses as "client_error" and "error"
7. Logstash tags 5xx responses as "server_error" and "error"  
8. Logstash sends structured JSON to Elasticsearch on port 9200
9. Elasticsearch stores in daily index: nginx-logs-YYYY.MM.dd
10. Kibana reads from Elasticsearch — dashboard updates in real time
11. Alert rule checks every 1 minute — fires if errors > 10 in 5 min
```

---

## Infrastructure

| Server | Instance Type | Services | Purpose |
|---|---|---|---|
| elk-server | c7i-flex.large | Elasticsearch, Logstash, Kibana | Central log processing and visualization |
| app-server-1 | t3.small | Nginx, Filebeat | Application server — generates logs |
| app-server-2 | t3.small | Nginx, Filebeat | Application server — generates logs |

---

## Security Groups

**ELK Server (elk-sg):**

| Port | Protocol | Source | Purpose |
|---|---|---|---|
| 22 | SSH | My IP | SSH access |
| 9200 | TCP | 0.0.0.0/0 | Elasticsearch API |
| 5044 | TCP | 0.0.0.0/0 | Logstash Beats input |
| 5601 | TCP | My IP | Kibana UI |
| 80 | HTTP | 0.0.0.0/0 | Nginx web traffic |

---

## Part 1 — EC2 Instances

Launch all 3 instances in us-east-1
with correct security groups.

Instance 1 — ELK Server:
→ AMI: Ubuntu 26.04 LTS 
→ Instance type: c7i-flex.large
   (since t2.medium was not available
    on  free tier account)
→ Storage: 20 GB ← increase during launch
→ Security group: ELK-sg

Instance 2 — App Server 1:
→ AMI: Ubuntu 26.04 LTS
→ Instance type: t3.small
→ Storage: 8 GB default is fine
→ Security group: ELK-sg

Instance 3 — App Server 2:
→ AMI: Ubuntu 26.04 LTS
→ Instance type: t3.small
→ Storage: 8 GB default is fine
→ Security group: ELK-sg

> **ELK server security group** showing ports 22, 9200, 5044, 5601 inbound rules.


![alt text](images/1.png)
![alt text](images/4.png)
![alt text](images/3.png)


> **All 3 EC2 instances running** — elk-server (c7i-flex.large), app-server-1 (t3.small), app-server-2 (t3.small).

![alt text](images/6.png)
![alt text](images/7.png)
![alt text](images/8.png)
![alt text](images/9.png)
![alt text](images/10.png)
![alt text](images/11.png)
![alt text](images/12.png)
![alt text](images/13.png)
![alt text](images/14.png)
![alt text](images/15.png)
![alt text](images/16.png)
![alt text](images/17.png)

---

## Part 2 — Elasticsearch Setup

SSH into the ELK server and install Elasticsearch 8.x. Disable security for this lab and configure to listen on all network interfaces so Logstash and Kibana can connect.

```bash
# Add Elastic repository
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
sudo apt install elasticsearch -y
```
![alt text](images/27.png)
![alt text](images/28.png)
![alt text](images/29.png)


Elasticsearch configuration (`/etc/elasticsearch/elasticsearch.yml`):

```yaml
cluster.name: elk-lab
node.name: elk-node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
```
![alt text](images/31.png)
![alt text](images/30.png)

JVM heap size (`/etc/elasticsearch/jvm.options.d/heap.options`):

```
-Xms512m
-Xmx512m
```

```bash
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch

# Test
curl http://localhost:9200
```

> **Elasticsearch responding** — curl returning cluster JSON with cluster_name: elk-lab.

![alt text](images/35.png)
![alt text](images/36.png)
![alt text](images/37.png)

---

## Part 3 — Logstash Pipeline

Install Logstash and create the Nginx log parsing pipeline. The grok pattern extracts structured fields from raw Nginx log lines and tags 4xx and 5xx errors automatically.

```bash
sudo apt install logstash -y
```

Pipeline configuration (`/etc/logstash/conf.d/nginx-pipeline.conf`):

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  if [log_type] == "nginx_access" {
    grok {
      match => {
        "message" => [
          '%{IPORHOST:client_ip} - %{DATA:user} \[%{HTTPDATE:timestamp}\] \"%{WORD:method} %{DATA:request} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code:int} %{NUMBER:bytes:int} \"%{DATA:referrer}\" \"%{DATA:user_agent}\"',
          '%{IPORHOST:client_ip} - %{DATA:user} \[%{HTTPDATE:timestamp}\] \"%{WORD:method} %{DATA:request} HTTP/%{NUMBER:http_version}\" %{NUMBER:response_code:int} %{NUMBER:bytes:int}'
        ]
      }
    }
    if "_grokparsefailure" not in [tags] {
      if [response_code] >= 500 {
        mutate { add_tag => ["server_error", "error"] }
      } else if [response_code] >= 400 {
        mutate { add_tag => ["client_error", "error"] }
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "nginx-logs-%{+YYYY.MM.dd}"
  }
}
```

```bash
sudo systemctl start logstash
sudo systemctl enable logstash
```

> **Logstash status** showing active (running) — pipeline listening on port 5044.

![alt text](images/38.png)
![alt text](images/39.png)
![alt text](images/40.png)
![alt text](images/41.png)

---

## Part 4 — Kibana Setup

Install Kibana and configure it to connect to Elasticsearch and listen on all interfaces so the browser can access it.

```bash
sudo apt install kibana -y
```

Kibana configuration (`/etc/kibana/kibana.yml`):

```yaml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
xpack.encryptedSavedObjects.encryptionKey: "your-32-char-key-here"
xpack.reporting.encryptionKey: "your-32-char-key-here"
xpack.security.encryptionKey: "your-32-char-key-here"
```

```bash
sudo systemctl start kibana
sudo systemctl enable kibana
```

> **Kibana home page** accessible at http://ELK_PUBLIC_IP:5601 showing the Elastic home screen.

![alt text](images/43.png)
![alt text](images/48.png)
![alt text](images/50.png)
![alt text](images/63.png)
![alt text](images/78.png)

---

## Part 5 — Nginx and Filebeat on App Servers

Install Nginx and Filebeat on both app servers. Filebeat watches the Nginx log files and ships new lines to Logstash using the ELK server's private IP to keep traffic inside the VPC.

```bash
# Install Nginx
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# Install Filebeat
sudo apt install filebeat -y
```

Filebeat configuration (`/etc/filebeat/filebeat.yml`):

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    fields:
      log_type: nginx_access
      server: app-server-1
    fields_under_root: true

  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    fields:
      log_type: nginx_error
      server: app-server-1
    fields_under_root: true

output.logstash:
  hosts: ["ELK_PRIVATE_IP:5044"]
```

```bash
sudo systemctl start filebeat
sudo systemctl enable filebeat

# Test connection to Logstash
sudo filebeat test output
```

> **Filebeat test output on app-server-1** — showing parse host OK, dns lookup OK, dial up OK, talk to server OK.

![alt text](images/54.png)
![alt text](images/55.png)
![alt text](images/56.png)
![alt text](images/57.png)
![alt text](images/58.png)
![alt text](images/60.png)
![alt text](images/61.png)
![alt text](images/62.png)


> **Filebeat test output on app-server-2** — same successful connection to Logstash.

![alt text](images/66.png)
![alt text](images/67.png)
![alt text](images/68.png)
![alt text](images/70.png)
![alt text](images/71.png)
![alt text](images/73.png)

---

## Part 6 — Verify Log Pipeline

Generate test traffic on both app servers and confirm logs are flowing through to Elasticsearch.

```bash
# Generate normal requests (run on both app servers)
for i in {1..50}; do curl -s http://localhost/ > /dev/null; done

# Generate 404 errors
for i in {1..20}; do curl -s http://localhost/notfound-$i > /dev/null; done
```
![alt text](images/62.png)


Verify on ELK server:

```bash
# Check Elasticsearch index was created
curl http://localhost:9200/_cat/indices?v

# Confirm parsed fields exist
curl "http://localhost:9200/nginx-logs-*/_search?pretty&size=1" | \
  grep -E "client_ip|response_code|method"
```
![alt text](images/74.png)
![alt text](images/75.png)
![alt text](images/76.png)
![alt text](images/77.png)

---

## Part 7 — Kibana Dashboard

Create the Nginx Monitoring Dashboard with 4 visualizations in Kibana Lens.

### Visualization 1 — Error Rate Over Time

- Chart type: Line
- Horizontal axis: @timestamp
- Vertical axis: Count of records
- Filter: `tags : "error"`
- Shows: 4xx and 5xx error count over time

### Visualization 2 — Requests Per Minute

- Chart type: Area
- Horizontal axis: @timestamp
- Vertical axis: Count of records
- No filter — shows total traffic

### Visualization 3 — Top Client IPs

- Chart type: Pie
- Slice by: client_ip.keyword
- Metric: Count of records
- Shows: top 10 IP addresses making requests

### Visualization 4 — HTTP Response Code Breakdown

- Chart type: Pie
- Slice by: response_code
- Metric: Count of records
- Shows: distribution of 200, 404, 405 etc.

> **Error Rate Over Time** — line chart showing error spike when 404 traffic was generated.

![alt text](images/114.png)

> **Requests Per Minute** — area chart showing total traffic volume over time.

![Requests Per Minute](images/116.png)

> **Top Client IPs** — pie chart showing which IPs generated the most traffic.

![Top Client IPs](images/117.png)

> **HTTP Response Code Breakdown** — pie chart showing 200: 64.94%, 404: 34.76%, 405: 0.3%.

![Response Codes](images/119.png)

> **Full Nginx Monitoring Dashboard** — all 4 visualizations on one screen.

![Full Dashboard](images/120.png)

---

## Part 8 — Alert Configuration

Create an alert rule in Kibana that fires automatically when the error count exceeds 10 in 5 minutes — enabling proactive error detection without manual log checking.

```
Stack Management → Rules → Create rule
Rule type: Elasticsearch query
Query type: KQL or Lucene
Data view: Nginx Logs
Query: tags : "error"
Condition: WHEN count() IS ABOVE 10 FOR THE LAST 5 minutes
Schedule: Every 1 minute
Rule name: High Error Rate Alert
```

> **Alert rule creation page** — showing High Error Rate Alert with query and threshold configured.

![Alert Rule Config](images/129.png)
![Alert Rule Config](images/130.png)
![Alert Rule Config](images/134.png)
![Alert Rule Config](images/135.png)
![Alert Rule Config](images/136.png)


> **Rules list** — showing High Error Rate Alert saved with status OK and last run time.

![Alert Rule saved](images/137.png)
![Alert Rule saved](images/138.png)

---



---

## 💡 What I Learned

Probably the biggest thing I got out of this project was actually understanding what each part of ELK is for, instead of just knowing the acronym. Filebeat is basically a lightweight log shipper — it collects logs and forwards them. Logstash is where the processing happens, including parsing and transforming the logs. Elasticsearch stores and indexes the data so it can be searched efficiently.

Grok patterns took me a while to understand. You're essentially using pattern matching to break an unstructured Nginx log line into individual fields that can be queried and filtered later. Once it clicked, it made a lot more sense, but the syntax definitely wasn't intuitive at first.

Kibana Lens was another useful discovery. I expected to need a query language just to build basic charts, but a lot of it can be done through drag-and-drop. Selecting fields and putting them on axes or into slices is enough to build things like line charts and pie charts from the log data.
I also learned an important networking lesson while testing the setup. I initially had Filebeat sending logs to the Logstash public IP. It worked, but since both servers were inside the same VPC, using the private IP made more sense for internal communication and avoided exposing the Logstash endpoint publicly.

Elasticsearch and memory were another lesson. I initially put Elasticsearch on a `t2.small`, thinking it would be enough for a small test setup, but it kept getting killed because of memory pressure. I had to increase the instance resources and configure the JVM heap properly — in my setup, `512MB` on a `c7i-flex.large` — before it became stable.

Kibana alerting was fairly straightforward once I got to it. You define a rule, have it evaluate the Elasticsearch data on a schedule, and trigger an alert when a condition or threshold is met. It's useful because the dashboard doesn't have to be watched constantly just to catch an increase in errors.

---

## Project Summary

This project started with a simple problem: when something goes wrong in production, how do you quickly figure out what happened?

If the logs are spread across multiple servers, troubleshooting means connecting to each machine, searching through different log files, and trying to piece the events together. The more servers you have, the harder that becomes.

The goal of this project was to bring those logs into one place using the ELK Stack. Filebeat collects the Nginx logs, Logstash processes and parses them, Elasticsearch stores and indexes them, and Kibana provides a central place to search and visualize the data.

One of the more interesting parts was getting the Logstash Grok pattern right. Nginx logs start as plain text, with information such as the client IP, timestamp, HTTP method, URL, response code, and response size all contained in a single line. The Grok pattern breaks that line into separate fields so Elasticsearch can work with the data properly. Once the logs are structured, I can filter for things like `response_code: 404`, look at requests by `client_ip`, or identify logs tagged as errors.

I also added alerting so the system doesn't depend on someone constantly watching the Kibana dashboard. The alert checks the log data periodically and can trigger when the error rate crosses a defined threshold.

The main thing I wanted to understand was the complete flow — from a request generating a log on an EC2 server, through Filebeat and Logstash, into Elasticsearch, and finally into Kibana where the data can be searched, visualized, and monitored.


---

## Conclusion

Building this project made it pretty clear why centralized logging matters once you've got more than a couple of servers running. Instead of hopping between machines and digging through separate log files, everything ends up in one place where you can actually search and analyze it.

What helped more than anything was working through the whole pipeline instead of learning each ELK component in isolation. Watching a log go from Filebeat, get parsed by Logstash, land in Elasticsearch, and finally show up in Kibana made the relationship between all four pieces much clearer than reading about them separately.

The Grok patterns were probably the part I spent the most time on. That's not something you fully understand from theory alone. The same goes for figuring out why Elasticsearch kept crashing until I sorted out the memory settings, and learning why using private IPs between instances made more sense than defaulting to public IPs.

If there's one thing I'd take away from this project, it's that logs become much more useful when they're structured and centralized. Having unstructured log files sitting on different servers isn't very helpful when something breaks and you need to figure out what happened quickly.

---

## 👨‍💻 Author

**Premkumar Patil**
Cloud & DevOps Enthusiast | Learning AWS and SRE

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/premkumar-patil-bb9471253)
[![Medium](https://img.shields.io/badge/Medium-Follow-black?style=flat-square&logo=medium)](https://medium.com/@premkpatil9423)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-grey?style=flat-square&logo=github)](https://github.com/Premkumar-Patil001)

---

## 📌 Tags

`AWS` `ELK Stack` `Elasticsearch` `Logstash` `Kibana` `Filebeat` `Nginx` `Log Aggregation` `Centralized Logging` `Monitoring` `Alerting` `Grok` `SRE` `DevOps` `Ubuntu` `EC2` `Cloud Engineering`
