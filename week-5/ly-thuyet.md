# Caching và Redis

## 1. Caching

**Caching là gì?** Cache là nơi lưu tạm bản sao của dữ liệu hay dùng, đặt ở chỗ đọc nhanh hơn nơi lưu gốc (thường là RAM thay vì database trên ổ đĩa).

**Tại sao cần?**
- Tăng tốc độ phản hồi.
- Giảm tải cho database.
- Phục vụ được nhiều người dùng hơn với cùng cấu hình server.

Đổi lại, dữ liệu trong cache có thể bị cũ, và hệ thống phức tạp hơn.

**Cache ở đâu?** Trình duyệt, CDN, trong ứng dụng, cache server riêng, bộ đệm của database.

| | Application-level cache | Distributed cache |
|---|---|---|
| Vị trí | Trong RAM của chính ứng dụng | Server cache riêng (VD Redis) |
| Tốc độ | Nhanh nhất | Rất nhanh (qua mạng) |
| Khởi động lại ứng dụng | Mất dữ liệu | Vẫn còn |
| Nhiều server ứng dụng | Mỗi server 1 bản, dễ lệch | Dùng chung 1 bản |

**Khi nào nên cache?** Dữ liệu được đọc nhiều, ít thay đổi, giống nhau cho nhiều người, chấp nhận được việc cũ một chút.

**Khi nào không nên?** Dữ liệu thay đổi liên tục, cần chính xác tuyệt đối (tiền, quyền truy cập, bảo mật), hoặc có quá nhiều tổ hợp khác nhau nên ít được dùng lại.

## 2. Chiến lược Caching

**Cache-Aside:** ứng dụng tự quản lý cache. Khi đọc, ứng dụng kiểm tra cache trước. Nếu có thì trả về ngay, nếu không thì đọc database rồi lưu vào cache kèm thời gian sống. Khi dữ liệu thay đổi, ứng dụng cập nhật database rồi xóa cache tương ứng.

**Write-Through:** mỗi lần ghi, dữ liệu được ghi vào cả cache và database cùng lúc, nên cache luôn có dữ liệu mới nhất.

| | Cache-Aside | Write-Through |
|---|---|---|
| Đọc lần đầu | Chậm (cache miss) | Nhanh |
| Ghi | Nhanh | Chậm hơn (ghi 2 nơi) |
| Dữ liệu trong cache | Chỉ dữ liệu được đọc | Mọi dữ liệu được ghi |
| Khi cache lỗi | Vẫn chạy, đọc database | Ảnh hưởng thao tác ghi |
| Phù hợp | Đọc nhiều, ghi ít (phổ biến nhất) | Dữ liệu vừa ghi cần đọc ngay |

## 3. Redis cơ bản

**Redis** là database lưu dữ liệu trong RAM theo dạng key - value, tốc độ rất cao, thường dùng làm cache, lưu phiên đăng nhập, bộ đếm, hàng đợi. Mỗi key có thể đặt thời gian sống, hết hạn thì tự xóa.

**Kiểu dữ liệu:**
- **String:** một key ứng với một giá trị (chữ, số, JSON).
- **Hash:** một key chứa nhiều cặp trường - giá trị, giống một object.
- **List:** danh sách có thứ tự, thêm và lấy ở hai đầu.
- **Set:** tập hợp các phần tử không trùng nhau.
- **Sorted Set:** tập hợp có điểm số, tự sắp xếp theo điểm (dùng cho bảng xếp hạng).

**Lệnh cơ bản:**
- **SET:** ghi giá trị cho một key.
- **GET:** đọc giá trị của một key.
- **DEL:** xóa một key.
- **EXPIRE:** đặt thời gian sống cho key.
- **TTL:** xem key còn sống bao lâu (-1 là không có hạn, -2 là không tồn tại).

**Cài đặt bằng Docker:** tải image Redis chính thức, chạy container và mở cổng 6379. Sau đó dùng công cụ dòng lệnh `redis-cli` để kết nối và kiểm tra.

## 4. Tích hợp Redis vào project

1. Cài Redis và thêm thư viện client cho ngôn ngữ đang dùng (Java: Jedis hoặc Lettuce; Node: redis; Python: redis-py).
2. Cấu hình kết nối: địa chỉ, cổng, mật khẩu (nếu có).
3. Áp dụng Cache-Aside cho dữ liệu như thông tin sản phẩm, người dùng:
   - Request đến, kiểm tra cache theo key.
   - Có dữ liệu thì trả về ngay.
   - Không có thì truy vấn database, lưu kết quả vào cache với TTL, rồi trả về.

## 5. Các vấn đề thường gặp

**TTL:** dữ liệu càng ít thay đổi và càng chấp nhận được việc cũ thì TTL càng dài. TTL quá ngắn làm cache ít tác dụng, quá dài làm người dùng thấy dữ liệu cũ lâu.

**Cache Invalidation:** khi dữ liệu thay đổi, phải xóa hoặc cập nhật cache để tránh trả dữ liệu cũ. Cách phổ biến là cập nhật database trước rồi xóa cache, để lần đọc sau tự nạp lại dữ liệu mới.

**Cache Avalanche:** rất nhiều key hết hạn cùng lúc (hoặc cache server bị sập), khiến mọi request đổ dồn vào database và làm database quá tải. Cách xử lý: cộng thêm thời gian ngẫu nhiên vào TTL, chạy cache dự phòng, giới hạn số request, nạp sẵn cache khi khởi động.

**Cache Penetration:** truy vấn dữ liệu không tồn tại nên không bao giờ có trong cache, mọi request đều xuống database. Cách xử lý: lưu cả kết quả "không tồn tại" vào cache với TTL ngắn, kiểm tra dữ liệu đầu vào, giới hạn số request, dùng Bloom filter.
