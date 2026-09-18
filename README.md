# BÀI TẬP 6: CHIẾN LƯỢC CỬA SỔ TRƯỢT: TIME-BASED VS COUNT-BASED

## Phần 1 – Phân tích cơ chế thu thập dữ liệu

1. **TIME_BASED (Dựa trên thời gian - Time-based sliding window):**
   - **Cơ chế hoạt động:** Cửa sổ trượt được chia thành các khoảng thời gian cố định (ví dụ: các ô thời gian - time buckets). Dữ liệu được tổng hợp dựa trên khoảng thời gian N giây gần nhất.
   - **Đặc điểm:** Phù hợp với lưu lượng truy cập ổn định theo thời gian. Nếu traffic thưa thớt, các ô thời gian cũ có thể chứa dữ liệu cũ không phản ánh đúng trạng thái hiện tại của hệ thống cho đến khi chúng dịch chuyển ra ngoài cửa sổ.

2. **COUNT_BASED (Dựa trên số lượng request - Count-based sliding window):**
   - **Cơ chế hoạt động:** Cửa sổ trượt duy trì trạng thái của N request gần đây nhất (ví dụ: 100 request gần nhất).
   - **Đặc điểm:** Phản ánh chính xác tỷ lệ lỗi dựa trực tiếp trên khối lượng request thực tế được xử lý, không phụ thuộc vào thời gian trôi qua nhanh hay chậm.

---

## Phần 2 – Lựa chọn giải pháp cho Flash Sale

1. **Rủi ro của TIME_BASED (60 giây) khi server sập ở giây đầu tiên:**
   - Giả sử lúc 12:00:00, traffic tăng vọt lên 50.000 request/giây và server chết ngay lập tức ở giây đầu tiên do quá tải.
   - Với cửa sổ thời gian 60 giây, hệ thống Circuit Breaker sẽ phải chờ hết khoảng thời gian gom mẫu (hoặc đủ dữ liệu trong cửa sổ thời gian) mới tính toán lại tỷ lệ lỗi. Trong suốt 59 giây tiếp theo, các request tiếp tục đổ vào và bị kẹt hoặc timeout vì Circuit Breaker chưa kịp mở do chưa đủ chu kỳ tổng hợp thời gian, gây lãng phí tài nguyên và làm trầm trọng thêm tình trạng sập hệ thống.

2. **Lựa chọn loại Cửa sổ trượt:**
   - **Lựa chọn:** `COUNT_BASED`.
   - **Lý do:** Trong kịch bản Flash Sale, lưu lượng tăng từ 10 request/phút lên 50.000 request/giây chỉ trong tích tắc. `COUNT_BASED` giúp Circuit Breaker phản ứng cực kỳ nhanh chóng dựa trên số lượng request thực tế (ví dụ: trong vòng 100 request gần nhất nếu có tỷ lệ lỗi vượt ngưỡng, mạch sẽ ngắt ngay lập tức) mà không phải chờ đợi khoảng thời gian 60 giây trôi qua, giúp bảo vệ hệ thống kịp thời trước khi quá tải lan rộng.

---

## Phần 3 – Triển khai cấu hình

Cấu hình Circuit Breaker sử dụng `COUNT_BASED` được đặt trong file `application.yml`.