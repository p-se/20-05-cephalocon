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
  external_labels: []
rule_files: []
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus.ecorp.com:9090']
  - job_name: 'node stats'
    file_sd_configs:
      - files: ['/etc/prometheus/SUSE/*.yaml']
</code>
</pre>

Note:
* Daemon behavior includes used port, path and such
* Runtime covers scrape targets, label mangling and rules definition
* In SUSE Enterprise Storage we use salt (in DeepSea)
* many more service discovery integrations
* kubernetes, openstackm ec2, gce


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
