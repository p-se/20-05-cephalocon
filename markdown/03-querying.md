<!-- .slide: data-state="section-break-2" id="section-query" data-timing="10s" -->
# Querying


<!-- .slide: data-state="normal" id="query-intro" data-timing="60s" -->
## Query language

### PromQL

* Expressions return either an _Instant Vector_ or a _Range Vector_
<span style="font-size:20px;">(or String or Scalar)</span>
* _Instant Vector_ gives a single value per metric and time stamp

```xquery
> ceph_my_metric
ceph_my_metric{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 723
ceph_my_metric{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 714
ceph_my_metric{ceph_daemon="osd.2",instance="data2.ceph",device="sda"} 236
ceph_my_metric{ceph_daemon="osd.3",instance="data3.ceph",device="sda"} 923
```
* _Range Vector_ gives a range of values per metric and time stamp

```xquery
> ceph_my_metric[30s]
ceph_my_metric{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} [723,872]
ceph_my_metric{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} [714,234]
ceph_my_metric{ceph_daemon="osd.2",instance="data2.ceph",device="sda"} [236,987]
ceph_my_metric{ceph_daemon="osd.3",instance="data3.ceph",device="sda"} [923,872]
```


<!-- .slide: data-state="normal" id="query-labels" data-timing="60s" -->
## Query language - Labels

Labels can be matched with `=`, `!=`, `=~` and `!~`.

<pre><code class="xquery"> > ceph_my_metric{ceph_daemon="osd.0"}
ceph_my_metric{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 723
</code></pre>

<pre class="fragment"><code class="xquery"> > ceph_my_metric{instance=~"data1.\*"}
ceph_my_metric{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 723
ceph_my_metric{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 714
</code></pre>

<pre class="fragment"><code class="xquery"> > ceph_my_metric{device!="sdb"}
ceph_my_metric{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 723
ceph_my_metric{ceph_daemon="osd.2",instance="data2.ceph",device="sda"} 236
ceph_my_metric{ceph_daemon="osd.3",instance="data3.ceph",device="sda"} 923
</code></pre>


<!-- .slide: data-state="normal" id="query-operators" data-timing="60s" -->
## Query language - Operators
### Binary Operators

* For combinations of Scalar and Vector values:
  * Arithmetic ` + - * / % ^`
  * Comparison ` == != < > <= >=`
  * Operate on values if labels match
* Only for two vectors:
  * Set operators ` and or unless`
  * Returns a new vector if labels match
* Vector matching: find an element in the right-hand side for each entry in the left-hand side
  * Can be controlled with keywords `on` and `ignoring`
* Aggregations - operates on all elements of a vector (_across all label dimensions_)
  * `sum stddev max topk count quantile ...`
  * `by` and `without` control result vector labels


<!-- .slide: data-state="normal" id="query-functions" data-timing="60s" -->
## Query language - Functions
### Functions modify do what?

* `abs(v instant-vector)` - modifies all values to their absolute value
* `deriv(v range-vector)` - per-second derivative using linear regression
* `rate(v range-vector)` - per-second average rate in increase
* `time()` - returns the current linux time stamp
* `label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)`
* `<aggregation>_over_time(v range-vector)` - aggregation operators for Range Vectors

<div class="fragment" style="padding-top:170px;">Many more - see https://prometheus.io/docs/prometheus/latest/querying/functions/</div>


<!-- .slide: data-state="normal" id="query-level1" data-timing="60s" -->
## Simple queries

<pre class="fragment"><code class="xquery"> > ceph_my_metric{instance=~"data1.\*"}
ceph_my_metric{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 723
ceph_my_metric{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 714
</code></pre>

<pre class="fragment"><code class="xquery"> > ceph_my_metric_size - ceph_my_metric
{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 2394
{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 9823
{ceph_daemon="osd.2",instance="data2.ceph",device="sda"} 1745
{ceph_daemon="osd.3",instance="data3.ceph",device="sda"} 9976
</code></pre>

<pre class="fragment"><code class="xquery"> > sum(ceph_my_metric{instance=~"data1.\*"})
{} 1437
</code></pre>

<pre class="fragment"><code class="xquery"> > rate(ceph_my_metric{instance=~"data1.\*"}[5m])
{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 5.234236
{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 10.09257
</code></pre>


<!-- .slide: data-state="normal" id="query-level2" data-timing="60s" -->
## Complex queries - Correlating Exporters
### One label to rule them all.

* Correlate our metric with a smart attribute of its drive
* `reallocated_sectors` actually called `smartmon_reallocated_sector_ct_rw_value` <!-- .element style="font-size: 20px;" -->
* Has a `device` label
* How many `reallocated_sectors` for `ceph_my_metric` devices?

<pre class="fragment"><code class="xquery"> > reallocated_sectors and on(device) ceph_my_metric
reallocated_sectors{instance="data1.ceph",device="sda"} 0
reallocated_sectors{instance="data1.ceph",device="sdb"} 0
reallocated_sectors{instance="data2.ceph",device="sda"} 0
reallocated_sectors{instance="data3.ceph",device="sda"} 1
</code></pre>

<pre class="fragment"><code class="xquery"> > reallocated_sectors and on(device) ceph_my_metric{ceph_daemon="osd.1"}
reallocated_sectors{instance="data1.ceph",device="sda"} 0
</code></pre>

Note:
* when labels don't quite match use `label_replace`


<!-- .slide: data-state="normal" id="query-level3" data-timing="60s" -->
## Complex queries - Back to the Future
### Great Scott!

* Get a warning before things go wrong.
* Good to know an OSD is full.
* Better to know when an OSD _will_ be full

<pre class="fragment"><code class="xquery"> > predict_linear(ceph_my_metric[1h], 4*3600)
{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 1298.153
{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 1658.547
{ceph_daemon="osd.2",instance="data2.ceph",device="sda"} 1475.873
{ceph_daemon="osd.3",instance="data3.ceph",device="sda"} 500.239
</code></pre>

<pre class="fragment"><code class="xquery"> > (ceph_my_metric_size - ceph-my_metric) / deriv(ceph_my_metric[1h])
{ceph_daemon="osd.0",instance="data1.ceph",device="sda"} 78973
{ceph_daemon="osd.1",instance="data1.ceph",device="sdb"} 67814
{ceph_daemon="osd.2",instance="data2.ceph",device="sda"} 162
{ceph_daemon="osd.3",instance="data3.ceph",device="sda"} -872
</code></pre>


<!-- .slide: data-state="normal" id="alerts-rules" data-timing="60s" -->
## Alerting - Rules
### PromQL query that results in true/false.
<pre>
<code class="yaml hljs" style="font-size:23px; line-height: 28px;">
groups:
- name: my alerts
  rules:
  - alert: FullIn1h
    expr: (ceph_my_metric_size - ceph-my_metric) / deriv(ceph_my_metric[1h]) < 3600
    for: 5m
    labels:
      severity: critical
      foo: bar
    annotations:
      summary: Needs more!
</code>
</pre>

Note:
* _for_ is optional
* labels are user defined
* Annotations support label based templates


<!-- .slide: data-state="normal" id="more" data-timing="60s" -->
## Get started
Checkout the [monitoring sub-directory in the Ceph repository](https://github.com/ceph/ceph/tree/master/monitoring).

### Much more we didn't cover. <!-- .element style="padding-top: 50px" -->

* Alertmanager configuration - alert channels, inhibition, silencing, grouping
* Visualization - Grafana
* Federation
* Push Gateways
* Recording rules
* Label rewriting
* Security
* Remote storage
* ...


<!-- .slide: data-state="section-break" id="fin" data-timing="60s" -->
# Thank you - Questions?

<div style="color: #02d35f; padding-top: 300px;">
<span style="font-weight: bold;">
Many Thanks to
<a style="color: #02d35f !important;" href="https://github.com/aspiers/presentation-template">Adam Spiers</a>,
<a style="color: #02d35f !important;" href="https://github.com/fghaas/presentation-template/">Florian Haas</a> and 
<a style="color: #02d35f !important;" href="https://github.com/hakimel/reveal.js/">Hakim El-Hattab and contributors</a>
</span>
</div>


<!-- .slide: data-state="normal" id="credits" data-menu-title="QR code" data-timing="0" -->
<div class="qrcode palette-text" id="qrcode-talk" />

<h2><a href="https://jan--f.github.io/20-05-cephalocon/" target="_blank"
       id="talk">https://jan--f.github.io/20-05-cephalocon/</a></h2>

