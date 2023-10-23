*File này được tạo ra chỉ bởi vì OSI model khó nhớ vcl
# Một chút khái nghiện
- OSI (Open System Interconnection Reference) Model hay mô hình tham chiếu kết nối hệ thống mở. Mô hình này mô tả 7 tầng mà hệ thống máy tính sử dụng để giáo tiếp qua mạng. Đây là mô hình tiêu chuẩn đầu tiên cho truyền thông mạng, được tất cả các công ty máy tính và viễn thông lớn ấp dụng vào đầu 1980
- Bây giờ ngta chuyển dần sang dùng mô hình TCP/IP nhưng bằng cách thân kì nào đó ta vẫn phải học thứ củ lìn này từ môn này sang môn khác.
# 7 Tầng
Ok đây là 7 tầng mà chúng ta đ bao giờ nhớ :)) Tôi học trên youtube một cách nhớ như này: "Please Do Not Throw Sausage Pizza Away", mỗi chữ cái đầu tiên tương ứng với chữ cái đầu của một tầng
![image](https://github.com/Myozz/Web_Applications/assets/94811005/334178f1-dc52-4452-8d70-d3a8658a095c)

## Tầng 1 - Physical
- Tầng này gồm các thiết bị vật lú liên quan đến truyền dữ liệu (Switch, Router, Cáp,...)
- Dữ liệu ở đây được biến đổi thành bit (1/0)
## Tấng 2 - Data Link
- Tầng data link lấy các packet từ tấng network và chia thành các frame.
- Chức năng tầng này là điều khiển luồng và điều khiển lõi trong giao tiếp nội mạng
## Tấng 3 - Network
- Có nhiệm vụ tạo điều kiện thuận lợi cho việc truyền dữ liệu giữa 2 mạng khác nhau
- Nếu 2 thiết bị cùng mạng thì tầng này là không cần thiết
- Tầng này chia nhỏ các phân đoạn từ tấng Transport thành các packet.
- Tìm ra con đường tốt nhất điều truyền dữ liệu (định tuyến)
## Tầng 4 - Transport
- Có nhiệm vụ giao tiếp đầu cuối giữa 2 thiết bị.
- Lấy Data từ tấng Session và chia nó thành phân đoạn (Segments) trước khi gửi đến tấng Net.
- Tập hợp lại các segment thành dữ liệu để tầng session có thể sử dụng
## Tầng 5 - Session
- Quản lí giao tiếp 2 thiết bị
- Đảm bảo rằng phiên vẫn mở đủ lâu để chuyển tất cả dữ liệu đang được trảo đổi và sau đó nhanh chóng đóng phiên đã tránh lãng phí tài nguyên
## Tầng 6 - Presentation
- Có nhiệm vụ dịch, mã hoá, nén dữ liệu để phục vụ hiển thị dữ liệu
## Tầng 7 - App 
- Xác định giao diện giữa user và OSI
- Cung cấp các giao thức cho phép phần mềm gửi, nhận thông tin và trình bày dữ liệu có ý nghĩa cho người dùng
## Tóm tắt lại
![image](https://github.com/Myozz/Web_Applications/assets/94811005/0629de61-15e5-458b-a84c-7e536e1a9c9a)

# Đối chiếu với TCP/IP
![image](https://github.com/Myozz/Web_Applications/assets/94811005/b9c15abf-b7d8-4f0a-956f-7fc690e9a981)
