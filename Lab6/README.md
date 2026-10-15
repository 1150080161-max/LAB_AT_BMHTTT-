# LAB 6 - XÁC THỰC ACTIVE DIRECTORY VÀ CHÍNH SÁCH TRUY CẬP TRÊN pfSense

## 1. Thông tin sinh viên

- **Họ và tên:** Phạm Nguyễn Anh Toàn
- **MSSV:** 1150080161
- **Tên lab:** Lab 6 - Xác thực Active Directory và chính sách truy cập trên pfSense

## 2. Phiên bản và môi trường thực hành

- **Nền tảng ảo hóa:** VMware Workstation
- **Firewall:** pfSense Community Edition
- **Domain Controller:** Windows Server 2022
- **Client-A:** Windows 11
- **Client-B:** Windows 11
- **DMZ-WEB:** Ubuntu Linux
- **Web Server:** Nginx
- **Domain:** `lab6.test`

### Địa chỉ IP sử dụng

| Thiết bị / Interface | Địa chỉ IP | Vai trò |
|---|---|---|
| pfSense WAN | DHCP `192.168.250.0/24` | Kết nối Internet |
| pfSense MGMT | `192.168.56.254/24` | Truy cập WebGUI |
| pfSense LAN | `10.6.0.1/24` | Gateway LAN |
| pfSense DMZ | `172.16.6.1/24` | Gateway DMZ |
| DC01 | `10.6.0.10/24` | Active Directory + DNS |
| Client-A | `10.6.0.21/24` | Máy người dùng A |
| Client-B | `10.6.0.22/24` | Máy người dùng B |
| GENERAL_SITE | `172.16.6.10` | Website chung |
| JOB_SITE | `172.16.6.20` | Website việc làm |
| SOCIAL_SITE | `172.16.6.30` | Website mạng xã hội |
| RESTRICTED_SITE | `172.16.6.40` | Website hạn chế |

> LAB 6 sử dụng topology riêng gồm WAN, MGMT, LAN và DMZ để tránh ảnh hưởng các cấu hình của lab trước, đồng thời thuận tiện kiểm thử LDAP, Captive Portal, Firewall Alias, Schedule và các chính sách truy cập.

## 3. Cách dựng môi trường

1. Tạo máy ảo pfSense trên VMware.
2. Gắn 4 card mạng cho pfSense:
   - WAN: NAT
   - MGMT: VMnet1
   - LAN: `lab6-lan`
   - DMZ: `lab6-dmz`
3. Cấu hình địa chỉ IP cho WAN, MGMT, LAN và DMZ.
4. Cài Windows Server 2022 làm Domain Controller.
5. Cài Active Directory Domain Services và DNS.
6. Tạo domain:

```text
lab6.test
```

7. Cấu hình Client-A:

```text
IP: 10.6.0.21
Gateway: 10.6.0.1
DNS: 10.6.0.10
```

8. Cấu hình Client-B:

```text
IP: 10.6.0.22
Gateway: 10.6.0.1
DNS: 10.6.0.10
```

9. Cấu hình Ubuntu DMZ-WEB:

```text
IP: 172.16.6.10
Gateway: 172.16.6.1
```

10. Cài Nginx trên Ubuntu.
11. Bổ sung các IP website mô phỏng:

```bash
sudo ip addr add 172.16.6.20/24 dev ens33
sudo ip addr add 172.16.6.30/24 dev ens33
sudo ip addr add 172.16.6.40/24 dev ens33
```

12. Tích hợp pfSense với Active Directory bằng LDAP.
13. Cấu hình Captive Portal.
14. Tạo Alias và Schedule.
15. Tạo Firewall Rules theo các kịch bản của LAB 6.
16. Kiểm tra Firewall Log và State Table.
17. Xuất file cấu hình `config.xml`.

## 4. Tích hợp Active Directory với pfSense

### 4.1. Cấu hình LDAP

Các thông số chính:

```text
Protocol version: 3
Search Scope: Entire Subtree
Base DN: DC=lab6,DC=test
Authentication Container: OU=LAB6-Users,DC=lab6,DC=test
Bind DN: CN=svc_pfsense_ldap,CN=Users,DC=lab6,DC=test
Initial Template: Microsoft AD
User Naming Attribute: samAccountName
Group Naming Attribute: cn
Group Member Attribute: memberOf
```

### 4.2. Kiểm tra LDAP

Vào:

```text
Diagnostics -> Authentication
```

Kiểm tra lần lượt:

```text
sv_a
sv_b
```

Cả hai tài khoản xác thực thành công.

**Kết quả:** PASS

## 5. Captive Portal

### 5.1. Tạo Captive Portal Zone

Zone:

```text
LAB6_USERS
```

Cấu hình:

```text
Interface: LAN
Authentication Method: Authentication backend
Authentication Server: LAB6-AD
```

### 5.2. Kiểm thử

- Client-A đăng nhập bằng `sv_a`.
- Client-B đăng nhập bằng `sv_b`.

Sau khi xác thực thành công, client có thể truy cập theo các rule firewall đã cấu hình.

**Kết quả:** PASS

## 6. Alias

### Host Alias

| Alias | IP |
|---|---|
| `CLIENT_A` | `10.6.0.21` |
| `CLIENT_B` | `10.6.0.22` |
| `POLICY_CLIENTS` | `10.6.0.21`, `10.6.0.22` |
| `GENERAL_SITE` | `172.16.6.10` |
| `JOB_SITE` | `172.16.6.20` |
| `SOCIAL_SITE` | `172.16.6.30` |
| `RESTRICTED_SITE` | `172.16.6.40` |

### Port Alias

```text
WEB_PORTS = 80, 443
```

**Kết quả:** PASS

## 7. Schedule

### LUNCH_WINDOW

```text
Monday - Friday
11:30 - 13:00
```

### WORK_HOURS

```text
Monday - Friday
08:00 - 17:00
```

### TEST_NOW

Được tạo tạm để kiểm thử các rule Schedule ngay tại thời điểm thực hành.

Sau khi kiểm thử, các rule được trả về đúng Schedule yêu cầu.

**Kết quả:** PASS

## 8. Các kịch bản kiểm thử

### Kịch bản 1 - Client-A bị chặn JOB_SITE

Rule:

```text
Action: Block
Source: CLIENT_A
Destination: JOB_SITE
Protocol: TCP
Port: 80
Log: Enabled
```

Kiểm thử:

```text
Client-A -> GENERAL_SITE: Allow
Client-A -> JOB_SITE: Block
```

Firewall Log ghi nhận:

```text
10.6.0.21 -> 172.16.6.20:80
```

**Kết quả:** PASS

### Kịch bản 2 - Chặn RESTRICTED_SITE cho Client-A và Client-B

Rule:

```text
Action: Block
Source: POLICY_CLIENTS
Destination: RESTRICTED_SITE
Protocol: TCP
Port: 80
Log: Enabled
```

Kết quả:

```text
Client-A -> 172.16.6.40: Block
Client-B -> 172.16.6.40: Block
```

GENERAL_SITE vẫn truy cập được.

**Kết quả:** PASS

### Kịch bản 3 - Client-B chỉ được truy cập SOCIAL_SITE trong giờ nghỉ

Rule 1:

```text
Action: Pass
Source: CLIENT_B
Destination: SOCIAL_SITE
Port: 80
Schedule: LUNCH_WINDOW
```

Rule 2:

```text
Action: Block
Source: CLIENT_B
Destination: SOCIAL_SITE
Port: 80
Schedule: Always
```

Trong lúc kiểm thử, `TEST_NOW` được sử dụng tạm thời.

Kết quả:

```text
Trong Schedule: Allow
Ngoài Schedule: Block
```

Sau khi kiểm thử:

```text
Schedule được trả lại LUNCH_WINDOW
```

**Kết quả:** PASS

### Kịch bản 4 - Chỉ cho HTTP/HTTPS trong WORK_HOURS

Rule 1:

```text
Action: Pass
Source: POLICY_CLIENTS
Destination: Any
Destination Port: WEB_PORTS
Schedule: WORK_HOURS
```

Rule 2:

```text
Action: Block
Source: POLICY_CLIENTS
Destination: Any
Schedule: Always
```

#### Trong WORK_HOURS

HTTP:

```text
PASS
```

Ping:

```text
TIMEOUT
```

#### Ngoài WORK_HOURS

HTTP:

```text
TIMEOUT
```

Ping:

```text
TIMEOUT
```

Sau khi test, WORK_HOURS được trả lại:

```text
08:00 - 17:00
```

**Kết quả:** PASS

### Kịch bản 5 - Cô lập DMZ khỏi LAN

Rule trên DMZ:

```text
Action: Block
Source: DMZ subnets
Destination: LAN subnets
Protocol: Any
Log: Enabled
```

Kiểm thử từ DMZ-WEB:

```bash
ping -c 4 10.6.0.10
```

Kết quả:

```text
4 packets transmitted
0 received
100% packet loss
```

Trong khi đó:

```text
Client-A -> GENERAL_SITE 172.16.6.10
```

vẫn truy cập được Nginx.

Rule tạm:

```text
Temporary allow DMZ Internet
```

đã được xóa sau khi cài Nginx xong.

**Kết quả:** PASS

### Kịch bản 6 - So sánh Block và Reject

Nguồn và đích:

```text
CLIENT_A -> SOCIAL_SITE
TCP/80
```

#### Reject

Trình duyệt hiển thị:

```text
ERR_CONNECTION_REFUSED
```

Kết nối bị từ chối gần như ngay lập tức.

#### Block

Trình duyệt hiển thị:

```text
ERR_CONNECTION_TIMED_OUT
```

Client phải chờ lâu hơn mới nhận lỗi.

### Nhận xét

```text
Reject: trả phản hồi từ chối ngay.
Block: bỏ gói tin và để client chờ timeout.
```

Rule thử nghiệm Kịch bản 6 đã được xóa sau khi test.

**Kết quả:** PASS

### Kịch bản 7 - Firewall Log và State Table

#### Firewall Log

Filter:

```text
Source: 10.6.0.21
Destination: 172.16.6.30
```

Log:

```text
Block Client-A to SOCIAL_SITE
10.6.0.21 -> 172.16.6.30:80
```

#### State Table

Filter:

```text
10.6.0.21
```

Kết quả xuất hiện kết nối TCP ở trạng thái:

```text
ESTABLISHED:ESTABLISHED
```

**Kết quả:** PASS

## 9. Ruleset LAN cuối cùng

Thứ tự rule:

```text
1. Block CLIENT_A -> JOB_SITE TCP/80
2. Block POLICY_CLIENTS -> RESTRICTED_SITE TCP/80
3. Pass CLIENT_B -> SOCIAL_SITE TCP/80 + LUNCH_WINDOW
4. Block CLIENT_B -> SOCIAL_SITE TCP/80
5. Pass POLICY_CLIENTS -> GENERAL_SITE TCP/80
6. Pass POLICY_CLIENTS -> Any WEB_PORTS + WORK_HOURS
7. Block POLICY_CLIENTS -> Any
```

pfSense sử dụng nguyên tắc:

```text
First Match
```

Rule đầu tiên khớp với lưu lượng sẽ được áp dụng.

Vì vậy các rule Block cụ thể phải được đặt phía trên các rule Pass rộng.

## 10. Ruleset DMZ cuối cùng

Sau khi xóa rule tạm, DMZ chỉ còn:

```text
Block DMZ subnets -> LAN subnets
```

Mục đích là không cho DMZ chủ động tạo kết nối vào LAN.

## 11. Evidence chính

Các ảnh minh chứng chính:

- Cấu hình 4 Network Adapter của pfSense.
- Interface Assignments.
- Địa chỉ IP WAN, MGMT, LAN và DMZ.
- Kiểm tra kết nối WAN.
- IP của DC01.
- IP Client-A và Client-B.
- Nginx hoạt động trên DMZ-WEB.
- LDAP Authentication `sv_a`.
- LDAP Authentication `sv_b`.
- Captive Portal login.
- Danh sách Alias.
- Danh sách Schedule.
- Rule và test Kịch bản 1.
- Rule và test Kịch bản 2.
- Rule và test Kịch bản 3.
- Rule và test Kịch bản 4.
- Rule và test Kịch bản 5.
- Reject test.
- Block test.
- Firewall Log.
- State Table.
- Ruleset LAN cuối.
- Ruleset DMZ cuối.
- Backup Configuration.

## 12. Câu hỏi phân tích

### 1. Vì sao LAB 6 phải dùng topology riêng thay vì tiếp tục sử dụng trực tiếp topology LAB 5?

**Trả lời:**

LAB 6 sử dụng topology riêng gồm WAN, MGMT, LAN và DMZ để phục vụ việc kiểm thử LDAP, Captive Portal và các policy firewall mà không ảnh hưởng cấu hình của LAB 5.

### 2. LDAP được dùng cho chức năng nào trên pfSense? Firewall rule chuẩn có thể dùng username AD làm Source trực tiếp không?

**Trả lời:**

LDAP được dùng để pfSense xác thực các tài khoản Active Directory như `sv_a` và `sv_b`, đồng thời được dùng làm Authentication Server cho Captive Portal.

Firewall rule thông thường của pfSense không dùng trực tiếp username AD làm Source mà áp dụng theo IP, Alias, Protocol, Port và Schedule.

### 3. Vì sao Captive Portal được đặt trên LAN nhưng WebGUI quản trị lại tách sang MGMT?

**Trả lời:**

Captive Portal đặt trên LAN vì Client-A và Client-B là các máy cần xác thực. WebGUI được tách sang MGMT `192.168.56.254` để việc quản trị pfSense không bị ảnh hưởng bởi Captive Portal hoặc các rule thử nghiệm trên LAN.

### 4. Search Scope Entire Subtree có ý nghĩa gì khi pfSense truy vấn Active Directory?

**Trả lời:**

Entire Subtree cho phép pfSense tìm tài khoản trong toàn bộ cây thư mục nằm dưới Base DN hoặc Authentication Container, kể cả các OU con.

### 5. Tại sao nên dùng service account thay vì Domain Administrator làm LDAP bind?

**Trả lời:**

Service account chỉ cần quyền đọc cần thiết để tìm và xác thực người dùng. Domain Administrator có quyền quá lớn nên nếu thông tin đăng nhập bị lộ sẽ gây rủi ro cao hơn.

### 6. Trong Kịch bản 3, vì sao cần một rule Block SOCIAL_SITE nằm dưới rule Pass có Schedule?

**Trả lời:**

Rule Pass chỉ hoạt động trong `LUNCH_WINDOW`. Khi Schedule hết hiệu lực, pfSense tiếp tục kiểm tra rule bên dưới. Rule Block đảm bảo Client-B bị chặn SOCIAL_SITE ngoài giờ nghỉ.

### 7. Vì sao Client-A/B dùng DNS 10.6.0.10 nhưng rule Internet của client không cần cho DNS tới 10.6.0.10?

**Trả lời:**

Client-A, Client-B và DC01 đều nằm trong subnet `10.6.0.0/24`. Lưu lượng DNS giữa client và `10.6.0.10` đi trực tiếp trong LAN và không đi qua pfSense.

### 8. Giải thích first-match bằng một ví dụ cụ thể trong ruleset tổng hợp.

**Trả lời:**

pfSense xét rule từ trên xuống. Ví dụ Block `CLIENT_A -> JOB_SITE` phải nằm trên Pass rộng `POLICY_CLIENTS -> WEB_PORTS`. Nếu Pass đặt trước, Client-A có thể được cho phép trước khi pfSense xét đến Block JOB_SITE.

### 9. Vì sao sau khi đổi rule cần tạo kết nối mới hoặc Reset States?

**Trả lời:**

pfSense lưu trạng thái của các kết nối trong State Table. Kết nối cũ có thể tiếp tục được xử lý theo state đã tồn tại, vì vậy cần tạo phiên mới hoặc Reset States để kiểm thử đúng rule mới.

### 10. Mô hình JOB_SITE/SOCIAL_SITE/RESTRICTED_SITE trong DMZ khác Web Category Filtering thật như thế nào?

**Trả lời:**

Trong LAB 6, các site được mô phỏng bằng địa chỉ IP trong DMZ và policy được áp theo IP/Port. Web Category Filtering thật thường phân loại website dựa trên domain, URL hoặc cơ sở dữ liệu nội dung.

### 11. Block và Reject khác nhau ra sao về hành vi phía client?

**Trả lời:**

Block bỏ gói tin nên client phải chờ timeout. Reject gửi phản hồi từ chối nên client nhận lỗi nhanh hơn.

Trong bài:

```text
Reject -> ERR_CONNECTION_REFUSED
Block  -> ERR_CONNECTION_TIMED_OUT
```

### 12. Nếu một rule Block có log nhưng Firewall Log không xuất hiện bản ghi, cần kiểm tra theo thứ tự nào?

**Trả lời:**

Kiểm tra:

1. Rule đã bật Log chưa.
2. Tạo traffic mới.
3. Filter theo đúng Source và Destination.
4. Kiểm tra thứ tự rule.
5. Kiểm tra state cũ.
6. Kiểm tra interface.
7. Kiểm tra Alias.
8. Kiểm tra Protocol và Port.

## 13. Lỗi gặp phải và cách khắc phục

### Lỗi 1 - Ubuntu chưa có Nginx

Ban đầu Ubuntu chưa cài Nginx và DNS không hoạt động khi tải package.

**Cách xử lý:**

Tạm tạo rule cho DMZ truy cập Internet, cấu hình DNS và cài Nginx.

Sau khi cài xong, rule tạm đã được xóa.

### Lỗi 2 - Chưa có IP `.20`, `.30`, `.40`

Ban đầu DMZ-WEB chỉ có:

```text
172.16.6.10
```

Bổ sung:

```bash
sudo ip addr add 172.16.6.20/24 dev ens33
sudo ip addr add 172.16.6.30/24 dev ens33
sudo ip addr add 172.16.6.40/24 dev ens33
```

### Lỗi 3 - Rule không chạy đúng do thứ tự

Một số rule Pass rộng có thể làm các Block phía dưới không được xét.

**Cách xử lý:**

Sắp xếp lại rule theo nguyên tắc first-match, đưa các Block cụ thể lên trên.

### Lỗi 4 - Schedule không đúng thời điểm kiểm thử

WORK_HOURS và LUNCH_WINDOW không trùng thời gian đang thực hành.

**Cách xử lý:**

Dùng `TEST_NOW` hoặc tạm chỉnh Schedule để kiểm thử, sau đó trả về đúng thời gian yêu cầu.

### Lỗi 5 - Firewall Log chỉ hiện Default deny trên MGMT

**Cách xử lý:**

Tạo lại traffic trên Client-A rồi filter:

```text
Source: 10.6.0.21
Destination: 172.16.6.30
```

Sau đó log hiển thị đúng:

```text
Block Client-A to SOCIAL_SITE
```

### Lỗi 6 - Không thấy kết nối Nginx trong State Table

Kết nối HTTP kết thúc nhanh nên state có thể biến mất.

**Cách xử lý:**

Filter theo Client-A:

```text
10.6.0.21
```

State Table hiển thị kết nối TCP:

```text
ESTABLISHED:ESTABLISHED
```

## 14. Backup cấu hình

Vào:

```text
Diagnostics -> Backup & Restore
```

Thiết lập:

```text
Backup Area: All
Backup SSH Keys: Enabled
Encryption: Disabled
```

Sau đó chọn:

```text
Download configuration as XML
```

File `config.xml` được lưu lại để nộp cùng bài LAB 6.

## 15. Kết luận

Qua LAB 6, em đã dựng hoàn chỉnh mô hình mạng pfSense gồm WAN, MGMT, LAN và DMZ. pfSense được tích hợp với Active Directory bằng LDAP và sử dụng Captive Portal để xác thực tài khoản domain.

Các Alias và Schedule được sử dụng để xây dựng chính sách truy cập cho Client-A và Client-B. Các kịch bản gồm chặn JOB_SITE, chặn RESTRICTED_SITE, giới hạn SOCIAL_SITE theo giờ nghỉ, giới hạn HTTP/HTTPS trong WORK_HOURS, cô lập DMZ khỏi LAN, so sánh Block và Reject, kiểm tra Firewall Log và State Table đều đã được thực hiện.

Sau khi hoàn thành kiểm thử, các rule tạm được xóa, ruleset được sắp xếp lại theo nguyên tắc first-match và toàn bộ cấu hình pfSense được sao lưu dưới dạng `config.xml`.
