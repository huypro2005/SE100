# KHUYẾN MÃI — THIẾT KẾ DỮ LIỆU VÀ CÁCH TÍNH (SE100)

> **Trạng thái: bản đề xuất.** Khi nhóm chốt, phần bảng ở mục 3 sẽ được gộp vào [DATABASE.md](DATABASE.md) (thay mục PROMOTIONS hiện tại).
> Code mẫu chạy được: [mau-code/khuyen-mai/](mau-code/khuyen-mai/). Chạy test: `node --experimental-strip-types --test docs/mau-code/khuyen-mai/test.ts`.
> Tiền là số nguyên VNĐ. Giá niêm yết **đã gồm VAT** (chờ xác nhận, câu hỏi số 2 mục 8 DANH_GIA_USECASE).

## 1. Phạm vi đã chọn

| # | Loại | `promotion_type` | Tầng |
|---|---|---|---|
| 1 | Giảm % / tiền theo sản phẩm, thương hiệu, danh mục | `PRODUCT_DISCOUNT` | 1. Dòng hàng |
| 2 | Đồng giá giờ vàng | `FLASH_PRICE` | 1. Dòng hàng |
| 3 | Mua X tặng Y | `BUY_X_GET_Y` | 1. Dòng hàng |
| 5 | Combo giá cố định (combo là **khuyến mãi**, không phải mặt hàng riêng) | `COMBO` | 1. Dòng hàng |
| 9 | Ưu đãi theo hạng | Không phải bản ghi KM: dùng `CUSTOMER_TIERS.discount_rate` | 2. Hạng thẻ |
| 6 | Bậc thang theo hóa đơn | `ORDER_TIERED` | 3. Đơn hàng |
| 8 | Voucher mã | `VOUCHER` | 3. Đơn hàng |
| 7 | Quà tặng theo hóa đơn | `ORDER_GIFT` | 3. Đơn hàng (quà) |
| — | Điểm thưởng (1 điểm = 100đ, tối đa 50%) | Không phải KM: `ORDERS.points_used` | 4. Điểm |

Không làm: 4 (món thứ N+1), 10 (nhân điểm), 11 (chi tiêu đổi voucher), 12 (giảm cận date theo lô).

**Vì sao combo là khuyến mãi:** mỗi món trong combo là một dòng `ORDER_DETAILS` gắn **một lô** như hàng thường, nên FEFO, trả hàng và thuế từng món dùng lại cơ chế có sẵn. (Phần mềm làm combo thành mặt hàng riêng như KiotViet không cho thành phần là hàng quản lý theo lô – hạn dùng.) Giá bộ được chia về từng món **theo tỷ lệ giá niêm yết** — lựa chọn của nhóm, tham khảo cách phân bổ mặc định của IFRS 15.

## 2. Thứ tự tính (4 tầng)

| Tầng | Nội dung | Quy tắc |
|---|---|---|
| **1. Dòng hàng** | 1, 2, 3, 5 | Tự động. **Mỗi đơn vị sản phẩm hưởng tối đa 1 KM tầng 1.** Hệ thống chọn lần lượt KM có lợi nhất cho khách trên phần hàng còn lại (giá trị quà tính theo giá niêm yết; quà không bán tính theo giá vốn). Bằng nhau thì lấy KM tạo trước |
| **2. Hạng thẻ** | 9 | `discount_rate` × tiền sau tầng 1, áp cả món đã giảm ở tầng 1 |
| **3. Đơn hàng** | 6, 8, 7 | **Tối đa 1 khoản giảm tiền**: bậc thang và voucher cùng tính, lấy cái lợi hơn; **bằng nhau lấy bậc thang** (voucher không bị trừ lượt). Quà theo hóa đơn (7) cộng thêm được, chọn chương trình có ngưỡng cao nhất đạt được |
| **4. Điểm** | — | Trừ trên phần còn lại, tối đa `POINT_MAX_REDEEM_PERCENT` |

- **Ngưỡng** `min_order_value` của 6, 7, 8 xét trên **tiền sau tầng 1 và 2** (không tính dòng quà).
- **Phân bổ:** giảm tầng 2, 3, 4 chia xuống từng dòng theo tỷ lệ tiền còn lại của dòng. Làm tròn đến đồng, **dòng cuối nhận phần dư** để tổng luôn khớp.
- **Tách lô FEFO** sau khi tính giá: một dòng sản phẩm có thể tách thành nhiều dòng theo lô, tiền chia theo số lượng (`splitByBatches`).
- Bỏ `is_stackable` và `priority` của từng chương trình: thứ tự cố định ở trên thay thế.

## 3. Thiết kế bảng

### 3.1. PROMOTIONS (thay bảng hiện tại)

| Cột | Kiểu | Ràng buộc | Dùng cho | Mô tả |
|---|---|---|---|---|
| promotion_id | INT | PK | | |
| promotion_name | VARCHAR(255) | NOT NULL | | |
| description | TEXT | NULL | | |
| **promotion_type** | VARCHAR(30) | NOT NULL | Tất cả | Xem mục 1. **Đây là cột để phân biệt loại KM** |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'DRAFT' | Tất cả | `DRAFT`, `ACTIVE`, `PAUSED`. "Sắp diễn ra / Đã kết thúc" tính theo ngày |
| voucher_code | VARCHAR(50) | UNIQUE, NULL | 8 | Bắt buộc khi `VOUCHER`, NULL với loại khác |
| customer_id | INT | NULL, FK → CUSTOMERS | 8 | Voucher riêng 1 khách (quản lý tạo khi cần; trả hàng **không** phát voucher bù) |
| apply_scope | VARCHAR(20) | NULL | 1 | `ALL`, `PRODUCT`, `BRAND`, `CATEGORY` |
| discount_type | VARCHAR(20) | NULL | 1, 8 | `PERCENT`, `FIXED_AMOUNT` |
| discount_value | DECIMAL(12,2) | NULL | 1, 8 | |
| max_discount_amount | DECIMAL(12,2) | NULL | 1, 8 | Trần giảm (loại 1: trần cho mỗi dòng) |
| fixed_price | DECIMAL(12,2) | NULL | 2 | Giá đồng giá; chỉ áp khi thấp hơn giá niêm yết |
| combo_price | DECIMAL(12,2) | NULL | 5 | Giá cả bộ |
| buy_quantity | INT | NULL | 3 | Số món nhóm "mua" cần có |
| is_repeatable | BOOLEAN | NOT NULL, DEFAULT FALSE | 3 | TRUE: mua 4 tặng 2 (lặp theo bội số) |
| min_order_value | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | 7, 8 | Ngưỡng đơn (sau tầng 1, 2) |
| min_tier_id | INT | NULL, FK → CUSTOMER_TIERS | Tất cả | Hạng tối thiểu; khác NULL thì bắt buộc có khách |
| start_at / end_at | DATETIME | NOT NULL, end ≥ start | Tất cả | Thời gian chương trình |
| daily_start_time / daily_end_time | TIME | NULL | 2 | Khung giờ trong ngày (giờ vàng) |
| weekdays | VARCHAR(20) | NULL | 2 | Các thứ áp dụng, vd `"1,3,5"` (1 = thứ Hai … 7 = Chủ nhật). NULL = mọi ngày |
| max_quantity | INT | NULL | 2 | Tổng số lượng bán giá KM |
| usage_limit | INT | NULL | Tất cả | Tổng số đơn được dùng |
| limit_per_customer | INT | NULL | Tất cả | Số đơn mỗi khách; khác NULL thì bắt buộc có khách |
| created_by / created_at / updated_at | | | | |

Không dùng `JSONB`: mỗi thông số là một cột riêng (cho phép NULL), chạy được trên MySQL / SQL Server.

**Cột nào bắt buộc theo loại** (kiểm ở tầng ứng dụng khi lưu KM):

| Loại | Bắt buộc | Bảng con |
|---|---|---|
| `PRODUCT_DISCOUNT` | `apply_scope`, `discount_type`, `discount_value` | `PROMOTION_ITEMS` (role `TARGET`) / `PROMOTION_BRANDS` / `PROMOTION_CATEGORIES` theo scope |
| `FLASH_PRICE` | `fixed_price` | `PROMOTION_ITEMS` role `TARGET` |
| `BUY_X_GET_Y` | `buy_quantity` | `PROMOTION_ITEMS` role `BUY` và `GIFT` (kèm `quantity` quà) |
| `COMBO` | `combo_price` | `PROMOTION_ITEMS` role `COMBO_ITEM` (kèm `quantity`) |
| `ORDER_TIERED` | — | `PROMOTION_STEPS` ≥ 1 bậc |
| `ORDER_GIFT` | `min_order_value` | `PROMOTION_ITEMS` role `GIFT` |
| `VOUCHER` | `voucher_code`, `discount_type`, `discount_value` | — |

### 3.2. Bảng con

| Bảng | Cột | Ghi chú |
|---|---|---|
| **PROMOTION_ITEMS** | `promotion_id` (FK), `product_id` (FK), `role` (`TARGET` / `BUY` / `GIFT` / `COMBO_ITEM`), `quantity` (INT, DEFAULT 1). PK (`promotion_id`, `product_id`, `role`) | Quà có thể là SKU `is_sellable = FALSE` (túi tote) |
| **PROMOTION_BRANDS** | `promotion_id`, `brand_id`. PK cả hai | Phạm vi theo thương hiệu |
| **PROMOTION_CATEGORIES** | `promotion_id`, `category_id`. PK cả hai | Phạm vi theo danh mục |
| **PROMOTION_STEPS** | `step_id` PK, `promotion_id`, `min_order_value`, `discount_type`, `discount_value`, `max_discount_amount`. UNIQUE (`promotion_id`, `min_order_value`) | Mỗi bậc một dòng |

**Khóa sửa:** KM đã có đơn dùng thì không cho sửa mức giảm, điều kiện, bảng con (chỉ được tạm dừng / sửa ngày kết thúc). Nhờ vậy lúc trả hàng, dữ liệu KM giống hệt lúc bán.

### 3.3. ORDERS (sửa)

| Cột | Thay đổi | Lưu gì |
|---|---|---|
| subtotal_amount | Giữ | Σ `quantity × unit_price` (giá niêm yết; dòng quà = 0) |
| **line_discount_amount** | Mới | Tổng giảm tầng 1 |
| tier_discount_amount | Giữ | Giảm tầng 2 |
| **order_promotion_id** | Đổi tên từ `promotion_id` | KM giảm tiền cấp đơn **đã thắng** (bậc thang hoặc voucher). Voucher thua thì không ghi |
| **order_discount_amount** | Đổi tên từ `voucher_discount_amount` | Giảm tầng 3 |
| points_used, points_discount_amount | Giữ | Tầng 4 |
| discount_amount | Sửa công thức | `line + tier + order + points` |
| total_amount | Giữ | Σ `line_total` |

### 3.4. ORDER_DETAILS (sửa) — nơi lưu mọi con số dùng khi trả hàng

| Cột | Thay đổi | Lưu gì |
|---|---|---|
| unit_price | Giữ | Giá niêm yết lúc bán. **Dòng quà = 0** |
| **line_type** | Mới | `NORMAL` / `GIFT` |
| **line_promotion_id** | Mới, FK NULL | KM tầng 1 của dòng (1, 2, 3, 5); với dòng quà: KM sinh ra quà (3 hoặc 7) |
| **promo_group_no** | Mới, INT NULL | Số thứ tự bộ trong đơn: các dòng cùng (`line_promotion_id`, `promo_group_no`) là **một combo / một bộ mua-tặng** |
| **line_discount** | Mới | Giảm tầng 1 của dòng (combo: phần giảm đã tách về món này) |
| **tier_discount**, **order_discount**, **points_discount** | Mới (thay `discount_allocated`) | Phần giảm tầng 2, 3, 4 phân bổ cho dòng |
| line_total | Sửa công thức | `quantity × unit_price − line_discount − tier_discount − order_discount − points_discount` |
| tax_amount | Giữ | VAT tách ngược từ `line_total` |
| cost_price, batch_id | Giữ | Dòng quà vẫn trừ lô và ghi giá vốn |

Tách `discount_allocated` thành 3 cột để lúc trả hàng lấy **chính xác** tiền thực trả của dòng và tiền sau tầng 2 (xét lại ngưỡng quà), không phải ước lượng.

### 3.5. RETURN_ORDERS / RETURN_ORDER_DETAILS (sửa)

| Bảng.cột | Thay đổi | Lưu gì |
|---|---|---|
| RETURN_ORDER_DETAILS.**line_kind** | Mới | `RETURN` (nhận hàng về) / `GIFT_CHARGE` (tính tiền quà khách không trả lại, **không nhập kho**) |
| RETURN_ORDER_DETAILS.**refund_amount** | Mới | Tiền của dòng; **âm** với `GIFT_CHARGE` |
| RETURN_ORDERS.total_refund_amount | Giữ | Σ `refund_amount` (không âm) |

### 3.6. Không cần

| Thứ | Lý do |
|---|---|
| `PROMOTION_USAGES` | Lượt dùng đếm từ `ORDERS.order_promotion_id` và `ORDER_DETAILS.line_promotion_id` của đơn `COMPLETED` |
| `is_stackable`, `priority` | Thay bằng thứ tự 4 tầng cố định |
| `params JSONB` | Thay bằng cột riêng + `PROMOTION_STEPS` |

## 4. Đọc dữ liệu: muốn biết gì thì xem cột nào

| Câu hỏi | Xem |
|---|---|
| KM này là loại gì | `PROMOTIONS.promotion_type` |
| Dòng này được KM gì ở tầng 1 | `ORDER_DETAILS.line_promotion_id` (NULL = không có) |
| Dòng này là quà hay hàng bán | `ORDER_DETAILS.line_type` |
| Quà này từ KM nào | `line_promotion_id` của dòng quà → `promotion_type` là `BUY_X_GET_Y` hay `ORDER_GIFT` |
| Những dòng nào cùng một combo / một bộ mua-tặng | Cùng `line_promotion_id` **và** cùng `promo_group_no` |
| Đơn được giảm cấp đơn bằng gì | `ORDERS.order_promotion_id` → `promotion_type` là `ORDER_TIERED` hay `VOUCHER` |
| KM đã dùng bao nhiêu đơn | `COUNT(DISTINCT order_id)` đơn `COMPLETED` có `order_promotion_id = X` hoặc có dòng `line_promotion_id = X` |
| Giờ vàng đã bán bao nhiêu | `SUM(quantity)` dòng `line_promotion_id = X` của đơn `COMPLETED` |
| Tiền giảm từng tầng | 4 cột giảm ở ORDER_DETAILS (từng dòng) hoặc ORDERS (cả đơn) |

## 5. Lúc bán (POS-02, POS-03)

1. Service nạp: sản phẩm trong giỏ, KM `ACTIVE` còn hạn (kèm bảng con và số lượt đã dùng), khách, cấu hình điểm.
2. Gọi `calculateOrder(...)` ([tinh-gia.ts](mau-code/khuyen-mai/tinh-gia.ts)). Hàm trả về các dòng đã tính giá, tổng tiền và thông báo cho thu ngân (voucher không được áp, quà hết...).
3. Chọn lô FEFO cho từng dòng (kể cả dòng quà), tách bằng `splitByBatches`.
4. Trong **một transaction** khi hoàn tất: kiểm tra lại tồn và lượt dùng, ghi ORDERS + ORDER_DETAILS, trừ lô, ghi STOCK_MOVEMENTS `SALE` (dòng quà cũng vậy), điểm.

## 6. Lúc trả (POS-04)

**Nguyên tắc: không tính lại khuyến mãi.** Gọi `calculateReturn(...)` ([tra-hang.ts](mau-code/khuyen-mai/tra-hang.ts)):

| Bước | Lấy từ đâu | Kết quả |
|---|---|---|
| 1. Tiền từng dòng trả | `line_total × qty_trả / quantity` (lần trả cuối của dòng nhận phần còn lại) | Dòng `RETURN` |
| 2a. Mua X tặng Y còn đủ điều kiện không | Nhóm (`line_promotion_id`, `promo_group_no`); đếm số món "mua" còn giữ so với `buy_quantity` | Quà dư mà không trả lại → dòng `GIFT_CHARGE` |
| 2b. Quà theo hóa đơn còn đủ ngưỡng không | Σ (`qty × unit_price − line_discount − tier_discount`) phần còn giữ so với `min_order_value` | Như trên |
| 3. Thu hồi điểm đã tích | `floor(điểm đã tích × tiền hoàn / total_amount)` | `points_revoked` |

- Giá trị quà khi trừ: giá niêm yết; quà không bán (`is_sellable = FALSE`) lấy giá vốn. Khách mang quà trả lại thì không trừ.
- Giảm cấp đơn (bậc thang, voucher) **không tính lại** khi đơn còn lại xuống dưới ngưỡng; không trả lại lượt voucher.
- Dòng `GIFT_CHARGE` được tính vào "đã xử lý" của dòng quà, nên lần trả sau không trừ lần nữa.
- Combo: code đang **cho trả lẻ từng món** theo giá đã phân bổ (chưa chốt, xem mục 8).
- Đổi ý: cùng cách tính; số tiền là giá trị quy đổi sang đơn đổi hàng, không chi tiền. Đơn đổi phải có tổng ≥ giá trị quy đổi (thiếu thì chặn).
- **Không phát voucher bù** (kể cả hàng lỗi); điểm đã dùng không hoàn.

## 7. Ví dụ (đều có trong test.ts)

Giả định: Serum A 400k; Kem B 300k (KM1 giảm 20%); combo Toner C 200k + Kem D 250k = 380k; KM3 mua 2 B tặng mini B (80k); hạng Vàng 5%; bậc thang ≥500k −50k, ≥1tr −120k; `THU10` −10% tối đa 50k (đơn ≥300k); `GIAM100` −100k (đơn ≥800k); đơn ≥800k tặng tote (không bán, giá vốn 15k).

| # | Tình huống | Kết quả |
|---|---|---|
| 1 | Vàng mua A + B, nhập `THU10` | Bậc thang 50k = `THU10` 50k → lấy bậc thang, mã không bị trừ lượt. A 348.750, B 209.250, tổng 558.000 |
| 2 | Vãng lai 2A + B, nhập `GIAM100` | Bậc thang 120k > 100k → lấy bậc thang. Tặng tote. Tổng 920.000 |
| 3 | Mua 2 B (có cả KM1 và KM3) | Giảm 20% (120k) > mini (80k) → giảm 20%, không tặng mini |
| 4 | Combo C + D, nhập `THU10` | Tách: C −31.111, D −38.889. `THU10` 38.000. C 152.000, D 190.000 |
| 5 | Như VD1 + dùng 500 điểm, trả B vì lỗi | Hoàn 190.500; không voucher bù; thu hồi 18 điểm |
| 6 | Từ VD2, trả 1 A vì lỗi, không trả tote | Đơn còn 640k < 800k → trừ tote 15.000. Hoàn 353.846 − 15.000 = 338.846 |
| 7 | Chỉ có KM3, mua 2 B, trả 1 B, không trả mini | Mất điều kiện → trừ 80.000. Hoàn 220.000 |

## 8. Chưa chốt

| # | Câu hỏi | Code mẫu đang làm |
|---|---|---|
| 1 | Combo cho trả lẻ hay chỉ nguyên bộ? | Cho trả lẻ theo giá đã phân bổ |
| 2 | Hạng thẻ có áp lên món đã giảm tầng 1 không? | Có |
| 3 | Giá niêm yết đã gồm VAT? | Đã gồm, tách ngược |
| 4 | Chọn KM tầng 1 theo cách "lần lượt lấy KM lợi nhất" có thể không tối ưu tuyệt đối khi nhiều combo chồng nhau. Chấp nhận? | Có (dễ giải thích, dễ test) |
