# Khái liệm :||
- Một vulnerability (lỗ hổng) được định nghĩa là một điểm yếu (đại khai thế) trong thiết kế, quá trình thực hiện hay hành vi của hệ thống hay ứng dụng
- Một attacker có thể khai thác chúng để truy cập vào những thông tin hoặc làm vài trò mèo khác mà không cần được uỷ quyền.
- Lỗ hổng có thể bắt nguồn từ nhiều yếu tố, bao gồm thiết kế ngu, hoặc cũng có thể là người dùng ngu để bị theo dõi (đại khái thế)
- Bây giờ đến với các loại lỗ hổng
## Operating System
- Loại lỗ hổng này được tìm thấy bên trong HĐH và thường dẫn đến việc Priv Esc (tấn công leo thang ngã vỡ mồm)
## (Mis)Configuration-based
- Loại này thì xuất phát từ việc app hay serv được config ngu. Ví dụ, một website để lộ thông tin khách hàng :|| chắc kiểu như để tên khách hàng trong comment của source
## Weak or Default Credentials
- App hay Serv có đi kèm với xác thực sẽ thường set ở default khi mới được thiết lập, và mấy thằng đần cứ thế mà dùng chứ không thềm sửa. Ví dụ, một admin panel bắt login nhưng thằng admin vẫn set user với pass mặc định là admin - admin, ngu như chó
## Application Logic
- Lỗ hổng kiểu này là kết quả của việc thiết kế app kinh phí thấp. Ví dụ, cơ chế xác thực được triển khai ngu khiến attaker có thể mạo danh người dùng để truy cập bằng một vài hình thức kiểu như dùng cookie mạo danh một user nào đấy để bypass login panel (gần dùng 2 cái trước)
## Human-Factor
- Là lỗ hổng lợi dụng những thói quen của người dùng. Ví dụ, gửi email lừa đảo được thiết kế legit để lừa mấy con chó click vào, thế là đi

# Hệ thống chấm điểm lỗ hổng :|| nghe đần vc (CVSS & VPR)
- Nghe có vẻ không cần thiết nhưng thôi biết càng tốt :||
## Common Vuln Scroring Sys (CVSS)
- Một framework phổ biến để chấm điểm vuln. Điểm số được xác định bởi nhiều yếu tố, dưới đây chỉ là một số trong chúng:
  - Độ dễ khai thác
  - Có thể khai thác không?
  - Vuln này can thiệp vào bộ 3 CIA như này (Confidentially, Integrity, Availability)
- Phân loại điểm

![image](https://github.com/Myozz/Web_Applications/assets/94811005/c50db261-31c5-460a-a9d3-c281b88ccc13)
### Lợi ích của CVSS
- Có thời gian hđ lâu dài
- Phổ biến
- Free framwỏk
### Bất lượi
- CVSS không được thiết kế để hỗ trợ ưu tiên các lỗ hổng, thay vào đó chỉ cho thấy tính nghiêm trọng của lỗ hổng
- CVSS đánh giá rất kỹ các lỗ hổng có thể khai thác. Tuy nhiên chỉ 20% trong tất cả có sẵn cách khai thác
- CVSS hiếm khi cập nhật điểm số mặc dù thực tế nó nhiều cách khai thác mới

## Vuln Priority Rating (VPR)
- VPR framwork là một framework hiện đại hơn trong việc quản lý vuln. Framwork này được coi là risk-driven, nghĩa là vuln đươcj cho điểm dựa chủ yếu vào ảnh hưởng tới tổ chức của nó thay vì các yếu tố tác động khác như trên CVSS
- VPR dùng hệ thống phân loại điểm như CVSS. Tuy nhiên có 2 điểm khác biệt đáng chú ý là VPR không có mục "None/Informational" và bởi vì VPR sử dụng phương thức tính điểm khác, nên là sẽ khác nhiều CVSS

![image](https://github.com/Myozz/Web_Applications/assets/94811005/64ca9bd3-5626-4ff2-bb62-a8ea9ea94ea6)
### Lợi ích của VPR
- Là một framwork hiện đại :||
- VPR xem xét hơn 150 yếu tố khi đánh giá
- Điểm được cập nhật
### Bất lợi
- VPR không open-source
- Chỉ có thể áp dụng bên ngoài nên tảng thương mai
- VPR không được xem xét bộ 3 CIA như CVSS, có nghĩa rủi ro đối với tính bảo mật, toàn vẹn và tính sẵn có của dữ liệu không đóng vai trờ lớn trong việc chấm điểm các vuln trong VPR

# Vuln Database
- Rút kinh nghiện từ phần trước :|| viết xong cảm giác chả có tác dụng gì nên phân này nói qua thôi
- Bổ sung tí khái niệm: PoC (Proof of Concept): một công nghệ hay tool chứng minh tính hiệu quả khai thác của một lỗ hổng
## NVD (National Vuln Dtb) 
- Web này liệt kê tất cả lỗ hỗng được phân loại bằng các CVE (Common Vuln and Exposure)
- Những CVE này có format là ```CVE-YEAR-IDNUMBER```. Ví dụ, lỗ hổng mà mailware Wannacry sử dụng là CVE-2017-0144
- NVD cho phép xem tất cả ccs CVE được xác nhận, sử dụng filter để phân loại.
- Mặc dù web này luôn cập nhật lỗ hổng mới những nó không ngon khi cần tìm lỗ hổng cho app hay hoàn cảnh cụ thể
## Exploit-DB
- Ghi lại các lỗ hổng cho software hay app dưới dạng tên, tác giả và phiên bản của software hay app
- Có thể dùng để tìm đoạn trích của code (PoC) được sử dụng để khai thác một lỗ hổng cụ thể
