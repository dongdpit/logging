https://www.elastic.co/guide/en/elasticsearch/reference/8.15/rpm.html

Cấu hình jvm.options
---
-Dhttp.proxyHost=ip

-Dhttp.proxyPort=port

-Dhttps.proxyHost=ip

-Dhttps.proxyPort=port

Cấu hình elasticsearch.yml
---
network.host: ip

xpack.ml.enabled: false (Nếu máy chủ đời cũ)

Đổi password elastic
---
/usr/share/elasticsearch/bin/elasticsearch-reset-password -i -u elastic
