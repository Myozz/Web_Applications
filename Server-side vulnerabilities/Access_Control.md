# Access Control (Kiểm soát truy cập) là cl gì?
- Kiểm soát truy cập là việc áp dụng các ràng buộc về ai/cái gì được uỷ quyền để thực hiện các hành động hoặc truy cập tài nguyên. Trong web/app, kiểm soát truy cập phụ thuộc vào xác thực và quản lí phiên (session)
  - **Authentication** (xác thực): xác nhận rằng user là chính họ :|| nghe lú nhỉ, ý là account là của chính chủ ấy
  - **Session Management** (Quản lý phiên): Xác định HTTP request tiếp theo có phải đang được thực hiện bởi cùng một user không?
  - **Access Control** (Kiểm soát truy cập): Xác định xem liệu user có quyền để thực hiện hành động nào đó không?
- **Broken Access Control** (Kiểm soát truy cập ngố đét) khá phố biến và thường xuyên có mặt trong những lỗ hổng nghiêm trọng nhất. Thiết kế và Quản lý Access Control là một vấn đề phức tạp, cần các tổ chức cùng triển khai kỹ thuật. Các quyết định thiết kế Access Control phải được thực hiện bởi con người nên khả năng xảy ra sai sót là cao vcl

![image](https://github.com/Myozz/Web_Applications/assets/94811005/18205d4d-9bc1-49fa-9aca-a5c396f06417)
