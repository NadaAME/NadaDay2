# Filebeat

**installation**

```bash
sudo apt install filebeat

sudo vi /etc/filebeat/filebeat.yml
```

Change the default output to `logstash` not `elasticsearch`

```bash
find the output.elasticsearch section and comment out the following lines by preceding them with a #:

**/etc/metricbeat/metricbeat.yml**

#output.elasticsearch:
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]

```

Then, configure the `output.logstash` section. Uncomment the lines `output.logstash:` and `hosts: ["localhost:5044"]` by removing the `#`. This will configure Filebeat to connect to Logstash on your Elastic Stack server at port `5044`, the port for which we specified a Logstash input earlier:

`/etc/metricbeat/metricbeat.yml`

```
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```

**Functionality of Filebeat can be extended with Filebeat modules.** 

Let’s enable it:
`sudo filebeat modules enable system`

See a list of enabled and disabled modules by running:

```bash
sudo filebeat modules list
```

Load the ingest pipeline for the system module:

```bash
sudo filebeat setup --pipelines --modules system
```

Load the index template:

```bash
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```

As the dashboards load, Filebeat connects to Elasticsearch to check version information. To load dashboards when Logstash is enabled, you need to disable the Logstash output and enable Elasticsearch output:

```bash
sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601
```

Start and enable Filebeat:

```bash
sudo systemctl start filebeat
sudo systemctl enable filebeat
curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'

```

Now open **[Filebeat System] Syslog dashboard ECS** in **Dashboard**
