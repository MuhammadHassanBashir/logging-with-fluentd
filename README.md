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

## Query to check logs are being send to elastic cloud or not while index is already been sent there.

      curl -u "elastic:EWsTCrASie5n60gQcbMEArtY" -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io/world-learning-test/_search?pretty"

## Verify that your Elasticsearch cluster is up and running, and check its health status. You can do this by sending a request to the _cluster/health endpoint:

    curl -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io/_cluster/health" -u elastic:password

## view kibana mapping
     curl -u "elastic:EWsTCrASie5n60gQcbMEArtY" -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io/world-learning-test/_mapping"  

## facing error Rejection from elastic search.

      I was facing issue while using json format during **parsing** in fluentd config file <source-section: source.conf >. i have changed it to plain text by selecting type to none in <source-section: source.conf >.

          **<parse>
              @type none  # No specific format, treat logs as plain text
          </parse>** 

It successfully sent the logs. I found this command error details by using these flags in **output-section: output.conf** **@log_level debug and  log_es_400_reason true**. i was sending logs in json object format but elastic cloud was excepting string.. I confirm it my viewing mapping through GUI or using  this qury **curl -u "elastic:EWsTCrASie5n60gQcbMEArtY" -X GET "https://my-deployment-619a2a.es.us-central1.gcp.cloud.es.io/world-learning-test/_mapping" **

      fileConfigs:
        01_sources.conf: |-
          ## logs from podman
          <source>
            @type tail
            @id in_tail_container_logs
            @label @KUBERNETES
            path /var/log/containers/*.log
            pos_file /var/log/fluentd-containers.log.pos
            tag kubernetes.*
            read_from_head true
            <parse>
              @type none  # No specific format, treat logs as plain text
            </parse>
            emit_unmatched_lines true
          </source>
      
          # expose metrics in prometheus format
          <source>
            @type prometheus
            bind 0.0.0.0
            port 24231
            metrics_path /metrics
          </source>
      
          
      
        02_filters.conf: |-
          <label @KUBERNETES>
            <match kubernetes.var.log.containers.fluentd**>
              @type relabel
              @label @FLUENT_LOG
            </match>
      
            # <match kubernetes.var.log.containers.**_kube-system_**>
            #   @type null
            #   @id ignore_kube_system_logs
            # </match>
      
            <filter kubernetes.**>
              @type kubernetes_metadata
              @id filter_kube_metadata
              skip_labels true
              skip_container_metadata true
              skip_namespace_metadata true
              skip_master_url true
            </filter>
      
            <match **>
              @type relabel
              @label @DISPATCH
            </match>
          </label>
      
        03_dispatch.conf: |-
          <label @DISPATCH>
            <filter **>
              @type prometheus
              <metric>
                name fluentd_input_status_num_records_total
                type counter
                desc The total number of incoming records
                <labels>
                  tag ${tag}
                  hostname ${hostname}
                </labels>
              </metric>
            </filter>
      
            <match **>
              @type relabel
              @label @OUTPUT
            </match>
          </label>
      
        04_outputs.conf: |-
          <label @OUTPUT>
            <match **>
              @type elasticsearch
              @log_level debug
              log_es_400_reason true
              cloud_id "My-deployment:dXMtY2VudHJhbDEuZ2NwLmNsb3VkLmVzLmlvOjQ0MyQxYWJlNDQ4ZjBmN2I0NzUzYTFkZWMyYjhiMTVhOGUxNCRjYzk0N2NkYTRjNzM0ODFmYWRhZjNmOTE1OWU2Yzg4MQ=="
              cloud_auth "elastic:EWsTCrASie5n60gQcbMEArtY"
              Include_Tag_Key true
              Tag_Key tags
              tls On
              tls.verify Off 
              index_name "world-learning-test"
              Suppress_Type_Name On
              time_key @timestamp  # Set time_key to @timestamp
              time_format "%Y-%m-%dT%H:%M:%S.%NZ"
              <inject>
                time_key @timestamp
                time_type string
                time_format %Y-%m-%dT%H:%M:%S.%NZ
                tag_key fluentd_tag
              </inject>
            </match>
          </label>
      

## Facing issues on sending @timestamps from fluentd to elastic cloud.

      I only added this flag **include_timestamp true** under output file named 04_outputs.conf.. It will send timestamp to elastic cloud.

      Complete correct output file..

            04_outputs.conf: |-
          <label @OUTPUT>
            <match **>
              @type elasticsearch
              @log_level debug
              log_es_400_reason true
              cloud_id "My-deployment:dXMtY2VudHJhbDEuZ2NwLmNsb3VkLmVzLmlvOjQ0MyQxYWJlNDQ4ZjBmN2I0NzUzYTFkZWMyYjhiMTVhOGUxNCRjYzk0N2NkYTRjNzM0ODFmYWRhZjNmOTE1OWU2Yzg4MQ=="
              cloud_auth "elastic:EWsTCrASie5n60gQcbMEArtY"
              Include_Tag_Key true
              Tag_Key tags
              tls On
              tls.verify Off 
              index_name "world-learning-test"
              Suppress_Type_Name On
              time_key @timestamp  # Set time_key to @timestamp
              time_format "%Y-%m-%dT%H:%M:%S.%NZ"
              include_timestamp true
            </match>
          </label>

             
      
