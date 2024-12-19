https://www.elastic.co/guide/en/kibana/current/rpm.html

Cấu hình kibana.yml
---
server.port: port

server.host: "ip"

server.publicBaseUrl: "http://ip:port" (Nếu môi trường Production)

elasticsearch.hosts: ["https://ip:port"]

xpack.fleet.registryProxyUrl: "http://ip:port" (Nếu mạng dùng qua proxy)

Tạo token từ elasticsearch và lấy kibana code
---
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

/usr/share/kibana/bin/kibana-verification-code
