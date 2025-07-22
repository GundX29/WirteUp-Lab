# BÁO CÁO QUÁ KẾT QUẢ THỰC HIỆN RECONNAISSANCE TARGET thschool.edu.vn

---

🔍 Recon (Reconnaissance) là gì?
Recon (viết tắt của reconnaissance, tức "trinh sát") là giai đoạn đầu tiên trong quá trình tấn công an ninh mạng hoặc pentest. Mục tiêu là thu thập thông tin càng nhiều càng tốt về mục tiêu, bao gồm:

Tên miền

IP, Subnet

Dịch vụ đang chạy

Email nhân sự

Cấu trúc hệ thống

Các subdomain, directory

Công nghệ đang dùng (CMS, server type…)

Lỗ hổng tiềm ẩn

🧭 Các loại Recon
1. Passive Recon (Trinh sát thụ động)
Không tương tác trực tiếp với hệ thống mục tiêu.

Sử dụng nguồn công khai (OSINT): Google, WHOIS, Shodan, Wayback Machine…

An toàn, khó bị phát hiện.

2. Active Recon (Trinh sát chủ động)
Tương tác trực tiếp với hệ thống mục tiêu.

Dùng các tool như Nmap, Gobuster, Dirb, Nikto, Wappalyzer…


Target trong lần này là web của TH School (*https://thschool.edu.vn*)

<img width="1912" height="873" alt="Image" src="https://github.com/user-attachments/assets/cc4b7ff1-7c44-4f3d-971c-db03654fc309" />

Ta sẽ tuân thủ và đặt các mục tiêu theo workflow sau:

<img width="582" height="296" alt="Image" src="https://github.com/user-attachments/assets/50297e63-a342-4cb0-a38e-7ee300cc6145" />

## 1. Thu thập domain, subdomain

Ta sử dụng công cụ ***Amass*** để tìm ra các subdomain của mục tiêu
Thực hiện tra cứu subdomain của tên miền thschool.edu.vn:

``` bash
amass enum -passive -d thschool.edu.vn
```
<img width="534" height="246" alt="Image" src="https://github.com/user-attachments/assets/ec0b1d87-d49a-4b3c-bfff-1f4111e8aa8f" />

&rarr;  Đây là các subdomain mà ta vừa thu thập được.

Ngoài ra ta có thể sử dụng các web site online như crt.sh(*https://crt.sh/*) hay dnsdumpster(*https://dnsdumpster.com/*) để có thể thu thập các subdomain
<img width="1448" height="939" alt="Image" src="https://github.com/user-attachments/assets/aa7b82c6-38e0-4492-95bb-32a0ee8565f4" />

<img width="1341" height="932" alt="Image" src="https://github.com/user-attachments/assets/c6aac5db-67fc-48b2-bab7-465952ebe139" />


## 2. Thu thập các thông tin về địa chỉ IP
Ta sử dụng công cụ Nslookup để tra cứu thông tin DNS của tên miền thschool.edu.vn

Thực hiện tra cứu IP của tên miền thschool.edu.vn:

``` bash
nslookup thschool.edu.vn
```
<img width="445" height="157" alt="Image" src="https://github.com/user-attachments/assets/28c21ded-799a-4dc7-9e50-680d6ffe8820" />

&rarr; Địa chỉ IP mà ta tìm được chính là địa chỉ chứa trang web mục tiêu.

Tra cứu các máy chủ DNS của mục tiêu:

```bash
nslookup -type=ns thschool.edu.vn
```

<img width="482" height="330" alt="Image" src="https://github.com/user-attachments/assets/12551bb5-83c5-4834-b5c9-d8fc43e315f5" />

&rarr; Ta tìm được các máy chủ DNS của domain thschool.edu.vn.

Xác định máy chủ email (mail server):
```bash
nslookup -type=mx thschool.edu.vn
```

<img width="513" height="239" alt="Image" src="https://github.com/user-attachments/assets/a9df26ff-18eb-4de3-a8e2-be95adee33e1" />

&rarr; Ta tìm được các mail server chịu trách nhiệm tương ứng với từng subdomain


## 3. Qét các công dịch vụ

Ta sử dụng ***nmap*** 1 công cụ nổi tiếng và rất phô biến để thực hiện công việc:

Quét các cổng hiện đang chạy của mục tiêu:
```bash
nmap -p-65535 thschool.edu.vn
```

<img width="629" height="195" alt="Image" src="https://github.com/user-attachments/assets/53603a13-5a7e-4c3b-8cbe-d6ad09b1b4a9" />

Quét dịch vụ trên các cổng đang mở:

```bash
nmap -sV thschool.edu.vn
```

<img width="629" height="160" alt="Image" src="https://github.com/user-attachments/assets/1a8ba0dc-330c-40ee-85d8-cff2c0407363" />

&rarr; Kết quả cho thấy trên cổng 80 là phiên bản: Apache/2 (máy chủ web đang chạy Apache)

***P/s: Mình quét được 1 lần thì bị chặn sau đó không còn trả về được kết qua từ Nmap nữa***

<img width="628" height="209" alt="Image" src="https://github.com/user-attachments/assets/5220c135-cf1c-4c60-bcd5-59f73a8d1f64" />

Quét xác định hệ diều hành (OS) của mục tiêu :

```bash
nmap -O thschool.edu.vn
```

<img width="627" height="401" alt="Image" src="https://github.com/user-attachments/assets/a471fedc-764c-4b81-a709-d1b09195f6df" />

&rarr; Nmap dự đoán có thể là một trong các OS sau:

<ul>
<li>Linux 3.2 / Linux 4.4

<li>Windows XP SP3 / Windows 7 / Server 2012

<li>DD-WRT (firmware thiết bị mạng)

<li>VMware NAT Device (nếu đang ảo hóa)
</ul>

## 4. Techlonogy Fingerprinting

Tại bước này mục tiêu của chúng ta là tìm được và phân tích những công nghệ, framework được sử dụng trên mục tiêu.

Ta sẽ sử dụng hai công cụ xác khá phổ biến trên linux và cả extention trên website: Whatweb, Wappalyzer Plugin.

```bash
whatweb https://thschool.edu.vn
```

<img width="688" height="189" alt="Image" src="https://github.com/user-attachments/assets/1a9e8a0c-b9f4-4499-b96e-4d4094499c26" />

Ta cũng có thê áp dụng quét tương tự cho subdomains khác của thschool.edu.vn mà ta vừa tìm  dược bên trên:

```bash
whatweb https://hoalac.thschool.edu.vn
```

<img width="1704" height="119" alt="Image" src="https://github.com/user-attachments/assets/16bd1336-b4b1-4592-a123-369a039d8b7a" />

Sử dụng extension ***Wappalyzier*** để kiểm tra trên website: *https://thschool.edu.vn*

<img width="1856" height="587" alt="Image" src="https://github.com/user-attachments/assets/b42ef59d-cd1b-4e78-9104-093f8c4853e4" />

Thực hiện tương tự với các subdomain khác của mục tiêu

<img width="1920" height="589" alt="Image" src="https://github.com/user-attachments/assets/7a26c7bf-7682-4fcd-84f8-f02940317c66" />


<img width="1920" height="582" alt="image" src="https://github.com/user-attachments/assets/c003e1d0-ab22-4719-af99-39c583740fcd" />


## 5. Directory Scan
👉 Tìm các thư mục và tệp tin ẩn hoặc không được bảo vệ trên một máy chủ web.

🎯 Mục đích của Directory Scan:
Xác định các điểm vào tiềm năng (entry points) như /admin, /login, /config, v.v.

Tìm ra file backup, file cấu hình, hoặc script ẩn mà không nên để lộ.

Hỗ trợ tấn công như LFI, RCE, File Upload, v.v.

📌 Công cụ phổ biến để scan:
<ul>
<li>ffuf

<li>dirb

<li>dirbuster

<li>gobuster
</ul>

Hầu hết các công cụ đề sử dụng 1 công thức chung để scan đường dẫn đó là brute force và ta sẽ sử dụng wordlist chung cho các công cụ là: */usr/share/wordlists/dirb/common.txt* hoặc */usr/share/wordlists/dirb/big.txt*

1. công cụ ***Gobuster*** ( được viết bằng Go, một công cụ quét thư mục và tệp tin ẩn trên web server, hoạt động bằng cách gửi hàng loạt HTTP request đến server web, dựa trên wordlist):

```bash
gobuster dir -u https://thschool.edu.vn/ -w /usr/share/wordlists/dirb/common.txt -t 50 
```

Kết quả thực hiện:

<img width="894" height="841" alt="image" src="https://github.com/user-attachments/assets/ab6b90fb-ec75-4028-9c3e-ba68469247fd" />

&rarrr; có thể thấy một số File nhạy cảm như: *.htaccess, .hta, .htpasswd* bị chặn truy cập (403), tuy nhiên có thể thấy một số đường dẫn thư mục tiềm năng như: /admin, /index.php, /robots.txt, ...

Sử dụng công cụ khác là ***ffuf*** (một công cụ fuzzing phổ biến cũng có thể dùng Directory/file):

```bash
ffuf -u https://thschool.edu.vn/FUZZ -w /usr/share/wordlists/dirb/big.txt -t 100 -s
```

Kết quả thực hiện:

<img width="806" height="839" alt="image" src="https://github.com/user-attachments/assets/b138ba47-2d4a-4f19-a5ea-e1325b276163" />

<img width="555" height="674" alt="image" src="https://github.com/user-attachments/assets/3b1e0252-88a1-47d6-a440-789628a955b3" />


## 6. Parameter Discovery

Bước này tìm các tham số (parameters) ẩn hoặc không được tài liệu hóa mà ứng dụng web sử dụng – đặc biệt là trong URL, form hoặc request body.

Nhiều lỗ hổng web như:
<ul>
<li>IDOR (Insecure Direct Object Reference)

<li>XSS (Cross-Site Scripting)

<li>SQLi (SQL Injection)

<li>Privilege Escalation
</ul>
… chỉ có thể khai thác nếu bạn biết đúng parameter để gửi dữ liệu vào
Thực hiện sử dụng công cụ ***Arjun*** (một công cụ mã nguồn mở bằng ngôn ngữ python, dùng để tự động tìm ra các tham số (parameter) ẩn mà các ứng dụng web có thể xử lý, hay là tìm tham số HTTP GET/POST tiềm ẩn).


- Quét URL chính: *https://thschool.edu.vn*

```bash
arjun -u https://thschool.edu.vn
```

<img width="1420" height="225" alt="image" src="https://github.com/user-attachments/assets/76136427-0d7d-422a-ac77-7047fe42af45" />

&rarr; Phát hiện được các tham số đầu vào mà mục tiêu xử lý

- Tương tự quét trên 1 vài các subdomain mà ta vừa thu thập được
    
  - *https://admin.photos.thschool.edu.vn/login*
 
    ```bash
    arjun -u https://admin.photos.thschool.edu.vn/login
    ```
 
     <img width="809" height="257" alt="image" src="https://github.com/user-attachments/assets/daadcc75-195e-4988-9dcf-a01e49b39184" />


    &rarr; Ở đây có 1 tham số khá đặc biêt "based on: body length", có nghĩa là nội dung trả về sẽ có độ dài ngắn khác nhau từ đó ta có thể suy ra được tham số đầu vào này sẽ có sự ảnh hưởng
 
  - *https://vinh.thschool.edu.vn*
 
    ```bash
    arjun -u https://vinh.thschool.edu.vn
    ```

    <img width="1713" height="281" alt="image" src="https://github.com/user-attachments/assets/f396222d-2b10-41aa-8f75-02d461a2517b" />


---

## Kết luận:
- Ta đã thực hiện được cơ bản các bước recon mục tiêu 


---
