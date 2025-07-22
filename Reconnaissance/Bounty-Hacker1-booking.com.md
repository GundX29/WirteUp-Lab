
# Qúa trình Recon mục tiêu trang web booking.com

# 1. Thu thập thông tin domain và subdomain
Công cụ amass khá nổi tiếng:
```
amass enum -passive -d booking.com
```
<img width="898" height="776" alt="image" src="https://github.com/user-attachments/assets/8084ed64-f422-42d3-9c2f-ae912d0279d1" />

&rarr; Ở đây ta tìm được khá nhiều các subdomain khác nhau tuy nhiên khi thực hiện pentest lưu ý chỉ thực hiện các subdomain được cho phép 

<img width="1183" height="761" alt="image" src="https://github.com/user-attachments/assets/4d99a468-f112-46c9-92cf-00e46028bb2a" />

Tương tự công cụ với công cụ subfinder

```
subfinder -d booking.com -o subfinder_booking.txt
```
<img width="882" height="751" alt="image" src="https://github.com/user-attachments/assets/f7877f0c-f246-4cf2-b54b-e1c7604c0626" />

&rarr; Kết quả trả về có vẻ nhiều hơn so với amass

- Hoặc ta có thể tìm kiếm trên các công cụ online khác như crt.sh(là một dịch vụ truy vấn Certificate Transparency logs, dùng để tìm kiếm thông tin các chứng chỉ số SSL/TLS được cấp cho các domain) hay shodan, virustotal..

- Sau khi tìm được danh sách domain và subdomain ta tiếp tục lọc ra xem những domain nào còn sống. Ở đây ta dùng công cụ httpx
```
httpx -l subfinder_booking.txt -o live.txt
```

<img width="526" height="833" alt="image" src="https://github.com/user-attachments/assets/db1dd6b4-f3e2-43df-857a-509f5ebb5450" />

# 2. Thu thập thông tin về địa chỉ IP của mục tiêu

Ta sử dụng nslookup

```
nslookup booking.com
```
<img width="324" height="259" alt="image" src="https://github.com/user-attachments/assets/19982016-ae99-425b-9c08-53cc25a14a85" />

&rarr; Ta tìm được các địa chỉ IP của mục tiêu.Đây là các IP thuộc cụm Amazon AWS (CloudFront CDN) → tức là booking.com đang dùng Content Delivery Network để phân phối nội dung nhanh trên toàn cầu.

```
nslookup -type=ns booking.com
```
<img width="518" height="210" alt="image" src="https://github.com/user-attachments/assets/a8b889e9-ed34-4cf9-b595-023a6eed8f64" />


&rarr;  Mục tiêu đang sử dụng hệ thống DNS của Amazon Route 53

```
nslookup -type=mx booking.com
```
<img width="595" height="179" alt="image" src="https://github.com/user-attachments/assets/1e9047fd-a0ce-4b22-ab7e-e4578d9f4bf3" />

&rarr;  Các mail server của mục tiêu

# 3. Thu thập thông tin về các cổng mở và dịch vụ chạy trên các cổng đó

```
nmap -sV -sC -T4 -Pn -p- 192.168.95.2
```
<img width="799" height="253" alt="image" src="https://github.com/user-attachments/assets/0987e126-f88a-4122-aa5d-6201d7e16fe9" />
 
<li>Port 53 mở: dịch vụ DNS (dnsmasq 2.80)
<li>dnsmasq: thường được dùng để cache DNS, cấp phát DHCP trong mạng LAN

- Thực hiện tương tự với các subdomain mà ta tìm được

# 4. Xác định hệ điều hành của mục tiêu

```
nmap -O booking.com
```

<img width="901" height="280" alt="image" src="https://github.com/user-attachments/assets/11abcfa1-39fb-4579-80e2-a575cf70a269" />

&rarr;Ở đây bị chặn mất khi ta quét nmap và CloudFront không cho ta thấy OS thực sự. Nó dùng các biện pháp chống scan (rate-limit, firewall, throttling).

Ta có thể dùng các extention được tích hợp như wappalyzer

<img width="1487" height="491" alt="image" src="https://github.com/user-attachments/assets/7edea936-13b3-428f-ae8e-5d4ea1ddea18" />

# 5. Techlonogy Fingerprinting


