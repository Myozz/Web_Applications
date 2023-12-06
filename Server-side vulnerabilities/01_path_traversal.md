# Path Traversal (cơn mưa ngang qua path - duyệt đường dẫn, duyệt thư mục)
- Path traversal (directory traversal). Những lỗ hổng này cho phép atker đọc file bất kì trên server đang chạy một application. Nó có thể bao gồm:
  - Application code and data
  - Credentials for back-end systems (Thông tin xác thực cho back-end sys)
  - Sensitive Operating system files (file nhạy cảm)
- Trong một số trường hợp, một atker có thể ghi vào file bất kỳ trên server, cho phép họ có thể chỉnh sửa dữ liệu hay hành vi của app, từ đó chiếm toàn quyền server

# Đọc file bất kỳ qua path traversal
- Thử tưởng tượng một app shopping (shopee đi) hiện thị các mặt hàng sale. Nó sẽ phải load các img bằng cách sử dụng **F12** hay ngày xưa người ta gọi là **Kiểm tra phần tử**, ví dụ: ```<img src="/loadImage?filename=218.png">```
- ```loadimage``` URL sẽ lấy tham số ở ```filename``` (trong ví dụ là 218.png) và trả về nội dung của file được chỉ định. File img được lưu trữ ở ```/var/www/images/```. Để trả về một img, app sẽ thêm filename được request tới đường dẫn này và sử dụng filesystem API để đọc nội dung của file. Nói cách khác, app đọc từ path ```/var/www/images/218.png```
- Application nếu không có biện pháp phòng thủ trước kiểu tấn công path traversal. Kết quả tất yếu sẽ là atker có thể request URL sau đây để lấy được ```/etc/password```

      https://insecure-website.com/loadImage?filename=../../../etc/passwd
- URL này sẽ khiến app đọc đường dẫn ```/var/www/images/../../../etc/passwd```
  - **Giải thích cơ chế**: ```https://insecure-website.com/loadImage?filename=``` là nó đang ở path ```/var/www/images```. Nếu thêm vào ```../../../``` từ nó sẽ lùi path dần ra root dir. Rồi ```/etc/passwd``` thì nó sẽ vào và mở passwd
- Trên Unix-baseđ OS, đấy là một file tiêu chuẩn chứa thông tin chi tiết của users đã đăng ký trên server, và atker còn có thể lấy một file bất kỳ bằng kỹ thuật này
- Trên windows, cả ```../``` và ```..\``` đều dùng để lùi path. Đây là một ví dụ và một URL tương tự với Windows-based server

      https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini
  - Như đã nói, ```../``` và ```..\``` đều dùng để lùi path về path trước
## Lab
- Lab này cho ta thấy một con web như này ![image](https://github.com/Myozz/Web_Applications/assets/94811005/a006a1ed-c549-4b70-8457-c269e816ae72)
- Có thể thấy được đây là một con web shopping chứa cực kì nhiều hình ảnh, bắt đầu nào:
  - Dùng Inspect trên trình duyệt của bạn, hay phím tắt thường là **F12** ![image](https://github.com/Myozz/Web_Applications/assets/94811005/d0e24f03-7f5d-495c-9e9b-3fd561b7e079)
  - Bây giờ ta chia con trỏ vào một img bất kỳ rồi click
  - Trên **Element** sau đó sẽ hiện ra path của img đó ![image](https://github.com/Myozz/Web_Applications/assets/94811005/d3670b86-4fd0-4524-b63c-4e5f048b76ae)
  - Ok bây giờ đến bước sửa URL, ta sẽ copy path của img rồi paste rồi đuôi của url page, khi đó nó sẽ như này ```https://0a4800a404ca2b1481073efa00eb008f.web-security-academy.net/image?filename=../../../etc/passwd```. Khi đó ta sẽ lấy được ```/etc/passwd```

# Những trở ngại chung khi khai thác lộ hổng path traversal (truyền tải đường dẫn)
- Nhiều app đặt user input vào đường dẫn tệp sẽ triển khai các biện pháp def chống lại các cuộc tấn công truyền tải đường dẫn. Những điều này có thể sẽ bị bỏ qua
- Nếu một dải/khối trình tự duyệt thư mục từ filename dược user cung cáp, nó có lẽ sẽ khả thi để bypass def bằng cách sử dụng các thuật toán đa dạng
- Bạn có lẽ sẽ có thể sử dụng một path tuyệt đối từ filesystem root, như ```filename=/etc/passwd```, để tham chiếu trực tiếp một file mà không cần sử dụng bất cứ trình tự đường dẫn nào
- Bạn có thể sử dụng trình tự duyệt lồng nhau, như ```....//``` hay ```....\/```. Chúng sẽ quay trở lại các trình tự truyền tải đơn giản khi trình tự bên trong bị loại bỏ
- Trong một số hoàn cảnh, như là trong một URL path hay tham số ```filename``` của một ```multipart/form-data``` request
