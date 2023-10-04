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
- Một số app xác định quyền truy cập người dùng dựa vào role của họ khi login, và sau đó lưu trữ thông tin này trong một nơi gọi là **user-controlable location** (vị trí do người dùng kiểm soát :||) - tôi dịch đần vc. Vị trí này có thể là:
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

# Horizontal Privilege Escalation (Leo thang đặc quyền theo chiều ngang)
- Horizontal Priv Esc xảy ra nếu một user có thể chiếm quyền truy cập tài nguyên thuộc sở hữu của user khác, thay vì là tài nguyên của riêng họ. Ví dụ, nếu một nhân viên có thể truy cập vào bản ghi của các nhân viên khác tự nhiên như ruồi, thì đó là horizontal priv esc
- Horizon Priv Esc Attack có thể sử dụng cùng phương thức với vertical priv esc. Ví dụ, một user có thể truy cập tài khoản của họ bằng URL:

      https://insecure-website.com/myaccount?id=123
- Nếu một atker sửa tham số ```id``` thành id của user khác, họ có thể chiếm quyền truy cập account page, dữ liệu và các tính năng của user đó
- **NOTE**: Bên trên là một ví dụ của một **Insecure direct object reference** (IDOR) - tham chiếu đối tượng trực tiếp không an toàn. Loại lỗ hổng này phát sinh ở giá trị tham số của user-controller được sử dụng để truy cập tài nguyên hay tính năng trực tiếp
- Trong một số app, các tham số có thể bị khai thác có giá trị ở dạng không dễ đoán. Ví dụ, thay vì dùng các số theo thứ tự tăng dần làm id hay đại khái vậy, một app có thể sử dụng **Globally Unique Indentifiers** (GUIDs) - Số nhận dạng toàn cầu (Có thể hiểu nó có cơ chế gần như số seri hay ID CCCD). Tuy nhiên, các GUIDs của các user có thể bị rò rỉ ra một nơi khác trong app, nơi mà user được tham chiếu đên (ví dụ như tin nhắn hay bình luận)

## Lab (GUID)
- Lab này yêu cầu tìm GUID của carlos rồi gửi API key của user này đi để hoàn thành
- Dm lúc nào hay bị quên ghi lại quá :|| đại khái lab này là vào blog mà author của nó là thằng carlos, sau đó check source để xem id của nó. Lấy nó rồi priv esc thôi

# Horizon to Vertical Priv Esc
- **NOTE**: Điểm khác nhau giữa Vertical và Horizontal là:
  - Vertical tấn công để chiếm quyền truy cập các user cấp cao hơn
  - Horizontal tấn công chiếm quyền truy cập các user có quyền hạn ngang hàng, từ đó triển khai các mục đích khác
- Thường thì một Horizon Priv Esc Atk có thể được chuyển sang Vertical Priv Esc, bằng cách xâm nhập vào tài khoản của user có đặc quyền cao hơn. Ví dụ, một Horizon Esc có thể cho pháp một atker reset hay chôm pass của user khác. Nếu atker hướng mục tiêu tới admin và có được tài khoản có họ, thì họ sẽ có thể chiếm quyền truy cập của admin và thực hiện vertical priv esc
- Một atker có thể chiếm quyền truy cập vào account page của user khác, bằng cách sử dụng kĩ thuật giả mạo tham số đã được đề cập trong phần **Horizon Priv Esc**

      https://insecure-website.com/myaccount?id=456
- Nếu user mục tiêu là một app admin, thì atker sẽ chiếm quyền truy cập tới admin acc page. Page này sẽ tiết lộ admin pass hay cung cập phương tiện đổi pass, hay cũng có thể cung cập quyền truy cập trực tiếp vào các tính năng đặc quyền
## Lab củ lìn
- Lab này có một admin acc page, nơi chứa pass của các user
- Để hoàn thành, chôm pass của admin và sử dụng để xoá ```carlos``` (tội thằng carlos vcl :))
- login to this fcking user ![image](https://github.com/Myozz/Web_Applications/assets/94811005/ff908b42-5cc0-415e-939a-ea69a6a21d90)
- Thay id là ```administrator```
- Vào admin panel, ta thấy có một pass đang chờ bị xử :|| ![image](https://github.com/Myozz/Web_Applications/assets/94811005/aef28323-9927-4a80-9e7f-01ab867c67a1)
- Dùng inspect (kiểm tra phần tử) để sửa type của password thì nó sẽ hiện lên, từ đấy login vào admin và xoá cu các-lốt
