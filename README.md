# logging-with-fluentd

## Step to pull and install the helm chart

      helm repo add fluent https://fluent.github.io/helm-charts
      helm repo update
      helm pull fluent/fluentd --untar  # with this you will get untar fluentd chart including all files(template , values.yaml)
      helm install fluent fluent/fluentd
      helm upgrade fluentd ./fluentd    ----> after making any changes in chart, you can use helm upgrade for reflecting the effect on running fluentd pods.. here ./fluentd is fluentd chart name in current directory.
     


## Troubleshooting
## Command to list indices.

    curl -u "elasticclouduser:elasticclouduserPassword" -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io:443/_cat/indices?v"

## Command to delete specfic indices.

    curl -u "elastic:password" -X DELETE "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io:443/world-learning"

## Verify that your Elasticsearch cluster is up and running, and check its health status. You can do this by sending a request to the _cluster/health endpoint:

    curl -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io/_cluster/health" -u elastic:password

