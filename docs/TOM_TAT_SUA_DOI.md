# TÓM TẮT LỖI & USE CASE ĐÃ SỬA — SE100

> Bản rút gọn của [DANH_GIA_USECASE.md](DANH_GIA_USECASE.md). Bản đầy đủ có phân tích chi tiết từng nhóm.
> Đã chốt: bỏ role SALES, bỏ công nợ NCC, khuyến mãi = voucher cấp đơn + hạng thẻ tối thiểu.

## 1. Lỗi phát hiện trong tài liệu

### Không nhất quán
- Mã UC trong ma trận khác đặc tả: ma trận ghi `STK-01..08`, đặc tả là `STK-04..08`; ma trận ghi `AUTH-04..12`, đặc tả chỉ đến `AUTH-10`.
- Nhóm `UC-POS` không có trong danh sách quy tắc đặt tên (mục 1.1).
- Actor trong đặc tả lệch với ma trận: SUPP (Manager “đề xuất”), REP (chỉ ghi Admin), POS-03/04 (Manager “duyệt” nhưng đặc tả không có bước duyệt), STK-08 (role “Quản lý kho” không tồn tại).
- Role SALES có mô tả nhưng không có cột ma trận và không có UC nào.
- Chia UC không đều: PROD gộp CRUD vào 1 UC, còn SUPP/CUS/PROM tách “danh sách / tìm kiếm / chi tiết” thành 3 UC.

### Copy–paste
- SUPP-01 bước 4: “System trả về tokens và thông tin user”.
- SUPP-07 tiêu đề ghi “Ngừng hợp tác” (nội dung là lịch sử nhập).
- REP-07, REP-08 tên ghi “Xem báo cáo khách hàng”.
- AUTH-04 tiêu đề ghi “Đổi mật khẩu” (nội dung là quản lý tài khoản).
- POS-04 ghi ID `UC04`; tham chiếu cũ `UC02`, `UC07`, `UC09`.
- CUS-06 nhắc “đơn đang giao”, CUS-04 nhắc “KH đã bị xóa” (bán tại quầy, không giao hàng, không xóa KH).

### Thiếu
- UC Mở ca (là điều kiện của bán hàng nhưng không có đặc tả).
- Toàn bộ đặc tả Xuất hủy (DB có 3 bảng, các UC khác tham chiếu “UC09”).
- Nhập kho không chọn NCC; không có bảng chi tiết phiếu nhập.
- Có tích điểm nhưng không có UC dùng điểm.
- Không có UC xem Audit Log, cấu hình hệ thống (ngưỡng cận hạn N, tỷ lệ điểm, chính sách đổi trả).
- Không có báo cáo lợi nhuận gộp / COGS dù ma trận và DB có nhắc.

### Lỗi nghiệp vụ / DB
| Vị trí | Lỗi |
|---|---|
| PROD-01 | Form không có **giá bán**, VAT, ngưỡng tồn (DB có) |
| STK-06 ↔ Business Rule 2 | Mâu thuẫn ngày hết hạn: bán hết ngày ghi trên hạn hay ngừng bán ngay trong ngày đó |
| STK-04 | Nhập thêm vào lô cũ với giá khác thì `cost_price` bị ghi đè, sai giá vốn |
| INVENTORY_CHECKS | Không có chỗ lưu phạm vi / cờ khóa lô; `counted_qty NOT NULL` trái với “lưu tạm” |
| ORDERS | `payment_method NOT NULL` nhưng đơn được tạo trước khi thanh toán; thiếu trạng thái nháp, `completed_at`, tiền khách đưa, tiền thừa |
| ORDERS / ORDER_DETAILS | Giảm giá gộp một cột, không phân bổ xuống dòng → **không tính được tiền hoàn** |
| RETURN_ORDER_DETAILS | Không liên kết `order_detail_id` → không kiểm được số lượng còn được trả |
| PROMOTION_USAGES | `customer_id NOT NULL` → khách vãng lai không dùng được voucher |
| PROM-02 | Form có “đối tượng tuổi”, “sản phẩm áp dụng” nhưng DB không có cột |
| REP-01 | Công thức doanh thu trừ chiết khấu 2 lần; Rule “đơn hoàn tiền toàn bộ không tính” không làm được |
| SUPP-02, CUS-02 | Form có địa chỉ, email, ngày sinh… nhưng DB thiếu cột |
| USERS | Thiếu cột cho khóa tạm 15 phút, mã NV, mật khẩu tạm |
| DAILY_SALES_REPORTS | Không có cơ chế sinh dữ liệu, sai số khi trả hàng muộn |

## 2. Use case: cũ → mới (58 → 40)

| Mới | Tên | Từ mã cũ |
|---|---|---|
| PROD-01..03 | Quản lý sản phẩm / danh mục / thương hiệu | Giữ nguyên, bổ sung trường form |
| STK-01 | Lập phiếu nhập kho theo lô (**chọn NCC**) | STK-04 |
| STK-02 | Tra cứu tồn kho theo sản phẩm & lô | STK-05 |
| STK-03 | Theo dõi hàng cận hạn / hết hạn | STK-06 |
| STK-04 | Lập phiếu kiểm kho | STK-07 |
| STK-05 | Duyệt điều chỉnh sau kiểm kho | STK-08 |
| STK-06 | Lập đề nghị xuất hủy | **Mới** |
| STK-07 | Duyệt đề nghị xuất hủy | **Mới** |
| SUPP-01 | Quản lý NCC (thêm, sửa, ngừng, khôi phục) | SUPP-02, 03, 06, 09 |
| SUPP-02 | Tra cứu & xem chi tiết NCC | SUPP-01, 04, 05, 08 |
| SUPP-03 | Xem lịch sử nhập hàng của NCC | SUPP-07 |
| CUS-01 | Tra cứu & xem hồ sơ KH | CUS-01, 04, 05 |
| CUS-02 | Quản lý hồ sơ KH (thêm, sửa, khóa/mở ưu đãi) | CUS-02, 03, 06, 07 |
| CUS-03 | Tích điểm, thu hồi điểm & xét hạng (Hệ thống) | CUS-08 |
| CUS-04 | Dùng điểm thưởng khi thanh toán | **Mới** |
| CUS-05 | Cấu hình tích điểm & hạng thẻ | CUS-09 |
| PROM-01 | Tra cứu khuyến mãi | PROM-01, 05 |
| PROM-02 | Quản lý khuyến mãi (tạo, sửa, tạm dừng, kích hoạt) | PROM-02, 03, 04 |
| PROM-03 | Áp dụng voucher khi bán hàng | PROM-06 |
| PROM-04 | Xem hiệu quả khuyến mãi | PROM-07 |
| POS-01 | Mở ca làm việc | **Mới** |
| POS-02 | Lập đơn bán hàng (FEFO) | POS-01 |
| POS-03 | Thanh toán & xuất hóa đơn | POS-02 |
| POS-04 | Đổi trả & hoàn tiền (1 giao dịch) | POS-03, 04 |
| POS-05 | Kiểm đếm tiền & chốt ca | POS-05 |
| REP-01 | Dashboard tổng quan | REP-07 |
| REP-02 | Báo cáo doanh thu & lợi nhuận gộp (có so sánh kỳ) | REP-01, 10 |
| REP-03 | Báo cáo bán hàng (ca, thu ngân, top SP) | REP-02, 05 |
| REP-04 | Báo cáo tồn kho & hạn dùng | REP-03 |
| REP-05 | Báo cáo nhập hàng | REP-04 |
| REP-06 | Báo cáo khách hàng | REP-06 |
| — | Lọc thời gian, xuất Excel → chức năng chung, không phải UC | REP-08, 09 |
| AUTH-01..03 | Đăng nhập, đăng xuất, đổi mật khẩu | Giữ nguyên |
| AUTH-04 | Quản lý tài khoản nhân viên (gồm gán vai trò, reset mật khẩu) | AUTH-04, 05, 06, 09 |
| AUTH-05 | Quản lý vai trò & phân quyền | AUTH-07, 08, 10 |
| AUTH-06 | Xem nhật ký hệ thống (Audit Log) | **Mới** |
| AUTH-07 | Cấu hình hệ thống | **Mới** |

## 3. Database: thay đổi chính (vẫn 30 bảng)

- **Bỏ 3 bảng:** `DISPOSAL_ALLOCATIONS` (thay bằng cột `blocked_added_qty`), `PROMOTION_USAGES` (thay bằng `ORDERS.promotion_id`), `DAILY_SALES_REPORTS` (báo cáo truy vấn trực tiếp).
- **Thêm 3 bảng:** `PURCHASE_ORDER_DETAILS`, `SYSTEM_SETTINGS`, `REFRESH_TOKENS` (chỉ cần nếu dùng JWT).
- **Bỏ cột:** `debt_limit`, `paid_amount`, `debt_amount`, `salesperson_id`, `is_stackable`.
- **Cột quan trọng nhất cần thêm:**
  - `ORDER_DETAILS.discount_allocated`, `line_total`: tính tiền hoàn.
  - `RETURN_ORDER_DETAILS.order_detail_id`, `unit_cost`: kiểm số lượng trả, trừ giá vốn.
  - `ORDERS.completed_at`, `promotion_id` và các cột tách giảm giá (hạng / voucher / điểm).
  - `BATCHES.locked_by_check_id`, `created_at`: khóa khi kiểm kho, tính tuổi tồn.
- **Rút gọn trạng thái:**
  - `ORDERS`: `DRAFT / COMPLETED / CANCELLED`.
  - `RETURN_ORDERS`: `COMPLETED / CANCELLED`.
  - `PURCHASE_ORDERS`: `DRAFT / COMPLETED / CANCELLED`.
  - `DISPOSAL_REQUESTS`: `WAITING_APPROVAL / APPROVED / REJECTED`.
  - `PROMOTIONS`: chỉ lưu `ACTIVE / PAUSED`, trạng thái theo ngày thì tính khi hiển thị.

Chi tiết từng cột: xem trang **Cơ sở dữ liệu** (chế độ “So sánh”) trong [html/index.html](html/index.html).

## 4. Quy tắc đã thống nhất lại

1. Tồn khả dụng = `on_hand − blocked`, chỉ tính lô còn hạn và không bị khóa kiểm kho.
2. Thứ tự giảm giá: hạng thẻ → voucher → điểm, phân bổ xuống từng dòng hàng.
3. Doanh thu thuần = tiền hàng − giảm giá − hàng bán bị trả (trả hàng trừ vào kỳ phát sinh trả). Lợi nhuận gộp = doanh thu thuần chưa VAT − giá vốn.
4. Mọi thay đổi tồn kho đi kèm 1 dòng `STOCK_MOVEMENTS` trong cùng transaction.

## 5. Còn chờ nhóm xác nhận

1 cửa hàng hay nhiều chi nhánh? · Giá bán đã gồm VAT? · Có bán lô trong ngày hết hạn? · Tỷ lệ dùng điểm? · Trả hàng có hạ hạng thẻ? · Có thanh toán kết hợp? · Đổi trả có cần quản lý duyệt? · Dùng JWT hay session?
