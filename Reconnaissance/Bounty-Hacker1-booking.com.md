
# Qúa trình Recon mục tiêu trang web booking.com

# 1. Thu thập thông tin domain và subdomain
```
amass enum -passive -d booking.com
```
<img width="898" height="776" alt="image" src="https://github.com/user-attachments/assets/8084ed64-f422-42d3-9c2f-ae912d0279d1" />

&rarr; Ở đây ta tìm được khá nhiều các subdomain khác nhau tuy nhiên khi thực hiện pentest lưu ý chỉ thực hiện các subdomain được cho phép 

<img width="1183" height="761" alt="image" src="https://github.com/user-attachments/assets/4d99a468-f112-46c9-92cf-00e46028bb2a" />

Hoặc ta có thể tìm kiếm trên các công cụ online khác như crt.sh(là một dịch vụ truy vấn Certificate Transparency logs, dùng để tìm kiếm thông tin các chứng chỉ số SSL/TLS được cấp cho các domain)


