1. Cài đặt Filebeat
2. Cấu hình filebeat.yml
3. Kích hoạt và cấu hình module iis.yml
filebeat modules enable iis
4. Chạy Filebeat (hoặc Start trong service)
filebeat -c filebeat.yml -e
