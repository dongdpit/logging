Bước 1: Cấu hình Syslog server profile
---
Device => Server Profiles => Syslog

![image](https://github.com/user-attachments/assets/5faaf87a-bdb9-49a3-8333-1a81f31b7fc6)
![image](https://github.com/user-attachments/assets/facc6d72-00d6-43e5-a3c1-0b0a199c5d04)

 
Chú thích:

	Syslog Server: IP của ELK Server
 
	Transport: Giao thức khi cấu hình trên Filebeat - UDP/TCP hoặc SSL
 
	Port: Cổng khi cấu hình pipeline trên Logstash
 
	Format: mặc định là BSD cho giao thức UDP hoặc IETF cho TCP/SSL
 
Bước 2: Cấu hình Syslog forwarding cho các log Traffic, Threat…
---
![image](https://github.com/user-attachments/assets/ce3f7359-55e8-4986-a129-b207025d8219)
![image](https://github.com/user-attachments/assets/2d297945-3355-4cc9-9d00-412e1403f897)

Tạo Forwarding Profile

![image](https://github.com/user-attachments/assets/5af3bcb2-13e7-4d79-abaa-d9e2ea700fad)

Làm tương tự với các Forwarding Profile khác

![image](https://github.com/user-attachments/assets/edbb3c26-4812-4a46-aa18-a4b45536a28d)
![image](https://github.com/user-attachments/assets/36caffed-1740-4905-bdeb-31c0d0e9d4bc)
![image](https://github.com/user-attachments/assets/1f975f22-a925-4fc5-a9d7-d518521ce672)

Kiểm tra lại và lưu

![image](https://github.com/user-attachments/assets/65533a63-c016-4f2a-965c-fbf79418dbf0)

Bước 3: Cấu hình Syslog forwarding cho System, Configuration…
---
Device => Log Settings

![image](https://github.com/user-attachments/assets/a1335915-1157-4a24-9d4a-31d429a99a3e)

Tạo Syslog System

![image](https://github.com/user-attachments/assets/f1a5a016-710c-4bd4-8657-962e05fbc4f1)
 
Làm tương tự với các Profile khác

![image](https://github.com/user-attachments/assets/7624a971-ccb9-499c-8b87-0bdb94873830)
![image](https://github.com/user-attachments/assets/19da1758-e753-4808-b209-f81041bbd384)

Bước 4: Cấu hình Log Forwarding cho Security Policy
---
Policies => Security

![image](https://github.com/user-attachments/assets/f99102c0-3a58-48ba-a478-8dd71da51ab5)
![image](https://github.com/user-attachments/assets/d1720a4f-4bcd-4767-87c9-707c55f9f4e0)

Cuối cùng, Commit để lưu tất cả cấu hình

![image](https://github.com/user-attachments/assets/fee4bb2b-6a1d-4f23-8032-8f3e35fc65eb)

Link tham khảo: https://docs.paloaltonetworks.com/pan-os/9-1/pan-os-admin/monitoring/use-syslog-for-monitoring/configure-syslog-monitoring
