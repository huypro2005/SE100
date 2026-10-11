# ĐÁNH GIÁ USE CASE & DATABASE — HỆ THỐNG QUẢN LÝ CỬA HÀNG MỸ PHẨM (SE100)

> Tài liệu đánh giá bản `USECASE-SE100.pdf` (81 trang, 7+1 nhóm UC, 30 bảng) và đề xuất phương án chỉnh sửa.
> Bối cảnh đã chốt với nhóm: **5 người, tài liệu dùng để code trong học kỳ**; **bỏ role SALES**; **bỏ công nợ NCC**; **khuyến mãi = voucher cấp đơn + điều kiện hạng thẻ tối thiểu**.
> Giả định (chưa hỏi, cần xác nhận ở mục 8): hệ thống chạy cho **một cửa hàng** (không đa chi nhánh).

---

## 0. Tóm tắt nhanh

**Kết luận chung:** Nghiệp vụ cốt lõi (lô – hạn dùng – FEFO, kiểm kho có snapshot, cách ly hàng lỗi bằng `blocked_qty`, chốt ca, lưu giá vốn COGS theo dòng bán) được thiết kế **tốt và có chiều sâu**, đây là điểm mạnh nên giữ. Tuy nhiên tài liệu đang có 3 vấn đề lớn:

1. **Không nhất quán** giữa ma trận phân quyền ↔ đặc tả ↔ database (mã UC lệch, actor lệch, form có trường nhưng DB không có cột, DB có cột nhưng không UC nào dùng).
2. **Thiếu UC quan trọng**: Mở ca, toàn bộ Xuất hủy (có 3 bảng DB nhưng không có đặc tả), Nhập kho chưa gắn NCC, Dùng điểm thưởng, Xem Audit Log, Cấu hình hệ thống; thiếu bảng **chi tiết phiếu nhập**.
3. **Độ chi tiết UC không đồng đều**: PROD gộp CRUD vào 1 UC, trong khi SUPP/CUS/PROM tách “xem danh sách / tìm kiếm / xem chi tiết” thành 3 UC riêng; REP tách “lọc theo thời gian”, “xuất file” thành UC dù đó là chức năng chung.

**Quy mô:** bản hiện tại 58 UC + 30 bảng — hơi lớn cho 5 người code trong 1 kỳ, nhưng phần lớn là do tách vụn. Phương án đề xuất: **40 UC (39 UC người dùng + 1 tác vụ tự động), 30 bảng** (bỏ 3 bảng, thêm 3 bảng, bổ sung cột còn thiếu), giữ nguyên toàn bộ nghiệp vụ có giá trị.

| Nhóm | Hiện tại | Đề xuất | Thay đổi chính |
|---|---|---|---|
| PROD | 3 | 3 | Bổ sung trường còn thiếu (giá bán, VAT, ngưỡng tồn…) vào đặc tả |
| STK | 5 (đánh số 04–08) | 7 | Đánh số lại 01–07, **thêm 2 UC xuất hủy**, nhập kho gắn NCC |
| SUPP | 9 | 3 | Gộp xem/tìm/chi tiết; bỏ công nợ |
| CUS | 9 | 5 | Gộp; **thêm UC dùng điểm** |
| PROM | 7 | 4 | Gộp tạo/sửa/tạm dừng; bỏ đối tượng tuổi & theo sản phẩm |
| POS | 5 | 5 | **Thêm Mở ca**; gộp Đổi trả + Hoàn tiền |
| REP | 10 | 6 | Lọc/Xuất/So sánh thành chức năng chung |
| AUTH | 10 | 7 | Gộp; **thêm Audit Log, Cấu hình hệ thống** |
| **Tổng** | **58** | **40** | Ít hơn 18 UC nhưng đủ nghiệp vụ hơn |

---

## 1. Lỗi xuyên suốt toàn tài liệu

### 1.1. Mã UC không khớp giữa ma trận và đặc tả

| Vị trí | Ma trận ghi | Đặc tả thực tế |
|---|---|---|
| Nhập kho | `UC-STK-01` | `UC-STK-04` |
| Tra cứu tồn, theo dõi date | `UC-STK-02..03` | `UC-STK-05..06` |
| Kiểm kho | `UC-STK-04..05` | `UC-STK-07..08` |
| Xuất hủy | `UC-STK-06..08` | **Không có đặc tả** |
| Quản trị | `UC-AUTH-04..12` | Chỉ có đến `AUTH-10` |
| Báo cáo COGS | `UC-REP-01, 08..10` | REP-08/09/10 là Xuất file / Lọc / So sánh, không phải báo cáo COGS |
| Nhóm POS | Có trong ma trận | **Không có trong danh sách quy tắc đặt tên 1.1** |

Ngoài ra trong đặc tả có tham chiếu kiểu cũ `UC02`, `UC04`, `UC07`, `UC09` (POS-03, POS-04, STK-06) — cần thay bằng mã đầy đủ.

### 1.2. Lỗi copy–paste cần sửa

- `UC-SUPP-01` bước 4: “System trả về tokens và thông tin user” (chép từ UC đăng nhập).
- `UC-SUPP-07` tiêu đề ghi “Ngừng hợp tác nhà cung cấp” nhưng nội dung là “Xem lịch sử nhập hàng”.
- `UC-REP-07`, `UC-REP-08` tên UC ghi “Xem báo cáo khách hàng”.
- `UC-AUTH-04` tiêu đề ghi “Đổi mật khẩu” nhưng nội dung là “Quản lý tài khoản”.
- `UC-POS-04` Use Case ID ghi `UC04`.
- `UC-CUS-06` ngoại lệ 3a “đơn hàng Đang giao” và `UC-CUS-04` “đã bị xóa” — cửa hàng bán tại quầy, không có giao hàng và không xóa KH.
- Bảng 2.10 / 2.3.7 đánh số “UC-10”, “UC-07”.
- Ở PROD-03, STK-06..08 phần text bị lệch cột (Actor/Precondition đảo chỗ) khi xuất PDF — kiểm tra lại bảng gốc.

### 1.3. Actor trong đặc tả lệch với ma trận phân quyền

- **SUPP**: ma trận cho Manager “Đề xuất” thêm/sửa NCC, đặc tả chỉ có Admin; không có luồng “đề xuất”.
- **REP**: toàn bộ đặc tả ghi Actor = Admin/Owner, nhưng ma trận cho Manager xem, NV kho xem báo cáo kho, Thu ngân xem ca cá nhân.
- **POS-03/04**: ma trận cho Manager “Duyệt” đổi trả và “Duyệt hạn mức” hoàn tiền, nhưng đặc tả không có bước duyệt nào.
- **POS-05**: ma trận “Ký nhận bàn giao” cho Manager, đặc tả không có.
- **STK-08**: actor “Quản lý kho” — không có role này (chỉ có STORE_MANAGER).
- Role **SALES** có mô tả nhưng không có cột ma trận, không có UC → **đã chốt bỏ**.

### 1.4. Nguyên tắc độ chi tiết UC đề xuất (áp dụng thống nhất)

| Loại | Quy tắc | Ví dụ |
|---|---|---|
| Dữ liệu danh mục (master data) | **1 UC “Quản lý X”** gồm thêm / sửa / ngừng / khôi phục làm luồng phụ | Quản lý NCC, Quản lý sản phẩm |
| Tra cứu | **1 UC “Tra cứu X”** gồm danh sách + tìm kiếm/lọc + xem chi tiết | Tra cứu khách hàng |
| Nghiệp vụ có trạng thái | **Mỗi bước đổi trạng thái chứng từ = 1 UC** | Lập phiếu kiểm → Duyệt điều chỉnh |
| Chức năng dùng chung (lọc thời gian, xuất Excel, phân trang) | **Không phải UC**, ghi vào mục “Quy tắc chung” hoặc làm `<<extend>>` | Lọc báo cáo, Xuất báo cáo |
| Tác vụ tự động | Actor = **Hệ thống**, là `<<include>>` của UC kích hoạt | Tích điểm khi hóa đơn hoàn tất |

Phân quyền chi tiết (ai được khóa, ai được sửa) xử lý bằng **permission theo hành động** (vd `CUSTOMER_CREATE`, `CUSTOMER_LOCK`), không cần tách UC.

---

## 2. Đánh giá từng nhóm

### 2.1. UC-PROD — Danh mục, Thương hiệu, Sản phẩm

**Đánh giá UC:** Độ chi tiết hợp lý (1 UC quản lý cho mỗi loại master data), luồng ngoại lệ tốt (không cho đổi đơn vị tính khi đã có giao dịch, chỉ xóa khi chưa tham chiếu).

**Vấn đề:**
- **PROD-01 bước 3 thiếu hẳn giá bán** — form chỉ có SKU, tên, danh mục, thương hiệu, quy cách, ĐVT, mã vạch. Trong khi DB có `retail_price`, `tax_rate`, `skin_type`, `min_stock_alert`. Thiếu giá bán thì POS không chạy được.
- “Quy cách” (vd 50ml) có trong form nhưng **DB không có cột**.
- Danh mục/thương hiệu có “mô tả” trong form nhưng DB không có cột `description`.
- Chưa nói khi **đổi giá bán** thì đơn cũ có bị ảnh hưởng không (thực tế không, vì `order_details.unit_price` đã lưu giá tại thời điểm bán — nên ghi rõ).
- PROD-01 thiếu luồng **khôi phục** sản phẩm đã ngừng kinh doanh.

**Đánh giá DB:**

| Bảng | Nhận xét | Đề xuất |
|---|---|---|
| CATEGORIES | Thiếu `description`. Danh mục phẳng là đủ (không cần cây cha–con) | + `description` |
| BRANDS | Thiếu `description` | + `description`, + `origin_country` (tùy chọn) |
| PRODUCTS | Tốt. Thiếu quy cách, mô tả, ảnh, thời điểm tạo/sửa | + `volume` (VARCHAR, “50ml”), + `description`, + `image_url`, + `created_at`, `updated_at` |

**Phương án:** Giữ 3 UC. Sửa form PROD-01: thêm Giá bán (*), Thuế suất VAT, Loại da phù hợp, Ngưỡng tồn tối thiểu, Quy cách, Ảnh. Thêm luồng 1c “Khôi phục kinh doanh”.

---

### 2.2. UC-STK — Nghiệp vụ kho

**Đánh giá UC:** Đây là phần **mạnh nhất** của tài liệu: phiếu nhập có nháp, chống ghi nhận 2 lần, kiểm kho có snapshot + khóa phạm vi, duyệt điều chỉnh theo **chênh lệch cộng vào tồn hiện tại** (đúng, vì sau khi mở khóa vẫn có thể bán trước khi duyệt).

**Vấn đề:**
1. **Thiếu đặc tả xuất hủy** (ma trận có `STK-06..08`, DB có 3 bảng DISPOSAL, STK-06 và POS-03 đều tham chiếu “UC09”) — đây là lỗ hổng lớn nhất nhóm kho.
2. **Nhập kho (STK-04) không chọn Nhà cung cấp** → không nối được với SUPP (lịch sử nhập hàng của NCC, giá nhập gần nhất).
3. **Không có bảng chi tiết phiếu nhập**: hiện chỉ có `PURCHASE_ORDERS` (header). Không biết phiếu nhập gồm những lô nào, giá nhập bao nhiêu → SUPP-07, SUPP-08, REP-04 không làm được.
4. **STK-04 bước 6 “dùng lô có thông tin khớp”**: nếu cùng lô nhập 2 lần với 2 giá khác nhau thì `BATCHES.cost_price` bị ghi đè → sai COGS.
5. **Quy tắc ngày hết hạn mâu thuẫn**: STK-06 3b nói “hàng còn dùng được hết ngày ghi trên hạn”, còn Business Rule 2 loại lô `exp_date <= CurrentDate` (tức ngừng bán ngay trong ngày hết hạn).
6. **Cơ chế khóa kiểm kho chưa có chỗ lưu**: `INVENTORY_CHECKS` không có phạm vi kiểm, `BATCHES` không có cờ khóa → POS không biết lô nào đang bị khóa.
7. **STK-07 bước 3a “lưu tạm, ô trống không hiểu là 0”** nhưng `counted_qty` là `NOT NULL`.
8. **STK-08 3b “Yêu cầu làm rõ/kiểm lại”** và **STK-07 6a “Hủy phiên”**, **7b “Hoàn tất không chênh lệch”** — các trạng thái này không có trong enum `status`.
9. Ngưỡng cảnh báo cận hạn **N ngày** lưu ở đâu? Không có bảng cấu hình (REP-03 lại dùng 30/60 ngày cứng).
10. `DISPOSAL_ALLOCATIONS` (from_blocked / from_unblocked) **quá phức tạp** cho bài toán chống trừ trùng — có thể giải bằng 1 cột (xem dưới).

**Đánh giá DB và đề xuất:**

| Bảng | Đề xuất |
|---|---|
| BATCHES | + `UNIQUE(product_id, lot_code)`; + `locked_by_check_id` (FK NULL → INVENTORY_CHECKS) để POS kiểm tra khóa nhanh; + `created_at`. `cost_price` = **giá vốn bình quân gia quyền** khi nhập thêm vào lô đã có: `(on_hand×cost_cũ + qty×cost_mới)/(on_hand+qty)` |
| STOCK_MOVEMENTS | + `created_by` (FK USERS); + `balance_after` (tồn sau biến động — để hiện thẻ kho). Chuẩn hóa `reference_type`: `PURCHASE`, `SALE`, `RETURN`, `CHECK_ADJUST`, `DISPOSAL`. Index `(batch_id, created_at)` |
| PURCHASE_ORDERS | Đổi nghĩa thành **Phiếu nhập kho** (có thể đổi tên `GOODS_RECEIPTS`). Bỏ `paid_amount`, `debt_amount`. Status: `DRAFT`, `COMPLETED`, `CANCELLED` (default `DRAFT`, không phải `COMPLETED`). + `supplier_invoice_no` (số HĐ của NCC, NULL), + `note`, + `confirmed_at` |
| **PURCHASE_ORDER_DETAILS (mới)** | `po_detail_id`, `po_id`, `product_id`, `lot_code`, `mfg_date`, `exp_date`, `quantity`, `unit_cost`, `line_total`, `batch_id` (NULL khi còn nháp, gán khi xác nhận) |
| INVENTORY_CHECKS | Status: `IN_PROGRESS`, `WAITING_APPROVAL`, `APPROVED`, `REJECTED`, `CANCELLED`, `NO_DIFFERENCE`. + `scope_note` (mô tả phạm vi; phạm vi thực tế = tập lô trong DETAILS), + `submitted_at`, `approved_at`, `reject_reason`. Bỏ trạng thái “cần bổ sung”: từ chối rồi lập phiếu mới |
| INVENTORY_CHECK_DETAILS | `counted_qty` cho phép NULL khi `IN_PROGRESS`, bắt buộc khi gửi duyệt; `difference_qty` NULL đến khi gửi duyệt |
| DISPOSAL_REQUESTS | Status: `WAITING_APPROVAL`, `APPROVED`, `REJECTED` (bỏ `APPROVED_PENDING` / `COMPLETED` — duyệt là trừ tồn luôn, vì hàng đã được cách ly). + `approved_at`, `reject_reason` |
| DISPOSAL_DETAILS | + `blocked_added_qty` (lượng hệ thống đã cách ly thêm khi lập phiếu), + `unit_cost` (chụp giá vốn khi duyệt — phục vụ báo cáo tổn thất) |
| ~~DISPOSAL_ALLOCATIONS~~ | **Bỏ**, thay bằng quy tắc dưới |

**Quy tắc xuất hủy đơn giản hóa (thay DISPOSAL_ALLOCATIONS):**
- Khi **lập đề nghị** cho lô X với số lượng `q`:
  `pending = Σ quantity của các phiếu WAITING_APPROVAL khác trên lô X`
  `free_blocked = blocked_qty − pending`
  Nếu `q > free_blocked` → `blocked_qty += (q − free_blocked)` và ghi `blocked_added_qty = q − free_blocked`. Chặn nếu `q > on_hand_qty − pending`.
- Khi **duyệt**: `on_hand_qty −= q`, `blocked_qty −= q`, ghi STOCK_MOVEMENTS `DISPOSAL (−q)`.
- Khi **từ chối**: `blocked_qty −= blocked_added_qty` (trả lại phần đã cách ly thêm).

**Quy tắc ngày hết hạn (đề xuất thống nhất):** Lô được bán khi `exp_date >= hôm nay` (còn dùng hết ngày ghi trên bao bì). Cận hạn khi `hôm nay <= exp_date <= hôm nay + N`. Hết hạn khi `exp_date < hôm nay`. *(Cần nhóm xác nhận — mục 8.)*

**Danh sách UC-STK đề xuất:**

| Mã mới | Tên | Actor | Ghi chú |
|---|---|---|---|
| STK-01 | Lập phiếu nhập kho theo lô | NV kho | = STK-04 cũ + **chọn NCC (chỉ NCC ACTIVE)**, lưu chi tiết vào PURCHASE_ORDER_DETAILS |
| STK-02 | Tra cứu tồn kho theo sản phẩm & lô (thẻ kho) | NV kho, Manager, Thu ngân (chỉ xem tồn khả dụng) | = STK-05 cũ |
| STK-03 | Theo dõi hàng cận hạn / hết hạn | NV kho, Manager | = STK-06 cũ, N lấy từ SYSTEM_SETTINGS |
| STK-04 | Lập phiếu kiểm kho | NV kho | = STK-07 cũ; bắt đầu kiểm → gán `locked_by_check_id`; gửi duyệt/hủy → gỡ khóa |
| STK-05 | Duyệt điều chỉnh sau kiểm kho | Manager | = STK-08 cũ |
| STK-06 | Lập phiếu xuất kho khác (hủy / tester / trả NCC) | NV kho | **Mới viết đặc tả**. Chọn `issue_type`: `DESTROY`, `TESTER`, `RETURN_TO_SUPPLIER`. Điền sẵn từ STK-03 hoặc hàng trả hỏng (POS). Phiếu `TESTER` ưu tiên lô cận hạn nhất |
| STK-07 | Duyệt phiếu xuất kho khác | Manager | **Mới viết đặc tả**. Duyệt = trừ tồn, mọi loại phiếu (kể cả tester) đều cần Manager duyệt |

---

### 2.3. UC-SUPP — Nhà cung cấp

**Đánh giá UC:** Đặc tả viết kỹ phần giao diện, nhưng **tách quá vụn**: 01 (danh sách), 04 (chi tiết), 05 (tìm kiếm), 07 (lịch sử nhập), 08 (sản phẩm cung ứng) thực chất là **một màn hình tra cứu có tab**. 02/03/06/09 là CRUD + ngừng/khôi phục của cùng 1 đối tượng.

**Vấn đề:**
- SUPP-08 “Giá nhập thỏa thuận gần nhất” và “NCC chưa được gán sản phẩm phân phối” → ngụ ý có bảng NCC–Sản phẩm, nhưng DB không có. **Đề xuất: không cần bảng gán**, “sản phẩm cung ứng” = các sản phẩm đã từng có trong phiếu nhập của NCC, giá gần nhất lấy từ `PURCHASE_ORDER_DETAILS` mới nhất.
- SUPP-06 3a chặn khi còn đơn “Chờ giao/Đang xử lý” → không khớp trạng thái phiếu nhập. Sửa thành: còn phiếu nhập `DRAFT` thì cảnh báo.
- Form SUPP-02 có **Địa chỉ (*), Người liên hệ, Ghi chú chính sách chiết khấu** nhưng DB không có cột. 4a kiểm trùng SĐT nhưng DB `phone_number` không UNIQUE.
- Admin-only thêm/sửa NCC là quá chặt cho cửa hàng nhỏ, và ma trận đang lửng lơ “Manager: Đề xuất”. Đề xuất cho **Manager toàn quyền NCC**, NV kho chỉ xem.

**DB:**

| Bảng | Đề xuất |
|---|---|
| SUPPLIERS | Bỏ `debt_limit`. + `address` (NOT NULL), `contact_person`, `note`, `created_at`, `updated_at`. `phone_number` UNIQUE |
| PURCHASE_ORDERS / DETAILS | Như mục 2.2 |

**Danh sách UC-SUPP đề xuất:**

| Mã mới | Tên | Actor | Gộp từ |
|---|---|---|---|
| SUPP-01 | Quản lý nhà cung cấp (thêm, sửa, ngừng hợp tác, khôi phục) | Admin, Manager | 02, 03, 06, 09 |
| SUPP-02 | Tra cứu & xem chi tiết NCC (danh sách, tìm kiếm, thông tin, tab sản phẩm cung ứng) | Admin, Manager, NV kho | 01, 04, 05, 08 |
| SUPP-03 | Xem lịch sử nhập hàng của NCC (danh sách phiếu → chi tiết lô) | Admin, Manager, NV kho | 07 |

---

### 2.4. UC-CUS — Khách hàng, hạng thẻ, tích điểm

**Đánh giá UC:** CUS-08 (tích điểm tự động) và CUS-09 (cấu hình, kiểm tra mốc hạng tăng dần) là ý tốt. Phần CRUD tách vụn như SUPP.

**Vấn đề:**
1. **Tích điểm nhưng không có chỗ dùng điểm.** CUS-08 nói cộng vào “Điểm khả dụng”, nhưng không UC nào cho đổi điểm → điểm vô nghĩa (hạng thẻ đã tính theo `total_spent`, không theo điểm). **Đề xuất thêm UC dùng điểm khi thanh toán.**
2. **Chiết khấu hạng thẻ** (`CUSTOMER_TIERS.discount_rate`) được nhắc ở vai trò Thu ngân nhưng không UC nào mô tả **khi nào áp, áp trước hay sau voucher**.
3. **Tỷ lệ quy đổi điểm (CUS-09) lưu ở đâu?** Không có bảng. “Lịch sử thay đổi cấu hình”, “lịch sử nâng hạng” (CUS-08 bước 5) cũng không có chỗ lưu.
4. Thu hồi điểm khi **trả một phần** chưa có công thức. Có **hạ hạng** khi trả hàng không? Chưa nói.
5. “Khóa tài khoản khách hàng” — khách không có tài khoản đăng nhập. Nên gọi là **“Khóa/Mở khóa ưu đãi khách hàng”**.
6. Form CUS-02 có Email, Ngày sinh, Giới tính, Địa chỉ, Ghi chú → DB thiếu toàn bộ. REP-06 cần “khách mới theo kỳ” → DB thiếu `created_at`.
7. Precondition CUS-08: “đơn bị **Hủy** thì thu hồi điểm” — đơn chỉ được tích điểm khi COMPLETED, mà COMPLETED thì không hủy được nữa (chỉ trả hàng). Bỏ chữ “Hủy”.

**DB:**

| Bảng | Đề xuất |
|---|---|
| CUSTOMERS | + `email`, `date_of_birth`, `gender`, `address`, `note`, `created_at`, `updated_at`. Giữ `skin_type` (NULL) làm thông tin hồ sơ |
| CUSTOMER_TIERS | + `tier_code` (BRONZE…), `tier_name` hiển thị (Đồng…), `min_spend` UNIQUE. Hạng mặc định = hạng có `min_spend = 0` |
| POINT_HISTORIES | + `return_id` (FK NULL), + `balance_after`. `action_type`: `EARN`, `REDEEM`, `REVOKE` (thu hồi khi trả hàng), `ADJUST` |
| Lịch sử nâng hạng / đổi cấu hình | **Không tạo bảng riêng**, ghi vào `AUDIT_LOGS` (action `TIER_UPGRADE`, `UPDATE_LOYALTY_CONFIG`) |
| Tỷ lệ quy đổi | Lưu vào **SYSTEM_SETTINGS** (mới, xem 2.8): `POINT_EARN_AMOUNT` (VNĐ/1 điểm), `POINT_REDEEM_VALUE` (1 điểm = ? VNĐ), `POINT_MAX_REDEEM_PERCENT` |

**Quy tắc đề xuất:**
- Điểm cộng = `floor(số tiền khách thực trả / POINT_EARN_AMOUNT)` khi đơn COMPLETED và KH ACTIVE.
- Trả hàng: điểm thu hồi = `floor(điểm đã cộng của đơn × tiền hoàn / tiền khách thực trả)`; `total_spent −= tiền hoàn`. Nếu điểm hiện có không đủ thì về 0 (không âm).
- **Không tự hạ hạng** (đơn giản, đúng với đa số cửa hàng). *(Cần xác nhận — mục 8.)*

**Danh sách UC-CUS đề xuất:**

| Mã mới | Tên | Actor | Gộp từ |
|---|---|---|---|
| CUS-01 | Tra cứu & xem hồ sơ khách hàng (lịch sử mua, lịch sử điểm) | Admin, Manager, Thu ngân | 01, 04, 05 |
| CUS-02 | Quản lý hồ sơ khách hàng (thêm — kể cả tạo nhanh tại POS bằng SĐT, sửa, khóa/mở khóa ưu đãi) | Thu ngân (thêm/sửa), Admin/Manager (khóa) | 02, 03, 06, 07 |
| CUS-03 | Tích điểm, thu hồi điểm & xét hạng tự động | Hệ thống | 08 |
| CUS-04 | **Dùng điểm thưởng khi thanh toán** | Thu ngân | **Mới**, `<<extend>>` của POS-03 |
| CUS-05 | Cấu hình tích điểm & hạng thẻ | Admin, Manager | 09 |

---

### 2.5. UC-PROM — Khuyến mãi, Voucher

**Đánh giá UC:** PROM-06 có bộ 4 rule kiểm tra rõ ràng, PROM-03 3a (khóa sửa mức giảm khi đã có người dùng) rất tốt.

**Vấn đề:**
1. Form PROM-02 có **“Đối tượng áp dụng (tuổi, hạng)”** và **“Sản phẩm/Danh mục áp dụng”** nhưng DB không có → **đã chốt**: bỏ tuổi và theo sản phẩm, giữ **hạng thẻ tối thiểu**.
2. Trạng thái lệch: đặc tả có `PENDING`, “Sắp diễn ra”, “Đã kết thúc”; DB có `ACTIVE/PAUSED/EXPIRED`. Nếu lưu `EXPIRED` thì phải có job chạy hàng ngày để cập nhật.
3. `PROMOTION_USAGES.customer_id NOT NULL` → **khách vãng lai không dùng được voucher nào**; nhưng Rule 4 lại chỉ có nghĩa khi có khách.
4. `is_stackable` có trong DB nhưng POS chỉ có 1 ô nhập mã và `ORDERS` không có chỗ ghi mã nào được áp → **bỏ, quy định 1 voucher/đơn**.
5. Ghi nhận lượt dùng **lúc nào**? Nếu ghi lúc “Áp dụng” mà đơn bị hủy thì mất lượt. Cần ghi lúc đơn COMPLETED và kiểm tra lại Rule 3, 4 tại thời điểm đó (tránh 2 quầy cùng dùng lượt cuối).
6. Trả hàng có trả lại lượt voucher không? Nếu trả một phần làm đơn xuống dưới `min_order_value` thì sao? Chưa nói.
7. PROM-01 và PROM-05 trùng nhau (danh sách + tìm kiếm).
8. PROM-07 (báo cáo hiệu quả) liệt kê 7 chỉ số + biểu đồ + top khách — nhiều. “Usage Rate” chưa định nghĩa.

**DB:**

| Bảng | Đề xuất |
|---|---|
| PROMOTIONS | Bỏ `is_stackable`. + `min_tier_id` (FK NULL → CUSTOMER_TIERS), `description`, `created_by`, `created_at`, `updated_at`. `usage_limit` và `limit_per_customer` cho phép NULL = không giới hạn. `status` chỉ lưu `ACTIVE` / `PAUSED` (do người dùng bật/tắt); “Sắp diễn ra / Đang chạy / Đã kết thúc” **tính từ ngày** khi hiển thị → không cần job |
| ~~PROMOTION_USAGES~~ | **Bỏ**. Vì 1 voucher/đơn: đưa `promotion_id` vào `ORDERS`. Lượt đã dùng = `COUNT(orders COMPLETED có promotion_id)`; theo khách = thêm điều kiện `customer_id` |

**Quy tắc đề xuất:**
- Voucher có `limit_per_customer` hoặc `min_tier_id` → **bắt buộc chọn khách hàng**. Voucher không có 2 điều kiện đó → khách vãng lai dùng được.
- Thứ tự giảm giá: **(1) chiết khấu hạng thẻ** trên tiền hàng → **(2) voucher** trên phần còn lại → **(3) điểm thưởng** trên phần còn lại. `min_order_value` so với tiền hàng sau bước (1).
- Trả hàng **không hoàn lại lượt voucher**; tiền hoàn tính theo giá đã phân bổ giảm (xem POS).

**Danh sách UC-PROM đề xuất:**

| Mã mới | Tên | Actor | Gộp từ |
|---|---|---|---|
| PROM-01 | Tra cứu chương trình khuyến mãi | Admin, Manager, Thu ngân | 01, 05 |
| PROM-02 | Quản lý chương trình khuyến mãi (tạo, sửa, tạm dừng, kích hoạt lại) | Admin, Manager | 02, 03, 04 |
| PROM-03 | Áp dụng voucher khi bán hàng | Thu ngân | 06, `<<extend>>` của POS-02 |
| PROM-04 | Xem hiệu quả chương trình khuyến mãi | Admin, Manager | 07 — rút gọn: số lượt dùng, tổng tiền giảm, doanh thu đơn có mã, số khách dùng, danh sách hóa đơn |

---

### 2.6. UC-POS — Bán hàng tại quầy, Đổi trả, Ca

**Đánh giá UC:** Rất tốt ở các điểm: FEFO tự tách dòng khi 1 lô không đủ (3a), đổi lô thủ công (3b), kiểm tra lại lô trước khi xuất hóa đơn (4a), chống trừ tồn 2 lần (4b), rollback (5a), hàng trả hỏng → tăng `blocked_qty` để FEFO không bán lại, chặn chốt ca khi còn chứng từ dở dang.

**Vấn đề:**
1. **Thiếu UC Mở ca** — là precondition của POS-01 nhưng không có đặc tả (nhập tiền đầu ca, chặn mở ca thứ 2).
2. **Nhóm POS không có trong danh sách nhóm UC (mục 1.1)** — cần bổ sung `UC-POS-xx`.
3. `ORDERS.payment_method NOT NULL` nhưng đơn được tạo ở POS-01 **trước khi** chọn phương thức thanh toán (POS-02).
4. Lưu nháp (POS-01 2a) không có trạng thái riêng trong enum; `created_at` mô tả là “thời điểm xuất hóa đơn” nhưng thực ra là lúc lập đơn → thiếu `completed_at` (báo cáo doanh thu phải theo ngày hoàn tất).
5. Không lưu **tiền khách đưa / tiền thừa** (`change_due` nhắc ở POS-02 nhưng không có cột).
6. `discount_amount` gộp chung voucher + hạng thẻ (+ điểm) → báo cáo khuyến mãi không tách được; không biết đơn dùng voucher nào.
7. **Hoàn tiền “trừ đi phần chiết khấu đã hưởng” (POS-04 bước 2) không tính được**, vì giảm giá nằm ở cấp đơn, `ORDER_DETAILS` không có phần giảm giá phân bổ cho từng dòng.
8. **`RETURN_ORDER_DETAILS` không liên kết `order_detail_id`** → không kiểm được “số lượng trả không vượt số còn lại trên đơn gốc” (POS-03 bước 4) khi cùng lô xuất hiện ở 2 dòng.
9. Tách **POS-03 (nhận hàng trả) và POS-04 (hoàn tiền)** tạo thêm trạng thái `WAITING_FOR_REFUND` và các ca ngoại lệ (trả xong chưa hoàn, chặn chốt ca…), trong khi thực tế hai việc này diễn ra liền nhau tại quầy, cùng 1 người. Bước “Manager duyệt” trong ma trận thì đặc tả không có.
10. POS-04 4a “nộp thêm tiền mặt vào ca” → không có UC/bảng thu–chi ngoài bán hàng. Đề xuất bỏ: không đủ tiền mặt thì hoàn bằng chuyển khoản.
11. Trả hàng không nói đến **thu hồi điểm** (chỉ CUS-08 1a nói) và **giá vốn của hàng trả** (để trừ COGS).
12. `salesperson_id` không có FK → **bỏ** theo quyết định bỏ SALES.
13. Thuế: hiện thiết kế là `total = subtotal − discount + tax` (giá niêm yết **chưa** VAT). Bán lẻ mỹ phẩm ở VN thường niêm yết giá **đã gồm** VAT → *(cần xác nhận — mục 8)*.
14. Chỉ một phương thức thanh toán/đơn (không chia tiền mặt + thẻ) — chấp nhận được nhưng nên ghi rõ là quy tắc.

**DB:**

| Bảng | Đề xuất |
|---|---|
| CASHIER_SHIFTS | + `shift_code`. Đổi tên `expected_cash`/`actual_cash` cho khớp đặc tả (`expected_closing_cash`, `actual_closing_cash`) hoặc ngược lại — chọn một. + `handover_confirmed_by` (FK NULL, Manager xác nhận bàn giao — tùy chọn). Ràng buộc: mỗi thu ngân tối đa 1 ca `OPEN` (kiểm tra ở tầng ứng dụng) |
| ORDERS | Bỏ `salesperson_id`. `payment_method` cho phép NULL đến khi hoàn tất. Status: `DRAFT` (đang lập/tạm giữ, chưa trừ tồn), `COMPLETED`, `CANCELLED`. + `promotion_id` (FK NULL), tách giảm giá: `tier_discount_amount`, `voucher_discount_amount`, `points_used`, `points_discount_amount` (giữ `discount_amount` = tổng). + `amount_received`, `change_due`, `completed_at` |
| ORDER_DETAILS | + `tax_rate` (chụp tại thời điểm bán), + `discount_allocated` (phần giảm cấp đơn phân bổ theo tỷ lệ tiền dòng), + `line_total` (= qty×unit_price − discount_allocated) |
| RETURN_ORDERS | Status: `COMPLETED`, `CANCELLED` (gộp nhận hàng + hoàn tiền thành 1 giao dịch). + `restocking_fee_amount`, `points_revoked`. Có `shift_id` của **ca hiện tại** (không phải ca của đơn gốc) |
| RETURN_ORDER_DETAILS | **+ `order_detail_id` (FK, bắt buộc)**; `batch_id` lấy theo dòng gốc. `refund_unit_price` = `line_total / quantity` của dòng gốc. + `unit_cost` (= `cost_price` dòng gốc, để trừ COGS) |
| RETURN_POLICIES | Giữ. + `is_default`, `status`. Quản lý trong UC Cấu hình hệ thống |

**Danh sách UC-POS đề xuất:**

| Mã mới | Tên | Actor | Ghi chú |
|---|---|---|---|
| POS-01 | **Mở ca làm việc** | Thu ngân | **Mới**: nhập tiền đầu ca; chặn nếu đã có ca OPEN |
| POS-02 | Lập đơn bán hàng tại quầy (FEFO) | Thu ngân | = POS-01 cũ. `<<include>>` tra cứu/tạo nhanh KH; `<<extend>>` PROM-03; tự áp chiết khấu hạng |
| POS-03 | Thanh toán & xuất hóa đơn | Thu ngân | = POS-02 cũ. `<<extend>>` CUS-04 (dùng điểm); `<<include>>` CUS-03 (tích điểm). Luồng phụ: In lại hóa đơn |
| POS-04 | Đổi trả & hoàn tiền | Thu ngân | **Gộp POS-03 + POS-04 cũ** thành 1 giao dịch: chọn dòng trả + lý do → kiểm chính sách theo lý do (xem mục 2.6.1) → tính giá trị hàng trả → hoàn tiền hoặc đổi hàng → cộng tồn (hỏng thì cộng cả `blocked_qty`) + thu hồi điểm trong 1 transaction |
| POS-05 | Kiểm đếm tiền & chốt ca | Thu ngân (Manager xác nhận bàn giao — tùy chọn) | = POS-05 cũ. Chặn khi còn đơn `DRAFT` thuộc ca |

#### 2.6.1. Quy tắc đổi trả đã chốt

| Lý do (`return_reason`) | Tình trạng hàng | Xử lý |
|---|---|---|
| `DEFECTIVE` (lỗi, hư, hết hạn) | Bất kỳ | Hoàn tiền (`CASH`/`BANK_TRANSFER`) hoặc đổi hàng, không tính phí. Hàng luôn vào `blocked_qty` chờ xuất hủy hoặc trả NCC |
| `WRONG_ITEM` (giao nhầm) | Bất kỳ | Hoàn tiền hoặc đổi hàng, không tính phí. Còn nguyên seal thì về tồn bán |
| `CHANGE_OF_MIND` (đổi ý, không hợp da) | Còn nguyên seal, trong `max_return_days` | **Chỉ đổi hàng, không hoàn tiền** (`refund_method = EXCHANGE`) |
| `CHANGE_OF_MIND` | Đã mở seal | **Từ chối**, không tạo phiếu |

**Đổi hàng (`EXCHANGE`):** phiếu trả sinh một giá trị quy đổi (`total_refund_amount`, không chi tiền). Thu ngân lập đơn bán mới với `exchange_return_id`; đơn này được trừ `exchange_credit_amount`.
- Hàng mới **đắt hơn**: khách nộp phần chênh lệch.
- Hàng mới **rẻ hơn** (còn dư), khách chọn một trong hai:
  1. **Nhận voucher** giá trị cố định (`FIXED_AMOUNT`, dùng 1 lần, riêng khách đó). Mệnh giá lấy từ `EXCHANGE_VOUCHER_VALUES` sao cho lệch không quá `EXCHANGE_VOUCHER_TOLERANCE` (5.000đ, thấp hơn hoặc cao hơn) so với phần dư. Không có mệnh giá phù hợp, hoặc khách vãng lai (không có hồ sơ) thì không cấp voucher.
  2. **Mua thêm một sản phẩm** để bù phần dư.
- Điểm thưởng vẫn thu hồi theo công thức hiện hành; đơn đổi tích điểm theo số tiền khách thực nộp.

**Công thức tiền hoàn (POS-04, áp dụng cho `DEFECTIVE`/`WRONG_ITEM` hoàn tiền):**
`Tiền hoàn = Σ (line_total_gốc / qty_gốc × qty_trả) × (1 − restocking_fee_percent)`
— đã tự động trừ phần giảm giá đã hưởng nhờ `discount_allocated`.

---

### 2.7. UC-REP — Báo cáo & Dashboard

**Đánh giá UC:** Có Business Rules chung (Rule 1–4) là tốt. Nhưng:

**Vấn đề:**
1. **REP-08 (Xuất), REP-09 (Lọc thời gian), REP-10 (So sánh kỳ) không phải UC độc lập** — là chức năng có trên mọi màn hình báo cáo. Ma trận lại gán chúng là “Báo cáo Doanh thu, Lợi nhuận COGS”.
2. **Không có báo cáo lợi nhuận gộp / COGS** nào trong đặc tả, dù ma trận, DB (`cost_price`, `total_cogs`) và Admin role đều nhắc.
3. **Công thức doanh thu bị trừ chiết khấu 2 lần**: REP-01 ghi “Doanh thu ròng = Tổng tiền hóa đơn hợp lệ − chiết khấu − hoàn một phần”, nhưng `total_amount` **đã trừ** chiết khấu rồi.
4. **Rule 2 “đơn hoàn tiền toàn bộ không tính”** — đơn vẫn ở trạng thái COMPLETED (không có trạng thái REFUNDED), và trả hàng có thể xảy ra ở **kỳ sau** ngày bán. Cách đúng và đơn giản hơn: **không loại đơn**, mà **trừ tiền trả hàng vào kỳ phát sinh trả hàng** (giống kế toán “hàng bán bị trả lại”). Rule 3 thừa vì Rule 1 đã chỉ lấy COMPLETED.
5. Actor toàn bộ là Admin/Owner — lệch ma trận.
6. REP-03 dùng 30/60 ngày cứng — nên dùng N từ cấu hình. Thiếu **giá trị tồn kho** (Σ tồn × giá vốn) và **giá trị hàng hủy** (tổn thất).
7. REP-05 “dead stock — số ngày lưu kho cao” cần ngày nhập lô → BATCHES thiếu `created_at` (đã đề xuất thêm).
8. REP-06 “Retention rate” chưa định nghĩa.
9. `DAILY_SALES_REPORTS`: bảng tổng hợp sẵn không có UC/job nào sinh ra nó; khi có trả hàng muộn thì số liệu cũ bị sai. Quy mô 1 cửa hàng **truy vấn trực tiếp là đủ nhanh** → **bỏ bảng**.

**Công thức thống nhất (đề xuất):**

| Chỉ tiêu | Công thức (trong khoảng `FromDate ≤ ngày ≤ ToDate`) |
|---|---|
| Tiền hàng (gross) | Σ `subtotal_amount` các đơn COMPLETED theo `completed_at` |
| Giảm giá | Σ `discount_amount` các đơn đó (tách được: hạng / voucher / điểm) |
| Hàng bán bị trả | Σ `total_refund_amount` các phiếu trả COMPLETED theo ngày trả |
| **Doanh thu thuần** | Tiền hàng − Giảm giá − Hàng bán bị trả (*nếu giá đã gồm VAT: trừ thêm VAT để ra doanh thu chưa thuế*) |
| Giá vốn (COGS) | Σ `cost_price × quantity` dòng bán − Σ `unit_cost × quantity` dòng trả |
| **Lợi nhuận gộp** | Doanh thu thuần (chưa VAT) − COGS |

**Danh sách UC-REP đề xuất:**

| Mã mới | Tên | Actor |
|---|---|---|
| REP-01 | Xem Dashboard tổng quan (nội dung theo quyền) | Admin, Manager; NV kho thấy widget kho |
| REP-02 | Báo cáo doanh thu & lợi nhuận gộp (có chế độ so sánh kỳ trước) | Admin (lợi nhuận), Manager (doanh thu) |
| REP-03 | Báo cáo bán hàng (theo ca, thu ngân, top sản phẩm bán chạy/bán chậm) | Admin, Manager; Thu ngân xem ca của mình |
| REP-04 | Báo cáo tồn kho & hạn dùng (tồn, giá trị tồn, dưới định mức, cận hạn, hàng hủy) | Admin, Manager, NV kho |
| REP-05 | Báo cáo nhập hàng (theo NCC, thương hiệu) | Admin, Manager |
| REP-06 | Báo cáo khách hàng (KH mới, phân bổ hạng, top chi tiêu) | Admin, Manager |

**Chức năng chung mọi báo cáo** (ghi 1 lần trong mục quy tắc, không làm UC): lọc nhanh (hôm nay/tuần/tháng/quý) + tùy chọn khoảng ngày, kiểm Rule `FromDate ≤ ToDate`, xuất **Excel** (PDF là tùy chọn nếu dư thời gian).

---

### 2.8. UC-AUTH — Xác thực, RBAC, Audit Log

**Đánh giá UC:** Luồng đăng nhập có khóa tạm sau 5 lần sai, khóa tài khoản thu hồi token ngay, chặn tự khóa mình, chặn bỏ hết quyền của Admin — rất tốt.

**Vấn đề:**
1. **Tên nhóm có “Audit Log” nhưng không có UC xem Audit Log.**
2. Thiếu: **sửa thông tin nhân viên**, **đặt lại mật khẩu** cho nhân viên quên mật khẩu (Admin reset), và **cấu hình tham số hệ thống** (ngưỡng cận hạn N, chính sách đổi trả, tỷ lệ điểm, thông tin cửa hàng in trên hóa đơn) — các UC khác đang tham chiếu tới những cấu hình này mà không ai quản lý.
3. AUTH-08 “Quản lý Permission” chỉ cho xem và sửa mô tả → permission gắn với mã nguồn, nên **seed sẵn, không cần UC riêng** — gộp vào màn hình gán quyền.
4. Tách vụn: AUTH-04/05/06/09 cùng là quản lý tài khoản; AUTH-07/10 cùng là cấu hình role.
5. Khóa tạm 15 phút (3b) cần lưu số lần sai và thời điểm mở khóa → `USERS` thiếu cột.
6. Thu hồi token ngay khi khóa (AUTH-06) và xóa refresh token phía server (AUTH-02) → cần chỗ lưu token/phiên.
7. Form tạo tài khoản có **Mã nhân viên**, “Mật khẩu tạm thời” → DB thiếu `employee_code`, thiếu cờ bắt đổi mật khẩu lần đầu.
8. `ROLES.role_code` có `SALE_STAFF` → bỏ.

**DB:**

| Bảng | Đề xuất |
|---|---|
| USERS | + `employee_code` UNIQUE, `failed_login_count`, `locked_until`, `must_change_password`, `last_login_at`, `updated_at` |
| ROLES | + `is_system` (role mặc định không được xóa, Admin không bị bỏ quyền quản trị) |
| PERMISSIONS | + `permission_name` (tên hiển thị) |
| AUDIT_LOGS | + `entity_type`, `entity_id` (để lọc “lịch sử thay đổi của NCC/Sản phẩm X”). Sửa tên cột `ip_address` |
| **REFRESH_TOKENS (mới, nếu dùng JWT)** | `token_id`, `user_id`, `token_hash`, `expires_at`, `revoked_at`. Nếu dùng session server thì không cần |
| **SYSTEM_SETTINGS (mới)** | `setting_key` (PK), `setting_value`, `description`, `updated_by`, `updated_at`. Chứa: `EXPIRY_WARNING_DAYS`, `POINT_EARN_AMOUNT`, `POINT_REDEEM_VALUE`, `POINT_MAX_REDEEM_PERCENT`, `STORE_NAME`, `STORE_ADDRESS`, `STORE_PHONE` |

**Danh sách UC-AUTH đề xuất:**

| Mã mới | Tên | Actor | Gộp từ |
|---|---|---|---|
| AUTH-01 | Đăng nhập | Mọi người dùng | 01 |
| AUTH-02 | Đăng xuất | Mọi người dùng | 02 |
| AUTH-03 | Đổi mật khẩu | Mọi người dùng | 03 (+ bắt buộc đổi lần đầu) |
| AUTH-04 | Quản lý tài khoản nhân viên (danh sách, tạo, sửa, khóa/mở, đặt lại mật khẩu, gán vai trò) | Admin | 04, 05, 06, 09 |
| AUTH-05 | Quản lý vai trò & phân quyền (CRUD role, ma trận tick permission) | Admin | 07, 08, 10 |
| AUTH-06 | **Xem nhật ký hệ thống (Audit Log)** | Admin | **Mới** |
| AUTH-07 | **Cấu hình hệ thống** (tham số, chính sách đổi trả, thông tin cửa hàng) | Admin, Manager | **Mới** |

> Có thể đổi tên nhóm thành `UC-SYS-xx` cho AUTH-06, 07 nếu muốn tách “Hệ thống” khỏi “Xác thực”.

---

## 3. Danh sách Use Case đề xuất cuối cùng (40 UC)

| Nhóm | Mã | Tên Use Case |
|---|---|---|
| **PROD** | PROD-01 | Quản lý sản phẩm / SKU |
| | PROD-02 | Quản lý danh mục |
| | PROD-03 | Quản lý thương hiệu |
| **STK** | STK-01 | Lập phiếu nhập kho theo lô |
| | STK-02 | Tra cứu tồn kho theo sản phẩm & lô |
| | STK-03 | Theo dõi hàng cận hạn / hết hạn |
| | STK-04 | Lập phiếu kiểm kho |
| | STK-05 | Duyệt điều chỉnh sau kiểm kho |
| | STK-06 | Lập phiếu xuất kho khác (hủy / tester / trả NCC) |
| | STK-07 | Duyệt phiếu xuất kho khác |
| **SUPP** | SUPP-01 | Quản lý nhà cung cấp |
| | SUPP-02 | Tra cứu & xem chi tiết nhà cung cấp |
| | SUPP-03 | Xem lịch sử nhập hàng của nhà cung cấp |
| **CUS** | CUS-01 | Tra cứu & xem hồ sơ khách hàng |
| | CUS-02 | Quản lý hồ sơ khách hàng |
| | CUS-03 | Tích điểm, thu hồi điểm & xét hạng (Hệ thống) |
| | CUS-04 | Dùng điểm thưởng khi thanh toán |
| | CUS-05 | Cấu hình tích điểm & hạng thẻ |
| **PROM** | PROM-01 | Tra cứu chương trình khuyến mãi |
| | PROM-02 | Quản lý chương trình khuyến mãi |
| | PROM-03 | Áp dụng voucher khi bán hàng |
| | PROM-04 | Xem hiệu quả chương trình khuyến mãi |
| **POS** | POS-01 | Mở ca làm việc |
| | POS-02 | Lập đơn bán hàng tại quầy (FEFO) |
| | POS-03 | Thanh toán & xuất hóa đơn |
| | POS-04 | Đổi trả & hoàn tiền |
| | POS-05 | Kiểm đếm tiền & chốt ca |
| **REP** | REP-01 | Dashboard tổng quan |
| | REP-02 | Báo cáo doanh thu & lợi nhuận gộp |
| | REP-03 | Báo cáo bán hàng |
| | REP-04 | Báo cáo tồn kho & hạn dùng |
| | REP-05 | Báo cáo nhập hàng |
| | REP-06 | Báo cáo khách hàng |
| **AUTH** | AUTH-01 | Đăng nhập |
| | AUTH-02 | Đăng xuất |
| | AUTH-03 | Đổi mật khẩu |
| | AUTH-04 | Quản lý tài khoản nhân viên |
| | AUTH-05 | Quản lý vai trò & phân quyền |
| | AUTH-06 | Xem nhật ký hệ thống |
| | AUTH-07 | Cấu hình hệ thống |

*(Đếm: 3+7+3+5+4+5+6+7 = 40 dòng; trong đó CUS-03 là tác vụ tự động — số UC người dùng thao tác là 39.)*

### Quan hệ include / extend chính

```
POS-02 Lập đơn ──<<include>>──> CUS-01 Tra cứu KH (theo SĐT)
POS-02 Lập đơn ──<<extend>>───> CUS-02 Tạo nhanh KH
POS-02 Lập đơn ──<<extend>>───> PROM-03 Áp dụng voucher
POS-03 Thanh toán ──<<extend>>──> CUS-04 Dùng điểm
POS-03 Thanh toán ──<<include>>─> CUS-03 Tích điểm & xét hạng (Hệ thống)
POS-04 Đổi trả ──<<include>>────> CUS-03 Thu hồi điểm (Hệ thống)
STK-03 Cận hạn ──<<extend>>─────> STK-06 Lập đề nghị xuất hủy
```

---

## 4. Ma trận phân quyền đề xuất

Ký hiệu: ✔ toàn quyền · 👁 chỉ xem · — không có quyền

| Use Case | Admin | Manager | NV kho | Thu ngân |
|---|---|---|---|---|
| PROD-01..03 | ✔ | ✔ | ✔ | 👁 |
| STK-01 Nhập kho | ✔ | 👁 | ✔ | — |
| STK-02 Tra cứu tồn | ✔ | ✔ | ✔ | 👁 (chỉ tồn khả dụng) |
| STK-03 Cận hạn | ✔ | ✔ | ✔ | — |
| STK-04 Lập kiểm kho | ✔ | 👁 | ✔ | — |
| STK-05 Duyệt kiểm kho | ✔ | ✔ | — | — |
| STK-06 Lập đề nghị hủy | ✔ | 👁 | ✔ | — |
| STK-07 Duyệt hủy | ✔ | ✔ | — | — |
| SUPP-01 Quản lý NCC | ✔ | ✔ | — | — |
| SUPP-02..03 Tra cứu NCC, lịch sử | ✔ | ✔ | 👁 | — |
| CUS-01 Tra cứu KH | ✔ | ✔ | — | ✔ |
| CUS-02 Quản lý KH | ✔ | ✔ | — | Thêm/sửa (không khóa) |
| CUS-04 Dùng điểm | ✔ | ✔ | — | ✔ |
| CUS-05 Cấu hình điểm/hạng | ✔ | ✔ | — | — |
| PROM-01 Tra cứu KM | ✔ | ✔ | — | 👁 |
| PROM-02 Quản lý KM | ✔ | ✔ | — | — |
| PROM-03 Áp dụng voucher | ✔ | ✔ | — | ✔ |
| PROM-04 Hiệu quả KM | ✔ | ✔ | — | — |
| POS-01..05 | ✔ | 👁 (+ xác nhận bàn giao ca) | — | ✔ |
| REP-01 Dashboard | ✔ | ✔ | Widget kho | — |
| REP-02 Doanh thu & lợi nhuận | ✔ | Doanh thu (không lợi nhuận) | — | — |
| REP-03 Bán hàng | ✔ | ✔ | — | Ca của mình |
| REP-04 Tồn kho & hạn dùng | ✔ | ✔ | ✔ | — |
| REP-05, REP-06 | ✔ | ✔ | — | — |
| AUTH-01..03 | ✔ | ✔ | ✔ | ✔ |
| AUTH-04..06 | ✔ | — | — | — |
| AUTH-07 Cấu hình hệ thống | ✔ | ✔ | — | — |

---

## 5. Tổng hợp thay đổi Database

### 5.1. Bỏ (3 bảng + một số cột)

| Bảng | Lý do |
|---|---|
| DISPOSAL_ALLOCATIONS | Thay bằng cột `blocked_added_qty` + quy tắc mục 2.2 |
| PROMOTION_USAGES | 1 voucher/đơn → `ORDERS.promotion_id` là đủ |
| DAILY_SALES_REPORTS | Không có cơ chế sinh/cập nhật; truy vấn trực tiếp đủ nhanh, tránh lệch số khi trả hàng muộn |
| *(chỉ bỏ cột)* `SUPPLIERS.debt_limit`, `PURCHASE_ORDERS.paid_amount/debt_amount`, `ORDERS.salesperson_id`, `PROMOTIONS.is_stackable` | Đã chốt bỏ công nợ, SALES, cộng dồn voucher |

### 5.2. Thêm (3 bảng)

| Bảng | Mục đích |
|---|---|
| PURCHASE_ORDER_DETAILS | Chi tiết phiếu nhập (lô, hạn, SL, giá nhập) — **bắt buộc** cho STK-01, SUPP-02/03, REP-05 |
| SYSTEM_SETTINGS | Tham số: ngưỡng cận hạn, tỷ lệ điểm, thông tin cửa hàng |
| REFRESH_TOKENS | Thu hồi phiên khi đăng xuất/khóa (chỉ cần nếu dùng JWT) |

### 5.3. Sửa cột (tóm tắt — chi tiết ở từng mục 2.x)

| Bảng | Thêm / sửa |
|---|---|
| USERS | `employee_code`, `failed_login_count`, `locked_until`, `must_change_password`, `last_login_at`, `updated_at` |
| ROLES / PERMISSIONS | `is_system` / `permission_name` |
| AUDIT_LOGS | `entity_type`, `entity_id` |
| CASHIER_SHIFTS | `shift_code`, `handover_confirmed_by`; thống nhất tên cột tiền |
| CATEGORIES / BRANDS | `description` |
| PRODUCTS | `volume`, `description`, `image_url`, `created_at`, `updated_at` |
| BATCHES | `UNIQUE(product_id, lot_code)`, `locked_by_check_id`, `created_at`; giá vốn bình quân |
| STOCK_MOVEMENTS | `created_by`, `balance_after`; chuẩn hóa `reference_type` |
| INVENTORY_CHECKS | Enum status mở rộng, `scope_note`, `submitted_at`, `approved_at`, `reject_reason` |
| INVENTORY_CHECK_DETAILS | `counted_qty`, `difference_qty` cho phép NULL khi đang kiểm |
| DISPOSAL_REQUESTS / DETAILS | Status rút gọn, `approved_at`, `reject_reason` / `blocked_added_qty`, `unit_cost` |
| SUPPLIERS | `address`, `contact_person`, `note`, `created_at`, `updated_at`; `phone_number` UNIQUE |
| PURCHASE_ORDERS | Status `DRAFT/COMPLETED/CANCELLED` (default DRAFT), `supplier_invoice_no`, `note`, `confirmed_at` |
| CUSTOMER_TIERS | `tier_code`, `min_spend` UNIQUE |
| CUSTOMERS | `email`, `date_of_birth`, `gender`, `address`, `note`, `created_at`, `updated_at` |
| POINT_HISTORIES | `return_id`, `balance_after`; action `EARN/REDEEM/REVOKE/ADJUST` |
| PROMOTIONS | `min_tier_id`, `description`, `created_by`, `created_at`, `updated_at`; status chỉ `ACTIVE/PAUSED` |
| RETURN_POLICIES | `is_default`, `status` |
| ORDERS | Status `DRAFT/COMPLETED/CANCELLED`; `payment_method` NULL được; `promotion_id`, tách 3 loại giảm giá, `points_used`, `amount_received`, `change_due`, `completed_at` |
| ORDER_DETAILS | `tax_rate`, `discount_allocated`, `line_total` |
| RETURN_ORDERS | Status `COMPLETED/CANCELLED`, `restocking_fee_amount`, `points_revoked` |
| RETURN_ORDER_DETAILS | **`order_detail_id`**, `unit_cost` |

Tổng: 30 − 3 bảng bỏ + 3 bảng mới = **30 bảng** (29 nếu không dùng JWT).

---

## 6. Quy tắc nghiệp vụ dùng chung (thay mục “Business Rules” cuối PDF)

1. **Tồn khả dụng của lô** = `on_hand_qty − blocked_qty`, chỉ tính lô có `exp_date >= hôm nay` và `locked_by_check_id IS NULL`.
2. **FEFO**: sắp lô khả dụng theo `exp_date ASC`, rồi `batch_id ASC`.
3. **Mọi thay đổi `on_hand_qty`/`blocked_qty`** phải nằm trong cùng transaction với bản ghi `STOCK_MOVEMENTS` tương ứng.
4. **Chứng từ đã hoàn tất không sửa/xóa**; sai thì làm chứng từ ngược (trả hàng, kiểm kho điều chỉnh).
5. **Thứ tự giảm giá**: hạng thẻ → voucher → điểm thưởng. Tổng giảm được phân bổ xuống từng dòng theo tỷ lệ tiền dòng (`discount_allocated`).
6. **Báo cáo**: theo công thức mục 2.7; `FromDate ≤ ToDate`; đơn tính theo `completed_at`, trả hàng tính theo ngày trả.
7. **Ca làm việc**: mỗi thu ngân tối đa 1 ca OPEN; mọi đơn/phiếu trả phải gắn ca OPEN của người thao tác; ca CLOSED không phát sinh thêm chứng từ.
8. **Không cần job chạy nền**: trạng thái cận hạn/hết hạn, trạng thái “sắp diễn ra/đã kết thúc” của khuyến mãi đều **tính khi truy vấn**; tích điểm chạy **đồng bộ** trong transaction hoàn tất đơn.

---

## 7. Gợi ý chia việc cho 5 người & thứ tự làm

| Người | Phụ trách | UC |
|---|---|---|
| 1 | Nền tảng + Phân quyền | AUTH-01..07, middleware kiểm quyền, layout, Audit Log helper, seed permission |
| 2 | Danh mục + Nhập kho | PROD-01..03, SUPP-01..03, STK-01 |
| 3 | Kho nâng cao | STK-02..07 (tồn, cận hạn, kiểm kho, xuất hủy) |
| 4 | Bán hàng | POS-01..05 (FEFO, thanh toán, đổi trả, ca) |
| 5 | CRM + Khuyến mãi + Báo cáo | CUS-01..05, PROM-01..04, REP-01..06 |

**Thứ tự (phụ thuộc dữ liệu):**
1. AUTH + PROD + SUPP (có người dùng, sản phẩm, NCC)
2. STK-01 Nhập kho (có lô, có tồn) → STK-02
3. POS-01..03 + CUS-01/02 (bán được hàng) → tích hợp PROM-03, CUS-03/04
4. POS-04, POS-05, STK-03..07
5. REP + PROM-04 (cần dữ liệu từ các bước trên)

> Người 4 (POS) và người 5 (CUS/PROM) cần **thống nhất sớm** công thức giảm giá & phân bổ (quy tắc 5), vì POS-03/POS-04 phụ thuộc trực tiếp.

---

## 8. Các điểm cần nhóm xác nhận

Những điểm dưới đây tôi đã đưa ra phương án khuyến nghị trong tài liệu, nhưng là quyết định nghiệp vụ của nhóm:

| # | Câu hỏi | Khuyến nghị |
|---|---|---|
| 1 | Hệ thống cho **1 cửa hàng** hay nhiều chi nhánh? | 1 cửa hàng (DB hiện tại không có `store_id`) |
| 2 | Giá niêm yết `retail_price` **đã gồm VAT** chưa? | Đã gồm VAT (đúng thực tế bán lẻ); hóa đơn tách ngược ra tiền thuế |
| 3 | Lô có được bán **trong ngày hết hạn** không? Có ngừng bán trước hạn M ngày không? | Bán đến hết ngày `exp_date`; không chặn sớm (chỉ cảnh báo N ngày) |
| 4 | Có cho **dùng điểm** trừ tiền không? Tỷ lệ? | Có: 1 điểm = 1.000đ, tối đa 50% giá trị đơn (cấu hình được) |
| 5 | Trả hàng có làm **hạ hạng thẻ** không? | Không hạ hạng; chỉ giảm `total_spent` và thu hồi điểm |
| 6 | Có cần **thanh toán kết hợp** (một phần tiền mặt + một phần chuyển khoản) không? | Không — 1 phương thức/đơn |
| 7 | Hoàn tiền/đổi trả có cần **Manager duyệt** không (ma trận cũ có, đặc tả không có)? | Không cần duyệt; Manager xem lại qua báo cáo + Audit Log |
| 8 | Dùng **JWT hay session**? | Quyết định theo công nghệ nhóm chọn; nếu JWT thì thêm REFRESH_TOKENS |

### Quyết định đã chốt thêm (dòng hàng hóa)

| # | Nội dung | Quyết định |
|---|---|---|
| 9 | Khách đổi ý có được hoàn tiền không? | **Không**, chỉ đổi hàng (nguyên seal, trong hạn) |
| 10 | Đổi sang hàng rẻ hơn | Cấp voucher mệnh giá cố định (lệch tối đa ±5.000đ so với phần dư) hoặc khách mua thêm sản phẩm |
| 11 | Hàng trưng bày móp hộp, bạc màu | **Không làm** giảm giá theo dòng. Hàng trên kệ là hàng bán bình thường, trừ tồn khi bán theo FEFO |
| 12 | Tester | Xuất bằng phiếu `TESTER` dùng chung bảng `DISPOSAL_REQUESTS`; **Manager duyệt**; sau khi duyệt hệ thống hết trách nhiệm (không theo dõi hạn, không lập phiếu hủy) |
