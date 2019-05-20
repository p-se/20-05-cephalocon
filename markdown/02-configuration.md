<!-- .slide: data-state="section-break-2" id="section-config" data-timing="10s" -->
# Configuration


<!-- .slide: data-state="normal" id="config-intro" data-timing="60s" -->
## Configuration - Prometheus

* Daemon behavior through command line flags
* Runtime aspects via config file
* YAML is lingua franca for prometheus

<pre>
<code class="yaml hljs" style="font-size:20px; line-height: 25px;">
global:
  scrape_interval:     15s
  evaluation_interval: 15s
rule_files: []
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus.ecorp.com:9090']
  - job_name: 'node stats'
    file_sd_configs:
      - files: ['/etc/prometheus/node_exporters/*.yaml']
  - job_name: 'K8s Nodes'
    kubernetes_sd_configs:
      - role: node
</code>
</pre>

Note:
* Daemon behavior includes used port, path and such
* Runtime covers scrape targets, label mangling and rules definition
* k8s sd queries k8s rest api
* many more service discovery integrations
* dns, openstackm ec2, gce


<!-- .slide: data-state="normal" id="config-sd" data-timing="60s" -->
## Service discovery configuration

<div class="col-container">
<div class="col">
<pre>
<code class="yaml hljs" style="font-size:20px; line-height: 25px;">
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus.ecorp.com:9090']
</code>
</pre>
</div>

<div class="col">
<ul>
<li>Ideal for well known names (or addresses)</li>
<li>Prometheus won't notice changes</li>
</ul>
</div>
</div>

<div class="col-container fragment">
<div class="col">
<pre>
<code class="yaml hljs" style="font-size:20px; line-height: 25px;">
  - job_name: 'K8s Nodes'
    kubernetes_sd_configs:
      - role: node
        # api_server: $host
</code>
</pre>
</div>
<div class="col">
<ul>
<li>role can also be service, pod, endpoints or ingress</li>
</ul>
</div>
</div>

<div class="col-container fragment">
<div class="col">
<pre>
<code class="yaml hljs" style="font-size:20px; line-height: 25px;">
  - job_name: 'node stats'
    file_sd_configs:
      - files:
        - '/etc/prometheus/node_exporter/*.yaml'
</code>
</pre>
</div>
<div class="col">
<ul>
<li>files contain `static_config` fragments</li>
<li>very flexible service discovery</li>
<li>match against a file name glob</li>
<li>use config management to populate files</li>
</ul>
</div>
</div>

Note:
* since pull based, prometheus needs to know where to scrape
* static can be dynamic depending on DNS setup
* DNS sd also allows to query different DNS records

* api_server not necessary when running inside k8s

* files contain static_configs
* file_sd most versatile, allows for various partitioning approaches


<!-- .slide: data-state="normal" id="config-deployment" data-timing="60s" -->
## Deployment considerations
<h3>Monitoring is for diagnosing problems.</h3>

<ul>
<li>
Monitoring and Alerting should be resilient.
</li>
<img class="fragment" src="images/monitor-all-the-things.jpg" style="height:
400px; margin: 30px;" />
<li class="fragment">
Monitoring should scale.
</li>
</ul>

Note:
* 2 things important: available and scalable


<!-- .slide: data-state="normal" id="config-setups" data-timing="60s" -->
### Basic setup
<img class="full-screen" src="images/basic_cluster.svg" />


<!-- .slide: data-state="normal" id="config-setups" data-timing="60s" -->
### (Highly) available setup
<img class="full-screen" src="images/ha_cluster.svg" />

Note:
* replicate config somewhere else


<!-- .slide: data-state="normal" id="config-setups" data-timing="60s" -->
### Scalable setup
<img class="full-screen" src="images/scale_cluster.svg" />

Note:
* same config, only targets differ


<!-- .slide: data-state="normal" id="config-setups" data-timing="60s" -->
### Unicorns and rainbows
<img class="full-screen" src="images/scale_ha_cluster.svg" />

Note:
many more configurations possible
