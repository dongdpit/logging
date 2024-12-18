https://www.elastic.co/guide/en/elasticsearch/reference/8.15/rpm.html

Cấu hình jvm.options
---
-Dhttp.proxyHost=ip

-Dhttp.proxyPort=port

-Dhttps.proxyHost=ip

-Dhttps.proxyPort=port

Cấu hình elasticsearch.yml (node-1)
---
cluster.name: my-application

node.name: node-1

network.host: IP

http.port: 9200

cluster.initial_master_nodes: ["node.name"]

xpack.ml.enabled: false (Nếu máy chủ đời cũ)

xpack.fleet.registryProxyUrl: "http://ip:port" (Nếu mạng dùng qua proxy)

Đổi password elastic
---
/usr/share/elasticsearch/bin/elasticsearch-reset-password -i -u elastic

Check health Elastic
---
curl -k -u elastic:password https://ip:port/_cluster/health?pretty

Tạo token trên node chính
---
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node

Chạy trên các node phụ
---
/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token token

Cấu hình elasticsearch.yml (node-n)
---
cluster.name: my-application

node.name: node-n

network.host: IP

http.port: 9200

xpack.ml.enabled: false (Nếu máy chủ đời cũ)

xpack.fleet.registryProxyUrl: "http://ip:port" (Nếu mạng dùng qua proxy)

Check join node
---
curl -k -u elastic:password https://ip:port/_cat/nodes
