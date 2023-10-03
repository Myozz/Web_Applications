# Access Control (Kiểm soát truy cập) là cl gì?
- Kiểm soát truy cập là việc áp dụng các ràng buộc về ai/cái gì được uỷ quyền để thực hiện các hành động hoặc truy cập tài nguyên. Trong web/app, kiểm soát truy cập phụ thuộc vào xác thực và quản lí phiên (session)
  - **Authentication** (xác thực): xác nhận rằng user là chính họ :|| nghe lú nhỉ, ý là account là của chính chủ ấy
  - **Session Management** (Quản lý phiên): Xác định HTTP request tiếp theo có phải đang được thực hiện bởi cùng một user không?
  - **Access Control** (Kiểm soát truy cập): Xác định xem liệu user có quyền để thực hiện hành động nào đó không?
- **Broken Access Control** (Kiểm soát truy cập ngố đét) khá phố biến và thường xuyên có mặt trong những lỗ hổng nghiêm trọng nhất. Thiết kế và Quản lý Access Control là một vấn đề phức tạp, cần các tổ chức cùng triển khai kỹ thuật. Các quyết định thiết kế Access Control phải được thực hiện bởi con người nên khả năng xảy ra sai sót là cao vcl

![image](https://github.com/Myozz/Web_Applications/assets/94811005/18205d4d-9bc1-49fa-9aca-a5c396f06417)

# Vertical Privilege Escalation (leo thang đặc quyền theo chiều dọc?)
- Nếu một user có thể chiếm quyền truy cập vào các tính năng mà vốn không được cho phép truy cập, đó là **Vertical Priv Esc**
- Ví dụ: Nếu là một non-admin user có thể chiếm quyền kiểm soát một admin page/panel nơi mà họ có thể xoá user, thì đó là **Vertical Priv Esc**
  
# Unprotected Functionality (tính năng không được bảo vệ)
- Về cơ bản, **vertical priv esc** phát sinh ở nơi mà một app không có những biện pháp bảo vệ các chứng năng nhạy cảm.
- Ví dụ, admin func có thể được liên kết với admin's welcome page chứ không phải user's welcome page. Tuy nhiên, một user có thể truy cập admin func bằng cách duyệt các URL có liên quan tới admin
- Ví dụ, một website có thể chứa **Sensitive Functionality** ở URL:

      https://insecure-website.com/admin
- Cái này có thể được truy cập bởi bất cứ user nào, chứ không chỉ admin users người có liên kết với functionality trong giao diện người dùng của họ.
- Trong một số trường hợp, admin URL có thể được tiết lộ trong một địa chỉ khác, kiểu như ```robots.txt``` chẳng hạn:

      https://insecure-website.com/robots.txt
- Thâm chí nếu URL không hề được tiết lộ ở bất cứ đâu, một atker vẫn có thể sử dụng wordlist để brute-force ra địa chỉ của các **sensitive functionality**
## Lab
- Nhiệm vụ của ta là chiếm admin panel rồi xoá user ```carlos```
- Let's start:
  - Ở lab này thì ta có thể áp dụng gobuster hay burp măng để tìm các path nhưng tôi nghĩ không cần lắm nên đã thửu test luôn xem có path ```robots.txt``` hay không và có luôn :|| ![image](https://github.com/Myozz/Web_Applications/assets/94811005/22e6d074-7299-4272-b43a-2e75062c0d22)
  - Vào thử admin-panel bằng path trong ```robots.txt``` nhé :|| nó hiện ra một giao diện quản lý user ![image](https://github.com/Myozz/Web_Applications/assets/94811005/214d8879-8545-4262-ba84-f8b99732ccee)
  - Bấm delete và ez vcl

# Unprotected Functionality (tính năng không được bảo vệ) part 2
- Trong một số trường hợp, các functionality nhạy cảm được giấu trong một URL khó đoán :||. Đây là một ví dụ của cái gọi là **security by obscurity** (bảo mật tối nghĩa???). Tuy nhiên, ẩn functionality nhạy cảm không khiến kiểm soát truy cập hiệu quả hơn bởi user có thể tìm các URL bằng nhiều cách
- Ví dụ: Một app mà admin func được đặt ở URL: ```https://insecure-website.com/administrator-panel-yb556```
  - Có thể nó không thể đoán bởi atker. Tuy nhiên, app vẫn có thể leak URL tới user. URL có thể bị tiết lộ trong code JS của giao diện người dùng dựa vào vị trị (role) của user đó. Ví dụ: ![image](https://github.com/Myozz/Web_Applications/assets/94811005/6b366ce0-d604-462e-a7d9-67f23f246de6)
    - Đoạn script sẽ chuyển tới user's UI (user interface) có quả link trong script nếu họ là admin. Tuy nhiên, script củ lìn này lại làm lộ ra cái URL đấy :|| mà thế thì ai cũng xem được
## Lab đần đét
- Lại là con web củ lìn này, kiểu b gì ctrl + u chả có URL ![image](https://github.com/Myozz/Web_Applications/assets/94811005/0d7ba1f1-1bf2-4683-8fc9-595d9860ffea)

# Parameter-based access control methods (Phương thức kiểm soát truy cập dựa trên tham số?)
- Một số app xác định quyền truy cập người dùng dựa vào role của họ khi login, và sau đó lưu trữ thông tin này trong một nơi gọi là **user-controllable location** (vị trí do người dùng kiểm soát :||) - tôi dịch đần vc. Vị trí này có thể là:
  - Một field ẩn
  - Một cookie
  - Preset query string parameter (dịch đơn giản là truy vấn chuỗi đi :||)
    - Trông nó như này: ![image](https://github.com/Myozz/Web_Applications/assets/94811005/5b5a5ce7-8e03-4730-b590-e367116ed15e)
- App sẽ quyết định quyền truy cập dựa trên giá trị được gửi đi.
  - VD:
    - https://insecure-website.com/login/home.jsp?admin=true
      - Đại khái là login admin :||
    - https://insecure-website.com/login/home.jsp?role=1
      - Đại khái là login role = 1
- Cách này thiếu an toàn bởi một user có thể điều chỉnh giá trị và truy cập vào những functionality mà họ không được uỷ quyền, như admin :||

## Lab (lag vc)
- Giải xong lòi cả l nên lười viết :|| mai viết bù
