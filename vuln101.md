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
