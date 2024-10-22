---
title: Grafana y Zabbix
---

## Instalación
```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
```
```bash
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```
```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
```bash
sudo apt update
```
```bash
sudo apt -y install grafana
```

* Firewall
```bash
sudo ufw allow from 190.103.31.32 proto tcp to any port 3000
```



https://[your zabbix server ip or domain name]/zabbix/api_jsonrpc.php

sudo ufw allow from 172.16.11.8 proto tcp to any port 8080

sudo ufw allow from 190.103.31.32 proto tcp to any port 8080


http://172.16.11.25:8080/zabbix/api_jsonrpc.php

https://zabbix.fibextelecom.net/zabbix/api_jsonrpc.php