Truy cập SSH và chạy lệnh, sau đó restart syslog service:
---
esxcli network firewall ruleset set --ruleset-id=syslog -e=true

esxcli system syslog config set --loghost='tcp://ip:port'
