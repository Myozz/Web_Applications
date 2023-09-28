# Reconnaissance (Trinh sát)
## Passive
- Passive Recon đơn giản là thu thập thông tin nhưng không động chạm gì đến mục tiêu (kiểu như port scan), mà sử dụng những dữ liệu đã được lưu sẵn
  - Một số công cụ hỗ trợ: ```whois```, ```shodan.io```, ```nslookup```, ```bugcrowd```,...
## Active
- Active Recon: Ngược lại với Passive, Active tận dụng các tool chuyên dụng để khai thác thông tin từ mục tiêu
  - Một số công cụ hỗ trợ: ```nmap``` (nmap là đủ :||)
## Phân loại tool scan
### Target Validation (Xác thực mục tiêu)
- Tool: ```whois```, ```nslookup```, ```dnsrecon```
- Lấy các thông tin công khai của mục tiêu (tên công ty, địa chỉ,...)
### Finding Subdomain (Tìm subdomain)
- Tool: ```google fu```, ```nmap```, ```sublist3r```,```crt.sh```, ```amass```,...
- Ví dụ về một cái sub domain của ```facebook.com``` là ```m.facebook.com```, đại khái là thế
- Trong các tool kể trên thì ta thấy chỉ có ```nmap``` là active :||
- ```sublist3r``` là một tool ngon trong phần này nhưng đ hiểu sao nó không hoạt động được trên máy tôi :|| mặc dù đã thử nhiều cách fix nhưng vẫn đ được. Cho nên tôi chuyển qua dùng ```amass``` hay ```crt.sh```:|||
### Fingerprinting (Lấy dấu vân tay)
- Chúng nó không thể nghĩ ra cái tên nào khác à :||
- Tool: ```nmap```, ```wappalyzer```, ```whatweb```, ```netcat```,..'
- Kiểm tra web mục tiêu được xây dựng bằng các phương thức nào, cái này dùng wappalyzer là hợp lý nhất
### Data Breaches
- Tool: ```HaveIBeenPwned```, ```similar lists```,...
- Kiểm tra các thông tin bị rò rỉ
  
# Scanning and Emumeration (Quét + Liệt kê)
- Là quá trình scan kiểm vuln :||
- Công cụ: ```nmap```, ```Burp Suite```, ```nikto```
## Emumerate cùng Búp Măng Sui (Burp Suite)
- Không hiểu sao từ lúc dùng giao diện cmd thì chuyển sang lại một tool dùng GUI nó lại khó lạ thường :))
### Lấy subdomain bằng Burp
- Mục ```target```, chọn ```Scope Settings```, Tick vào ```Use advanced scope control```
- Add domain của bạn vào dưới dạng sau: ```.*\.<đầu>\.<đuôi>$```
  - Ví dụ ```.*\.facebook\.com$```
- Sau đấy ta chỉ cần truy cập vào trang web đó trên trình duyệt, rồi filter các target được add scope là có danh sách subdomain rồi
# Gaining Access
- Như tên, lấy quyền truy cập :|| sử dụng lỗ hổng vừa tìm được

# Maintaining Access 
- Duy trì quyền truy cập mà bạn vừa kiếm được :||

# Covering Tracks
- Tiếp tục theo dõi mục tiêu :||
