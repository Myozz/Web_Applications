# Unprotected Functionality (Chức năng không được bảo vệ)
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

# Unprotected Functionality (Chức năng không được bảo vệ) part 2
- Trong một số trường hợp, 
