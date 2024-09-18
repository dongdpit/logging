https://www.elastic.co/guide/en/elasticsearch/reference/8.15/rpm.html

Cấu hình jvm.options
---
-Dhttp.proxyHost=ip

-Dhttp.proxyPort=port

-Dhttps.proxyHost=ip

-Dhttps.proxyPort=port

Cấu hình elasticsearch.yml
---
cluster.name: elk-cluster

node.name: node-n

network.host: IP

discovery.seed_hosts: ["ip:port-host1", "ip:port-host2"]

cluster.initial_master_nodes: ["ip-node-1", "ip-node-2"]

xpack.ml.enabled: false (Nếu máy chủ đời cũ)

Đổi password elastic
---
/usr/share/elasticsearch/bin/elasticsearch-reset-password -i -u elastic

Tạo token trên node chính
---
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node

Chạy trên các node phụ
---
/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token token
