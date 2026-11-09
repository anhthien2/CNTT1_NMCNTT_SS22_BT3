# PHẦN 1: PHÂN TÍCH 2 GIẢI PHÁP XÁC THỰC

**1.  yếu tố xác thực danh tính số**

Theo chuẩn NIST, xác thực được chia thành 3 nhóm:

-   **Know - Cái bạn BIẾT:** Mật khẩu, PIN, câu hỏi bảo mật.
-   **Have - Cái bạn CÓ:** Điện thoại, token, smart card, mã OTP.
-   **Are - Cái bạn LÀ:** Vân tay, FaceID, giọng nói.

Kẻ tấn công phải có cả 2 yếu tố cùng lúc mới chiếm được tài khoản chứa source code trăm ngàn USD. Chỉ có mật khẩu là không đủ.

**2. Nguyên lý TOTP - Vì sao bật Chế độ Máy bay vẫn sinh mã được?**

TOTP - Time-based One-Time Password theo chuẩn RFC 6238, không hề dùng mạng.

Công thức: TOTP = HMAC-SHA1(Shared Secret, Current Time / 30s)

Quá trình:

1.  **Lúc quét QR lần đầu:** Server GitHub và App Authenticator của bạn cùng chia sẻ 1 khóa bí mật Shared Secret- ví dụ JBSWY3DPEHPK3PXP - khóa này chỉ lưu 2 nơi.
2.  **Lúc sinh mã:** Cả server và điện thoại đều lấy thời gian hiện tại chia cho 30 giây làm mốc, rồi băm với Shared Secret bằng HMAC-SHA1 để ra mã 6 số.
3.  **Vì sao Offline vẫn chạy?** Vì điện thoại không cần gọi API. Nó chỉ cần đồng hồ hệ thống. Dù bật Airplane Mode, đồng hồ vẫn chạy, nên mã ở điện thoại và mã server tính ra trùng khớp 100% trong 30 giây.

Đây là lý do dev đi GDC ở Mỹ, ngồi trên máy bay không có sóng vẫn đăng nhập được GitHub.

Ngược lại SMS OTP thì bắt buộc phải có sóng viễn thông, server nhà mạng phải gửi tin nhắn qua mạng SS7 đến SIM của bạn.
# PHẦN 2: BẢNG SO SÁNH TRADE-OFF & RA QUYẾT ĐỊNH
| Tiêu chí đánh giá | Giải pháp 1: SMS OTP | Giải pháp 2: TOTP Authenticator |
| :--- | :--- | :--- |
| 1. Chống SIM Swapping & Nghe lén SS7 | Kém - Lỗ hổng chết người. Kẻ xấu dùng Social Engineering gọi tổng đài báo mất SIM, chiếm số Founder hoặc tấn công SS7 để chặn tin nhắn. | Tuyệt đối an toàn. Shared Secret lưu trong Secure Enclave, không truyền qua sóng di động. Nhà mạng bị hack cũng không liên quan. |
| 2. Tính sẵn sàng Offline / Nước ngoài | Phụ thuộc sóng. Ra nước ngoài, lên máy bay, vào hầm GDC là không nhận được SMS, không deploy được. | 100% Offline. Hoạt động mọi lúc mọi nơi, kể cả Airplane Mode. |
| 3. Tiện dụng & Chi phí | Quen thuộc nhưng tốn kém 300-1500đ/SMS, bị delay. | Miễn phí 100%. Mở app là có mã ngay, copy 1 chạm. |
| 4. Rủi ro khi mất điện thoại | Làm lại SIM là lấy lại được, nhưng ai nhặt được SIM là chiếm luôn tài khoản. | Cần Backup Codes + Cloud Sync mã hóa để khôi phục an toàn. |
 **4. Kết luận lựa chọn cho Ban Giám đốc**

**Vì 3 lý do:**

1.  **An toàn:** GitHub/Stripe chứa cả source code và tiền - SMS dễ bị cướp SIM, NIST đã cấm dùng cho tài khoản quan trọng từ 2016.
2.  **Tiện cho team:** Dev hay đi GDC, ngồi máy bay, ra nước ngoài - SMS không có sóng là chịu chết, TOTP thì offline vẫn chạy.
3.  **Rẻ + chống lừa:** Miễn phí, không phụ thuộc nhà mạng nên không bị phishing.

**Triển khai:** Dùng Microsoft Authenticator/Authy có backup mã hóa, cấm để SMS làm 2FA chính.
# PHẦN 3: QUẢN LÝ RỦI RO & BACKUP CODES

**5. Mục đích và cách lưu trữ 10 Mã Khôi phục**

**Mục đích:** Backup Codes là 10 mã dùng 1 lần, mỗi mã 8 ký tự. Dùng khi bạn mất điện thoại, xóa nhầm app Authenticator, hoặc đồng hồ điện thoại bị lệch. Mỗi mã chỉ dùng được 1 lần, dùng xong là hủy. Nó là chìa khóa cuối cùng để không bị khóa vĩnh viễn khỏi GitHub.

**2 vị trí an toàn tuyệt đối để lưu:**

1.  **In ra giấy và cho vào két sắt** cùng với Emergency Kit của công ty. Đây là bản Offline không thể bị hack.
2.  **Lưu vào Password Vault - Bitwarden:** Tạo 1 Secure Note tên GitHub - Backup Codes - [Tên NV] trong Organization Vault và đính kèm file txt. Vault đã được mã hóa Zero-Knowledge nên an toàn hơn Drive.
