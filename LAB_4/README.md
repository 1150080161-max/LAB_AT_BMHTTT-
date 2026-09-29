# LAB 4 - NMAP: NHẬN DIỆN MÁY CHỦ, CỔNG VÀ DỊCH VỤ

## 1. Thông tin sinh viên

-   **Họ và tên:** Phạm Nguyễn Anh Toàn
-   **MSSV:** 1150080161
-   **Tên lab:** Lab 4 - Nmap: Nhận diện máy chủ, cổng và dịch vụ

## 2. Phiên bản và môi trường thực hành

-   **Máy scanner:** Ubuntu Linux VM
-   **VMware Workstation:** VMware Workstation 26H1
-   **Máy đích:** Metasploitable 2
-   **Nmap:** 7.98
-   **Mạng thực hành:** VMware Host-only
-   **Subnet:** `192.168.108.0/24`
-   **Ubuntu scanner:** `192.168.108.129/24`
-   **Metasploitable 2:** `192.168.108.131/24`
-   **VMnet1:** Host-only, `192.168.108.0/24`, subnet mask
    `255.255.255.0`

> Lưu ý: Trong lab này, Ubuntu được sử dụng thay cho Kali Linux làm máy
> scanner. Các thao tác quét chỉ thực hiện trong mạng Host-only của môi
> trường lab.

## 3. Cách dựng môi trường

1.  Tạo và khởi động máy ảo Ubuntu trên VMware Workstation.
2.  Cấu hình Network Adapter của Ubuntu ở chế độ **Host-only**.
3.  Cấu hình Metasploitable 2 ở cùng mạng **Host-only**.
4.  Kiểm tra địa chỉ IP của Ubuntu và Metasploitable 2.
5.  Kiểm tra kết nối giữa Ubuntu và Metasploitable 2 bằng ICMP.
6.  Cài đặt và kiểm tra Nmap trên Ubuntu.
7.  Thực hiện host discovery để xác định các máy đang hoạt động trong
    mạng lab.
8.  Thực hiện các kỹ thuật TCP/UDP scan, service detection, OS detection
    và NSE.
9.  Xuất kết quả scan sang các định dạng phục vụ phân tích.

## 4. Kiểm tra kết nối và nhận diện host

### 4.1. Kiểm tra IP

Ubuntu sử dụng địa chỉ:

``` text
192.168.108.129/24
```

Metasploitable 2 sử dụng địa chỉ:

``` text
192.168.108.131/24
```

Hai máy cùng nằm trong mạng:

``` text
192.168.108.0/24
```

### 4.2. Kiểm tra kết nối

Ubuntu ping thành công đến Metasploitable 2:

``` text
4 packets transmitted
4 packets received
0% packet loss
```

**Kết quả:** PASS

### 4.3. Host discovery

Lệnh sử dụng:

``` bash
nmap -sn 192.168.108.0/24
```

Kết quả phát hiện các host đang hoạt động, trong đó có:

``` text
192.168.108.1
192.168.108.129
192.168.108.131
```

**Kết quả:** PASS

## 5. Các kỹ thuật quét đã thực hiện

### TH1 - TCP SYN Scan

Lệnh:

``` bash
sudo nmap -sS 192.168.108.131
```

Kết quả xác định nhiều TCP port đang mở trên Metasploitable 2, bao gồm:

``` text
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
5432/tcp open  postgresql
5900/tcp open  vnc
8009/tcp open  ajp13
8180/tcp open  http
```

**Kết quả:** PASS

### TH2 - TCP Connect Scan

Lệnh:

``` bash
nmap -sT 192.168.108.131
```

TCP Connect Scan được thực hiện để đối chiếu với SYN Scan.

**Kết quả:** PASS

### TH3 - UDP Scan

Lệnh:

``` bash
sudo nmap -sU --top-ports 20 192.168.108.131
```

Một số kết quả đáng chú ý:

``` text
53/udp   open           domain
67/udp   closed         dhcps
68/udp   open|filtered  dhcpc
69/udp   open|filtered  tftp
123/udp  open|filtered  ntp
137/udp  open           netbios-ns
```

Kết quả cho thấy UDP scan có thể xuất hiện trạng thái `open|filtered`.

**Kết quả:** PASS

### TH4 - FIN, NULL và Xmas Scan

#### FIN Scan

``` bash
sudo nmap -sF 192.168.108.131
```

Các port TCP được phát hiện chủ yếu ở trạng thái:

``` text
open|filtered
```

#### NULL Scan

``` bash
sudo nmap -sN 192.168.108.131
```

Kết quả cũng xuất hiện nhiều port ở trạng thái:

``` text
open|filtered
```

#### Xmas Scan

``` bash
sudo nmap -sX 192.168.108.131
```

Kết quả tiếp tục cho thấy nhiều port ở trạng thái:

``` text
open|filtered
```

**Kết quả:** PASS

### TH5 - ACK Scan

Lệnh:

``` bash
sudo nmap -sA 192.168.108.131
```

Kết quả:

``` text
1000 unfiltered tcp ports
```

ACK Scan được sử dụng để đánh giá trạng thái lọc của firewall, không
dùng để khẳng định port đang mở.

**Kết quả:** PASS

### TH6 - Service Version Detection

Lệnh:

``` bash
sudo nmap -sV 192.168.108.131
```

Một số dịch vụ và phiên bản được nhận diện:

``` text
21/tcp   vsftpd 2.3.4
22/tcp   OpenSSH 4.7p1 Debian 8ubuntu1
25/tcp   Postfix smtpd
53/tcp   ISC BIND 9.4.2
80/tcp   Apache httpd 2.2.8
139/tcp  Samba smbd 3.X - 4.X
445/tcp  Samba smbd 3.X - 4.X
3306/tcp MySQL 5.0.51a-3ubuntu5
5432/tcp PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp VNC protocol 3.3
8009/tcp Apache Jserv
8180/tcp Apache Tomcat/Coyote JSP engine 1.1
```

**Kết quả:** PASS

### TH7 - OS Detection

Lệnh:

``` bash
sudo nmap -O 192.168.108.131
```

Nmap nhận diện:

``` text
Device type: general purpose
Running: Linux 2.6.X
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
```

**Kết quả:** PASS

### TH8 - NSE SMB OS Discovery

Lệnh:

``` bash
sudo nmap --script smb-os-discovery -p445 192.168.108.131
```

Kết quả:

``` text
445/tcp open microsoft-ds

OS: Unix (Samba 3.0.20-Debian)
Computer name: metasploitable
Domain name: localdomain
FQDN: metasploitable.localdomain
```

**Kết quả:** PASS

### TH9 - Aggressive Scan

Lệnh:

``` bash
sudo nmap -A 192.168.108.131
```

Kết quả kết hợp service detection, OS detection, NSE và traceroute.

Nmap tiếp tục nhận diện:

``` text
Running: Linux 2.6.X
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
```

**Kết quả:** PASS

## 6. Xuất và lưu kết quả

### 6.1. Normal output

Lệnh:

``` bash
sudo nmap -sV 192.168.108.131 -oN scan_result.txt
```

File được tạo:

``` text
scan_result.txt
```

### 6.2. XML output

Lệnh:

``` bash
sudo nmap -sV 192.168.108.131 -oX scan_result.xml
```

File được tạo:

``` text
scan_result.xml
```

Kết quả kiểm tra:

``` text
scan_result.txt
scan_result.xml
```

### 6.3. HTML

Trong quá trình thực hành, `xsltproc` chưa được cài trên Ubuntu khi máy
đang sử dụng Host-only. Phần XML → HTML được **bỏ qua**, vì đã có kết
quả TXT và XML phục vụ lưu trữ/phân tích.

### 6.4. Grepable output

Lệnh:

``` bash
sudo nmap -sV 192.168.108.131 -oG smb.txt
```

Sau đó:

``` bash
grep "445/open" smb.txt
```

Kết quả xác nhận:

``` text
445/open/tcp/netbios-ssn/Samba smbd 3.X - 4.X
```

**Kết quả:** PASS

## 7. Evidence chính

Các ảnh chụp chính trong quá trình thực hành:

-   **Hình 5:** OS Detection (`-O`) trên Metasploitable 2.
-   **Hình 6:** NSE SMB OS Discovery.
-   **Hình 7:** Kết quả Nmap được lưu dưới dạng TXT và XML.
-   **Hình 9:** TCP Connect Scan (`-sT`).
-   **Hình 10:** UDP Scan.
-   **Hình 11:** FIN Scan (`-sF`).
-   **Hình 12:** NULL Scan (`-sN`).
-   **Hình 13:** Xmas Scan (`-sX`).
-   **Hình 14:** ACK Scan (`-sA`).
-   **Hình 15:** Aggressive Scan (`-A`).
-   **Hình 16:** Grepable output và lọc cổng `445/open`.

> Hình 8 HTML được bỏ qua theo quyết định thực hành do môi trường
> Host-only.

## 8. Câu hỏi phân tích

### 1. Sự khác nhau giữa open, closed và filtered là gì? Nêu một tình huống mỗi trạng thái.

**Trả lời:**

-   **Open:** Có ứng dụng/dịch vụ đang lắng nghe trên port và Nmap có
    thể xác định port đang mở. Ví dụ: `21/tcp open ftp`.
-   **Closed:** Port có thể truy cập được nhưng không có dịch vụ lắng
    nghe.
-   **Filtered:** Nmap không xác định được port mở hay đóng vì gói tin
    bị firewall hoặc cơ chế lọc chặn.

### 2. Tại sao -sS thường cần quyền cao hơn -sT? Hai kỹ thuật khác nhau ở cơ chế thiết lập kết nối như thế nào?

**Trả lời:**

`-sS` (SYN scan) gửi gói SYN và phân tích phản hồi mà không hoàn thành
kết nối TCP đầy đủ, nên thường cần quyền cao để tạo và xử lý gói tin
thô.

`-sT` (TCP Connect scan) sử dụng cơ chế kết nối TCP thông thường của hệ
điều hành và hoàn thành quá trình bắt tay TCP, vì vậy thường không cần
quyền cao như `-sS`.

### 3. Vì sao FIN/Xmas/NULL có thể cho kết quả khó diễn giải trên một số hệ điều hành hoặc firewall?

**Trả lời:**

Các scan FIN/Xmas/NULL dựa vào cách hệ điều hành hoặc firewall phản hồi
những gói TCP có cờ đặc biệt hoặc không có cờ thông thường. Cách xử lý
các gói này có thể khác nhau giữa các hệ điều hành và thiết bị mạng. Vì
vậy kết quả có thể xuất hiện dạng `open|filtered`, khó xác định chính
xác port đang mở hay bị lọc.

### 4. ACK scan trả lời câu hỏi gì khác với SYN scan?

**Trả lời:**

SYN scan chủ yếu được dùng để xác định trạng thái open/closed của port.

ACK scan chủ yếu giúp xác định port có bị lọc (filtered) hay không, tức
là đánh giá khả năng firewall đang lọc gói tin. ACK scan không dùng để
khẳng định một port đang open.

### 5. Tại sao UDP scan thường chậm và dễ xuất hiện open\|filtered?

**Trả lời:**

UDP không có cơ chế bắt tay như TCP nên Nmap thường phải dựa vào phản
hồi ICMP hoặc phản hồi từ chính dịch vụ để xác định trạng thái. Nhiều
UDP port không trả lời khi nhận gói tin, khiến Nmap khó phân biệt giữa
open và filtered. Vì vậy UDP scan thường chậm và có thể xuất hiện
`open|filtered`.

### 6. -sV đóng vai trò gì trong quản lý lỗ hổng? Tại sao chỉ biết port 80 là chưa đủ?

**Trả lời:**

`-sV` giúp xác định dịch vụ và phiên bản đang chạy trên port.

Chỉ biết `80/tcp open` cho biết có dịch vụ HTTP nhưng chưa biết đó là
phần mềm nào và phiên bản nào. Khi dùng `-sV`, ví dụ trên Metasploitable
2 xác định được Apache httpd 2.2.8. Thông tin phiên bản giúp đối chiếu
với các lỗ hổng và đánh giá rủi ro chính xác hơn.

### 7. OS fingerprinting có những giới hạn nào? Vì sao không nên coi kết quả -O là tuyệt đối?

**Trả lời:**

OS fingerprinting dựa trên đặc điểm phản hồi của hệ thống mạng nên kết
quả có thể bị ảnh hưởng bởi firewall, thiết bị trung gian, cấu hình mạng
hoặc việc các hệ điều hành có đặc điểm tương tự nhau.

Vì vậy kết quả `-O` là ước đoán dựa trên fingerprint, không phải bằng
chứng tuyệt đối về hệ điều hành.

Trong bài lab, Nmap nhận diện Linux 2.6.X và ước đoán Linux 2.6.9 -
2.6.33.

### 8. NSE script báo timeout có đồng nghĩa "không có lỗ hổng" không? Giải thích.

**Trả lời:**

Không.

Timeout chỉ cho biết script không nhận được phản hồi cần thiết trong
thời gian cho phép. Nguyên nhân có thể là firewall, dịch vụ không phản
hồi, mạng hoặc script không tương thích.

Do đó không thể kết luận rằng không có lỗ hổng chỉ dựa trên việc NSE
timeout.

### 9. So sánh before/after hardening: thay đổi nào trong port state chứng minh biện pháp phòng thủ có hiệu lực?

**Trả lời:**

Biện pháp phòng thủ có hiệu lực khi sau hardening, những port/dịch vụ
không cần thiết chuyển từ trạng thái `open` sang `closed` hoặc
`filtered`, đồng thời dịch vụ không cần thiết không còn được truy cập từ
nguồn quét.

Cần chạy cùng một lệnh scan trước và sau khi hardening để so sánh.

### 10. Nêu ba cấu hình phòng thủ giúp giảm bề mặt tấn công mà không dựa vào "ẩn mình" trước Nmap.

**Trả lời:**

1.  Tắt hoặc gỡ các dịch vụ không cần thiết, giảm số lượng port đang mở.
2.  Cấu hình firewall chỉ cho phép các kết nối cần thiết.
3.  Cập nhật và vá lỗi phần mềm/dịch vụ, đồng thời loại bỏ các dịch vụ
    hoặc phiên bản không còn cần thiết.

## 9. Lỗi gặp phải và cách khắc phục

### Lỗi 1 - `xsltproc: command not found`

Khi chuyển XML sang HTML, Ubuntu báo:

``` text
Command 'xsltproc' not found
```

**Cách xử lý:**

Không cài thêm gói khi Ubuntu đang được giữ ở mạng Host-only. Phần HTML
được bỏ qua; kết quả TXT và XML vẫn được lưu đầy đủ.

### Lỗi 2 - Cảnh báo Nmap khi xuất XML

Khi chạy:

``` bash
sudo nmap -sV 192.168.108.131 -oX scan_result.xml
```

xuất hiện:

``` text
NSOCK ERROR ... Bind to 0.0.0.0:631 failed
Address already in use
```

Tuy nhiên Nmap vẫn hoàn thành scan và tạo được `scan_result.xml`.

**Kết quả:** Không ảnh hưởng đến kết quả scan trong bài thực hành.

## 10. Kết luận

Qua LAB4, em đã sử dụng Ubuntu Linux làm máy scanner thay cho Kali Linux
để thực hiện nhận diện host, quét TCP/UDP, phát hiện dịch vụ và phiên
bản, nhận diện hệ điều hành, thực hiện các kỹ thuật FIN/NULL/Xmas/ACK và
sử dụng NSE trên Metasploitable 2.

Các kết quả cho thấy Metasploitable 2 có nhiều dịch vụ TCP/UDP đang hoạt
động, trong đó có FTP, SSH, Telnet, HTTP, SMB/Samba, MySQL, PostgreSQL,
VNC và các dịch vụ khác. Việc sử dụng `-sV`, `-O`, `-A` và NSE giúp bổ
sung thông tin từ kết quả quét cổng cơ bản.

Toàn bộ hoạt động quét được thực hiện trong mạng **VMware Host-only
`192.168.108.0/24`**, với Ubuntu `192.168.108.129` là máy scanner và
Metasploitable 2 `192.168.108.131` là máy đích. Không thực hiện quét hệ
thống bên ngoài phạm vi môi trường lab.
