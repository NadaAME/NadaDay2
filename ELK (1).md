# ELK

### Elastic search

**Install elasticsearch**

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
echo "deb https://artifacts.elastic.co/packages/oss-7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch
```

**Enable local network**

```bash
sudo vi /etc/elasticsearch/elasticsearch.yml

#uncomment line
network.host: localhost
```

**Start Elasticsearch**

```bash
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
curl -X GET "localhost:9200"
```

### Kibana

```bash
sudo apt install kibana
sudo systemctl enable kibana
sudo systemctl start kibana
```

### Nginx

**Install nginx**

```bash
sudo apt update
sudo apt-get install nginx
sudo ufw app list
sudo ufw allow 'Nginx HTTP'
systemctl status nginx
sudo vi /etc/nginx/sites-available/kibana
```

Reverse proxy kibana

```bash
server {
    listen 80;

    server_name <YOURSERVERIP>;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Enable**

```bash
sudo ln -s /etc/nginx/sites-available/kibana /etc/nginx/sites-enabled/kibana

#check the configuration for syntax errors
sudo nginx -t
sudo systemctl reload nginx

sudo ufw allow 'Nginx Full'

```

**Check kibana**

`http://your_domain/status`

### Logstash

```bash
sudo apt install logstash
```

**Configure beats**

**Beats input**

```bash
sudo vi /etc/logstash/conf.d/02-beats-input.conf

input {
  beats {
    port => 5044
  }
}
```

**elasticsearch output**

```bash
sudo vi /etc/logstash/conf.d/30-elasticsearch-output.conf

output {
  if [@metadata][pipeline] {
	elasticsearch {
  	hosts => ["localhost:9200"]
  	manage_template => false
  	index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  	pipeline => "%{[@metadata][pipeline]}"
	}
  } else {
	elasticsearch {
  	hosts => ["localhost:9200"]
  	manage_template => false
  	index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
	}
  }
}
```

Test your Logstash configuration with this command:

```bash
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```

S**tart and enable Logstash**

```bash
sudo systemctl start logstash
sudo systemctl enable logstash
```