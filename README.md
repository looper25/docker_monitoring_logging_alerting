If you have any feedback regarding this monitoring/logging/alerting suite, any ideas for improvement, fixes, questions or comments, please feel free to contact me or do a PR!

### What is this?

This is an out of the box monitoring, logging and alerting suite for [Docker](https://www.docker.com/)-hosts and their containers, complete with dashboards to monitor and explore your host and container logs and metrics.

Monitoring: [cAdvisor](https://github.com/google/cadvisor) and [node_exporter](https://github.com/prometheus/node_exporter) for collection, [Prometheus](https://prometheus.io/) for storage, [Grafana](http://grafana.org/) for visualisation.

Logging: [Filebeat](https://www.elastic.co/products/beats/filebeat) for collection and log-collection and forwarding, [Logstash](https://www.elastic.co/products/logstash) for aggregation and processing, [Elasticsearch](https://www.elastic.co/products/elasticsearch) as datastore/backend and [Kibana](https://www.elastic.co/products/kibana) as the frontend.

Alerting: [elastalert](https://github.com/Yelp/elastalert) as a drop-in for Elastic.io's [Watcher](https://www.elastic.co/products/watcher) for alerts triggered by certain container or host log events and Prometheus' [Alertmanager](https://github.com/prometheus/alertmanager) for alerts regarding metrics.

![grafana_screenshot](https://github.com/uschtwill/docker_monitoring_logging/blob/master/screenshots/screenshot_grafana.png "Grafana Screenshot")

![kibana_screenshot](https://github.com/uschtwill/docker_monitoring_logging/blob/master/screenshots/screenshot_kibana.png "Kibana Screenshot")

![alerts_screenshot](https://github.com/uschtwill/docker_monitoring_logging/blob/master/screenshots/screenshot_alerts.png "Alerts Screenshot")

WARNING: This configuration is for testing purposes only. As it is simply forwarding ports at the moment, if your box is accessible publicly, all your logs and metrics will be out in the open. Switch off port forwarding before using this in an "online" environment.

The Grafana dashboard (a bit slimmed down) can also be found on grafana.net: [https://grafana.net/dashboards/395](https://grafana.net/dashboards/395).


### How to set it up?

This suite comes with storage directories for Kibana and Grafana that contain the configuration for the data sources and dashboards. these directories will be mounted into the two containers as volumes. This is for your convenience and eliminates some manual setup steps.

1. `git clone` this repository: `git clone https://github.com/uschtwill/docker_monitoring_logging_alerting.git`
2. `cd` into the folder: `cd docker_monitoring_logging_alerting`
3. Run the setup script: `sh setup.sh`
4. Enjoy and explore your logs and metrics:
  * To explore your logs: <a href="http://localhost:5601/app/kibana#/discover" target="_blank">localhost:5601/app/kibana#/discover</a>.
  - To explore your logging metrics: <a href="http://localhost:5601/app/kibana#/dashboard/Exploration" target="_blank">localhost:5601/app/kibana#/dashboard/Exploration</a>.
  + To see your most important container and host metrics at a glance: <a href="http://localhost:3000/dashboard/db/main-overview" target="_blank">localhost:3000/dashboard/db/main-overview</a>.
  * To explore any metric that's collected without having to build queries: <a href="http://localhost:3000/dashboard/db/data-exploration" target="_blank">localhost:3000/dashboard/db/data-exploration</a>.
  * To see all monitoring alerts and their status in prometheus: <a href="http://localhost:9090/alerts" target="_blank">localhost:9090/alerts</a>.
  * To manage your monitoring alerts (e.g. silence them) in Alertmanager: <a href="http://localhost:9093/#/alerts" target="_blank">localhost:9093/#/alerts</a>. Elastalert (logging alerts) unfortunately does not have a frontend.
  * Just to see what the cAdvisor frontend  looks like (you'll use Grafana for looking at monitoring metrics anyways): <a href="http://localhost:8080/containers/" target="_blank">localhost:8080/containers/</a>
  * To say hello to your Elasticsearch instance: `curl localhost:9200`.
5. Run any containers with the same logging options as defined in this suite's `docker-compose.yml`and add a `container_group` label to enable monitoring, logging and alerting for them.
6. AFTER you're done testing this suite and you want to revert to the state before setting up, run the cleanup script to clean up after yourself: `sh cleanup.sh`


### Alerting and Annotations in Grafana

This suite uses elastalert and Alertmanager for alerting. Rules for logging alerts (elastalert) go into ./elastalert/rules/ and rules for monitoring alerts (Alertmanager) go into ./prometheus/rules/. Alertmanager only takes care of the communications part the monitoring alerts, the rules themselves are defined "in" Prometheus.

Both Alertmanager and elastalert can be configured to send their alerts to various outputs. In this suite, Logstash and Slack are set up. The integration with Logstash works out of the box, for adding Slack you will need to insert your webhook url.

The alerts that are sent to Logstash can be checked by looking at the 'logstash-alerts' index in Kibana. Apart from functioning as a first output, sending and storing the alerts to Elasticsearch via Logstash is also neat because it allows us to query them from Grafana and have them imported to its Dashboards as annotations.

![annotations_screenshot](https://github.com/uschtwill/docker_monitoring_logging/blob/master/screenshots/screenshot_annotations.png "Annotations Screenshot")

The monitoring alerting rules, which are stored in the Prometheus directory, contain a fake alert that should be firing from the beginning and demonstrates the concept. Find it and comment it out to have some peace. Also, there should be logging alerts coming in soon as well, this suite by itself already consists of 10 containers, and something is always complaining. Of course you can also force things by breaking stuff yourself - the `blanket_log-level_catch.yaml` rule that's already set up should catch it.

If you're annoyed by non-events repeatedly triggering alerts, throw them in ./logstash/config/31-non-events.conf in order for logstash to silence them by overwritting their log_level upon import.


### Grafana/Prometheus Query Building

Unfortunately Grafana doesn't appear to have a [fancy query builder](https://youtu.be/sKNZMtoSHN4?t=2m14s) for Prometheus as it has for Graphite or InfluxDB, instead one has to plainly type out one's queries.

Alas, when building Grafana graphs/dashboards with Prometheus as a data storage, knowing it's query dsl and metric types is important. This also means, that documentation about using Grafana with an InfluxDB won't help you much, further narrowing down the number of available resources. This is kind of unfortunate.

Here you can find the official documentation for Prometheus on both the query dsl and the metric types:


[Information on Prometheus Querying](https://prometheus.io/docs/querying/basics/)

[Information on Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/)

Furthermore, since I couldn't find proper documentation on the metrics cAdvisor and Prometheus/Node-Exporter expose, I decided to just take the info from the /metrics entpoints and bring it into a human-readable format.

Check them [here](https://github.com/uschtwill/docker_monitoring_logging/tree/master/metrics-explained-for-grafana-query-building). Combining the information on the exposed metrics themselves with that on Prometheus' query dsl and metric types, you should be good to go to build some beautiful dashboards yourself.
