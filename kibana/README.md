https://www.elastic.co/guide/en/kibana/current/rpm.html

Cấu hình kibana.yml
---
server.port: port

server.host: "ip"

server.publicBaseUrl: "http://ip:port" (Nếu môi trường Production)

Tạo token từ elasticsearch và kibana code
---
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

/usr/share/kibana/bin/kibana-verification-code

Join cluster
---
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node

/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>
