# logging-with-fluentd

Command to list indices.

  curl -u "elasticclouduser:elasticclouduserPassword" -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io:443/_cat/indices?v"

Command to delete specfic indices.

  curl -u "elastic:password" -X DELETE "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io:443/world-learning"

Verify that your Elasticsearch cluster is up and running, and check its health status. You can do this by sending a request to the _cluster/health endpoint:

curl -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io/_cluster/health" -u elastic:password

