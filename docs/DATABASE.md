# ĐẶC TẢ CƠ SỞ DỮ LIỆU — HỆ THỐNG QUẢN LÝ CỬA HÀNG MỸ PHẨM (SE100)

> **Phiên bản: bản đề xuất** (đã áp dụng các sửa đổi trong [TOM_TAT_SUA_DOI.md](TOM_TAT_SUA_DOI.md)). Gồm **34 bảng** chia 5 cụm. Quy tắc nghiệp vụ cố định: [QUY_TAC_NGHIEP_VU.md](QUY_TAC_NGHIEP_VU.md); thiết kế khuyến mãi 4 tầng: [KHUYEN_MAI.md](KHUYEN_MAI.md).
> Ký hiệu cột “Thay đổi”: **Mới** = cột/bảng thêm so với PDF · **Sửa** = đổi ràng buộc hoặc giá trị · để trống = giữ như PDF.
> Kiểu dữ liệu viết theo MySQL / SQL Server phổ thông. Mọi bảng dùng `PK` tự tăng (`AUTO_INCREMENT` / `IDENTITY`) trừ khi ghi khác.
> Sơ đồ ERD tương tác: trang **Sơ đồ ERD** trong [html/index.html](html/index.html).

## Mục lục

| Cụm | Bảng |
|---|---|
| 1. Hệ thống & phân quyền | [USERS](#1-users) · [ROLES](#2-roles) · [PERMISSIONS](#3-permissions) · [USER_ROLES](#4-user_roles) · [ROLE_PERMISSIONS](#5-role_permissions) · [AUDIT_LOGS](#6-audit_logs) · [SYSTEM_SETTINGS](#7-system_settings) · [REFRESH_TOKENS](#8-refresh_tokens) |
| 2. Sản phẩm & kho – lô – date | [CATEGORIES](#9-categories) · [BRANDS](#10-brands) · [PRODUCTS](#11-products) · [BATCHES](#12-batches) · [STOCK_MOVEMENTS](#13-stock_movements) · [INVENTORY_CHECKS](#14-inventory_checks) · [INVENTORY_CHECK_DETAILS](#15-inventory_check_details) · [DISPOSAL_REQUESTS](#16-disposal_requests) · [DISPOSAL_DETAILS](#17-disposal_details) |
| 3. Nhà cung cấp & nhập hàng | [SUPPLIERS](#18-suppliers) · [PURCHASE_ORDERS](#19-purchase_orders) · [PURCHASE_ORDER_DETAILS](#20-purchase_order_details) |
| 4. Khách hàng & khuyến mãi | [CUSTOMER_TIERS](#21-customer_tiers) · [CUSTOMERS](#22-customers) · [POINT_HISTORIES](#23-point_histories) · [PROMOTIONS](#24-promotions) · [PROMOTION_ITEMS](#24a-promotion_items) · [PROMOTION_BRANDS](#24b-promotion_brands) · [PROMOTION_CATEGORIES](#24c-promotion_categories) · [PROMOTION_STEPS](#24d-promotion_steps) |
| 5. Bán hàng, đổi trả & ca | [CASHIER_SHIFTS](#25-cashier_shifts) · [RETURN_POLICIES](#26-return_policies) · [ORDERS](#27-orders) · [ORDER_DETAILS](#28-order_details) · [RETURN_ORDERS](#29-return_orders) · [RETURN_ORDER_DETAILS](#30-return_order_details) |

Bảng **đã bỏ** so với PDF: `DISPOSAL_ALLOCATIONS`, `PROMOTION_USAGES`, `DAILY_SALES_REPORTS` (lý do ở [mục 9](#9-thay-đổi-so-với-pdf)).

---

## CỤM 1 · HỆ THỐNG & PHÂN QUYỀN

Mô hình RBAC: `USERS → USER_ROLES → ROLES → ROLE_PERMISSIONS → PERMISSIONS`.

### 1. USERS
Tài khoản nhân viên đăng nhập hệ thống.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| user_id | INT | PK | Khóa chính | |
| employee_code | VARCHAR(20) | UNIQUE, NOT NULL | Mã nhân viên (NV0007) | Mới |
| username | VARCHAR(50) | UNIQUE, NOT NULL | Tên đăng nhập | |
| email | VARCHAR(100) | UNIQUE, NOT NULL | Email liên hệ / đăng nhập | |
| password_hash | VARCHAR(255) | NOT NULL | Mật khẩu băm BCrypt | |
| full_name | VARCHAR(100) | NOT NULL | Họ tên | |
| phone_number | VARCHAR(15) | NOT NULL | Số điện thoại | |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `LOCKED` (Admin khóa) | |
| failed_login_count | INT | NOT NULL, DEFAULT 0 | Số lần nhập sai mật khẩu liên tiếp | Mới |
| locked_until | DATETIME | NULL | Khóa tạm đến thời điểm này (sai 5 lần → +15 phút) | Mới |
| must_change_password | BOOLEAN | NOT NULL, DEFAULT TRUE | Bắt đổi mật khẩu tạm ở lần đăng nhập đầu | Mới |
| last_login_at | DATETIME | NULL | Lần đăng nhập gần nhất | Mới |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Thời điểm tạo | |
| updated_at | DATETIME | NULL | Thời điểm sửa gần nhất | Mới |

### 2. ROLES
Vai trò / chức vụ.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| role_id | INT | PK | Khóa chính | |
| role_code | VARCHAR(50) | UNIQUE, NOT NULL | `ADMIN`, `STORE_MANAGER`, `INVENTORY_STAFF`, `CASHIER` | Sửa: bỏ `SALE_STAFF` |
| role_name | VARCHAR(100) | NOT NULL | Tên hiển thị | |
| description | VARCHAR(255) | NULL | Mô tả nhiệm vụ | |
| is_system | BOOLEAN | NOT NULL, DEFAULT FALSE | Role mặc định: không được xóa; ADMIN không bị bỏ quyền quản trị | Mới |

### 3. PERMISSIONS
Quyền nguyên tử, **seed sẵn theo mã nguồn**, không tạo/xóa trên giao diện.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| permission_id | INT | PK | Khóa chính | |
| permission_code | VARCHAR(100) | UNIQUE, NOT NULL | Mã quyền, vd `POS_CHECKOUT`, `STOCK_ADJUST_APPROVE`, `REPORT_PROFIT` | |
| permission_name | VARCHAR(100) | NOT NULL | Tên hiển thị trên ma trận tick quyền | Mới |
| module | VARCHAR(50) | NOT NULL | `AUTH`, `PRODUCT`, `INVENTORY`, `SUPPLIER`, `CUSTOMER`, `PROMOTION`, `POS`, `REPORT`, `SYSTEM` | |
| description | VARCHAR(255) | NULL | Diễn giải | |

### 4. USER_ROLES
| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| user_id | INT | PK, FK → USERS | Người dùng |
| role_id | INT | PK, FK → ROLES | Vai trò |

### 5. ROLE_PERMISSIONS
| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| role_id | INT | PK, FK → ROLES | Vai trò |
| permission_id | INT | PK, FK → PERMISSIONS | Quyền |

### 6. AUDIT_LOGS
Nhật ký thao tác. Cũng dùng để lưu lịch sử nâng hạng thẻ và thay đổi cấu hình (không tạo bảng riêng).

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| log_id | BIGINT | PK | Khóa chính | |
| user_id | INT | NULL, FK → USERS | Người thực hiện (NULL = hệ thống) | |
| action | VARCHAR(100) | NOT NULL | `LOGIN`, `LOGOUT`, `CREATE_PRODUCT`, `APPROVE_CHECK`, `CLOSE_SHIFT`, `TIER_UPGRADE`, `UPDATE_SETTING`, `OVERRIDE_FEFO` (thu ngân đổi lô khác FEFO)… | |
| module | VARCHAR(50) | NOT NULL | Module phát sinh | |
| entity_type | VARCHAR(50) | NULL | Loại đối tượng bị tác động (`PRODUCT`, `SUPPLIER`, `ORDER`…) | Mới |
| entity_id | BIGINT | NULL | ID đối tượng | Mới |
| ip_address | VARCHAR(45) | NULL | IP client | Sửa tên (PDF ghi `ip_addres`) |
| details | TEXT (JSON) | NULL | Dữ liệu trước / sau khi đổi | |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Thời điểm ghi | |

### 7. SYSTEM_SETTINGS
**Bảng mới.** Tham số cấu hình dạng khóa – giá trị.

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| setting_key | VARCHAR(50) | PK | Khóa cấu hình |
| setting_value | VARCHAR(255) | NOT NULL | Giá trị (lưu dạng chuỗi) |
| description | VARCHAR(255) | NULL | Diễn giải |
| updated_by | INT | NULL, FK → USERS | Người sửa gần nhất |
| updated_at | DATETIME | NULL | Thời điểm sửa |

Dữ liệu mặc định:

| setting_key | Giá trị mẫu | Ý nghĩa |
|---|---|---|
| EXPIRY_WARNING_DAYS | 60 | Ngưỡng N ngày cảnh báo cận hạn |
| POINT_EARN_AMOUNT | 10000 | Số tiền (VNĐ) khách trả để được 1 điểm |
| POINT_REDEEM_VALUE | 100 | Giá trị 1 điểm khi đổi (VNĐ). Chủ cửa hàng tự đặt; 100 chỉ là giá trị mẫu |
| POINT_MAX_REDEEM_PERCENT | 50 | Tối đa % giá trị đơn được trừ bằng điểm |
| STORE_NAME / STORE_ADDRESS / STORE_PHONE | … | In trên hóa đơn |

### 8. REFRESH_TOKENS
**Bảng mới.** Chỉ cần nếu dùng JWT; dùng session phía server thì bỏ.

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| token_id | BIGINT | PK | Khóa chính |
| user_id | INT | NOT NULL, FK → USERS | Chủ phiên |
| token_hash | VARCHAR(255) | UNIQUE, NOT NULL | Băm của refresh token |
| expires_at | DATETIME | NOT NULL | Hết hạn |
| revoked_at | DATETIME | NULL | Thời điểm thu hồi (đăng xuất, bị khóa, đổi mật khẩu) |

---

## CỤM 2 · SẢN PHẨM & KHO – LÔ – DATE

### 9. CATEGORIES
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| category_id | INT | PK | Khóa chính | |
| category_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã danh mục (CAT_SKINCARE) | |
| category_name | VARCHAR(100) | NOT NULL | Tên danh mục | |
| description | VARCHAR(255) | NULL | Mô tả | Mới |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `INACTIVE` | |

### 10. BRANDS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| brand_id | INT | PK | Khóa chính | |
| brand_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã thương hiệu (LRP, CERAVE) | |
| brand_name | VARCHAR(100) | NOT NULL | Tên thương hiệu | |
| description | VARCHAR(255) | NULL | Mô tả | Mới |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `INACTIVE` | |

### 11. PRODUCTS
Hồ sơ sản phẩm / SKU.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| product_id | INT | PK | Khóa chính | |
| sku | VARCHAR(50) | UNIQUE, NOT NULL | Mã SKU | |
| barcode | VARCHAR(50) | UNIQUE, NULL | Mã vạch quét tại POS | |
| product_name | VARCHAR(255) | NOT NULL | Tên đầy đủ | |
| category_id | INT | NOT NULL, FK → CATEGORIES | Danh mục | |
| brand_id | INT | NOT NULL, FK → BRANDS | Thương hiệu | |
| unit | VARCHAR(20) | NOT NULL, DEFAULT 'Chai' | Đơn vị tính (Chai, Tuýp, Thỏi, Hộp). Không sửa khi đã có giao dịch | |
| volume | VARCHAR(30) | NULL | Quy cách (50ml, 236ml) | Mới |
| skin_type | VARCHAR(50) | NULL | Loại da phù hợp | |
| retail_price | DECIMAL(12,2) | NOT NULL, ≥ 0 | Giá bán niêm yết | |
| tax_rate | DECIMAL(4,2) | NOT NULL, DEFAULT 0.10 | Thuế suất VAT | |
| min_stock_alert | INT | NOT NULL, DEFAULT 5 | Ngưỡng tồn tối thiểu cảnh báo | |
| description | TEXT | NULL | Mô tả sản phẩm | Mới |
| image_url | VARCHAR(255) | NULL | Ảnh sản phẩm | Mới |
| is_sellable | BOOLEAN | NOT NULL, DEFAULT TRUE | FALSE = SKU **tester của hãng** (nhập giá 0đ, chỉ xuất bằng phiếu `TESTER`); POS và FEFO bỏ qua | Mới |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `DISCONTINUED` | |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | | Mới |
| updated_at | DATETIME | NULL | | Mới |

### 12. BATCHES
Lô hàng và hạn dùng. Trung tâm của nghiệp vụ kho.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| batch_id | INT | PK | Khóa chính | |
| product_id | INT | NOT NULL, FK → PRODUCTS | Sản phẩm | |
| lot_code | VARCHAR(50) | NOT NULL, **UNIQUE(product_id, lot_code)** | Số lô in trên bao bì | Sửa: thêm UNIQUE |
| mfg_date | DATE | NULL | Ngày sản xuất | |
| exp_date | DATE | NOT NULL | Hạn dùng, cơ sở FEFO | |
| cost_price | DECIMAL(12,2) | NOT NULL | Giá vốn. Nhập thêm vào lô đã có → bình quân gia quyền | Sửa: cách tính |
| on_hand_qty | INT | NOT NULL, DEFAULT 0, ≥ 0 | Tồn thực tế | |
| blocked_qty | INT | NOT NULL, DEFAULT 0, 0 ≤ blocked ≤ on_hand | Lượng bị cách ly (hỏng, mở nắp, chờ hủy) | |
| locked_by_check_id | INT | NULL, FK → INVENTORY_CHECKS | Khác NULL = lô đang bị khóa kiểm kho | Mới |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Ngày lô vào kho lần đầu (tính tuổi tồn) | Mới |

Tồn bán được của lô: `on_hand_qty − blocked_qty`, chỉ khi `exp_date >= hôm nay` và `locked_by_check_id IS NULL`.

### 13. STOCK_MOVEMENTS
Thẻ kho: mọi tăng/giảm tồn đều có một dòng ở đây, ghi trong cùng transaction.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| movement_id | BIGINT | PK | Khóa chính | |
| batch_id | INT | NOT NULL, FK → BATCHES | Lô bị biến động | |
| reference_type | VARCHAR(30) | NOT NULL | `PURCHASE`, `SALE`, `RETURN`, `CHECK_ADJUST`, `DISPOSAL`, `TESTER`, `SUPPLIER_RETURN` (3 giá trị cuối ứng với `issue_type` của DISPOSAL_REQUESTS) | Sửa: chuẩn hóa giá trị |
| reference_id | BIGINT | NOT NULL | ID chứng từ gốc (po_id, order_id, return_id, check_id, disposal_id; phiếu tester và trả NCC cũng dùng disposal_id). Không đặt FK vì trỏ nhiều bảng | |
| quantity_change | INT | NOT NULL, ≠ 0 | Có dấu: nhập/trả (+), bán/hủy (−), kiểm kho (±) | |
| balance_after | INT | NOT NULL | Tồn của lô sau biến động | Mới |
| reason | VARCHAR(255) | NULL | Ghi chú | |
| created_by | INT | NULL, FK → USERS | Người thực hiện | Mới |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Thời điểm | |

### 14. INVENTORY_CHECKS
Phiếu kiểm kho.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| check_id | INT | PK | Khóa chính | |
| check_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã phiếu (CHK20261003-01) | |
| scope_note | VARCHAR(255) | NULL | Mô tả phạm vi (phạm vi thực tế = tập lô trong DETAILS) | Mới |
| status | VARCHAR(30) | NOT NULL, DEFAULT 'IN_PROGRESS' | `IN_PROGRESS`, `WAITING_APPROVAL`, `APPROVED`, `REJECTED`, `CANCELLED`, `NO_DIFFERENCE` | Sửa: thêm 2 trạng thái |
| snapshot_at | DATETIME | NOT NULL | Thời điểm chụp tồn sổ sách | |
| created_by | INT | NOT NULL, FK → USERS | Nhân viên kho lập phiếu | |
| submitted_at | DATETIME | NULL | Thời điểm gửi duyệt | Mới |
| approved_by | INT | NULL, FK → USERS | Quản lý duyệt / từ chối | |
| approved_at | DATETIME | NULL | Thời điểm duyệt / từ chối | Mới |
| reject_reason | TEXT | NULL | Lý do từ chối | Mới |

### 15. INVENTORY_CHECK_DETAILS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| check_detail_id | INT | PK | Khóa chính | |
| check_id | INT | NOT NULL, FK → INVENTORY_CHECKS | Phiếu kiểm | |
| batch_id | INT | NOT NULL, FK → BATCHES; UNIQUE(check_id, batch_id) | Lô được đếm | |
| snapshot_qty | INT | NOT NULL | Tồn sổ sách lúc chụp | |
| counted_qty | INT | NULL khi đang đếm, bắt buộc khi gửi duyệt, ≥ 0 | Số đếm thực tế | Sửa: cho phép NULL |
| difference_qty | INT | NULL đến khi gửi duyệt | `counted_qty − snapshot_qty` | Sửa |
| note | VARCHAR(255) | NULL | Giải trình chênh lệch | |

### 16. DISPOSAL_REQUESTS
Phiếu đề nghị **xuất kho khác**: xuất hủy, xuất tester, trả nhà cung cấp. Cùng một luồng: NV kho lập → cách ly hàng → Manager duyệt → trừ tồn.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| disposal_id | INT | PK | Khóa chính | |
| disposal_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã phiếu (DISP20261005-01) | |
| issue_type | VARCHAR(30) | NOT NULL, DEFAULT 'DESTROY' | `DESTROY` (tiêu hủy), `TESTER` (đưa ra quầy làm tester), `RETURN_TO_SUPPLIER` (trả NCC) | Mới |
| reason_code | VARCHAR(30) | NOT NULL | `EXPIRED`, `NEAR_EXPIRY`, `DAMAGED`, `RECALL`, `RETURN_DEFECT`, `TESTER_USE` | Mới |
| status | VARCHAR(30) | NOT NULL, DEFAULT 'WAITING_APPROVAL' | `WAITING_APPROVAL`, `APPROVED` (đã trừ tồn), `REJECTED` | Sửa: bỏ `APPROVED_PENDING`, `COMPLETED` |
| reason | TEXT | NOT NULL | Diễn giải lý do (văn bản tự do) | |
| requested_by | INT | NOT NULL, FK → USERS | Nhân viên kho lập | |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Ngày lập | |
| approved_by | INT | NULL, FK → USERS | Quản lý duyệt / từ chối | |
| approved_at | DATETIME | NULL | Thời điểm duyệt / từ chối | Mới |
| reject_reason | TEXT | NULL | Lý do từ chối | Mới |

### 17. DISPOSAL_DETAILS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| disposal_detail_id | INT | PK | Khóa chính | |
| disposal_id | INT | NOT NULL, FK → DISPOSAL_REQUESTS | Phiếu hủy | |
| batch_id | INT | NOT NULL, FK → BATCHES | Lô bị hủy | |
| quantity | INT | NOT NULL, > 0 | Số lượng hủy | |
| blocked_added_qty | INT | NOT NULL, DEFAULT 0 | Lượng hệ thống cách ly thêm khi lập phiếu (trả lại nếu từ chối) | Mới (thay DISPOSAL_ALLOCATIONS) |
| unit_cost | DECIMAL(12,2) | NULL | Giá vốn lô, chụp lúc duyệt (báo cáo tổn thất) | Mới |
| item_reason | VARCHAR(255) | NULL | Lý do riêng từng lô | |

**Quy tắc phiếu `TESTER`:** Manager duyệt mới trừ tồn. Sau khi duyệt, hệ thống **không theo dõi tiếp** chai tester (hết hạn hay dùng hết thì nhân viên tự bỏ, không lập phiếu hủy). Chi phí tester = Σ `quantity × unit_cost` (bằng 0 với tester của hãng). Không phát sinh doanh thu, điểm thưởng, ca làm việc. Phiếu in kèm `exp_date` của lô để nhân viên biết khi nào thay.

---

## CỤM 3 · NHÀ CUNG CẤP & NHẬP HÀNG

### 18. SUPPLIERS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| supplier_id | INT | PK | Khóa chính | |
| supplier_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã NCC, hệ thống sinh (NCC003) | |
| supplier_name | VARCHAR(255) | NOT NULL | Tên công ty | |
| tax_code | VARCHAR(50) | UNIQUE, NULL | Mã số thuế | |
| phone_number | VARCHAR(15) | UNIQUE, NOT NULL | SĐT liên hệ | Sửa: thêm UNIQUE |
| email | VARCHAR(100) | UNIQUE, NULL | Email đặt hàng | Sửa: thêm UNIQUE |
| address | VARCHAR(255) | NOT NULL | Địa chỉ | Mới |
| contact_person | VARCHAR(100) | NULL | Người đại diện liên hệ | Mới |
| note | TEXT | NULL | Ghi chú chính sách chiết khấu | Mới |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `INACTIVE` | |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | | Mới |
| updated_at | DATETIME | NULL | | Mới |

~~debt_limit~~ đã bỏ (không quản lý công nợ).

### 19. PURCHASE_ORDERS
Phiếu nhập kho (một bước: nháp → hoàn tất, không có bước đặt hàng).

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| po_id | INT | PK | Khóa chính | |
| po_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã phiếu (PN20260928-01) | |
| supplier_id | INT | NOT NULL, FK → SUPPLIERS | NCC (chỉ chọn NCC ACTIVE) | |
| supplier_invoice_no | VARCHAR(50) | NULL | Số hóa đơn của NCC | Mới |
| total_amount | DECIMAL(14,2) | NOT NULL, DEFAULT 0 | Σ line_total | |
| status | VARCHAR(30) | NOT NULL, **DEFAULT 'DRAFT'** | `DRAFT`, `COMPLETED`, `CANCELLED` | Sửa: PDF mặc định COMPLETED |
| note | TEXT | NULL | Ghi chú | Mới |
| created_by | INT | NOT NULL, FK → USERS | Nhân viên kho | |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Ngày tạo | |
| confirmed_at | DATETIME | NULL | Thời điểm xác nhận nhập kho | Mới |

~~paid_amount~~, ~~debt_amount~~ đã bỏ.

### 20. PURCHASE_ORDER_DETAILS
**Bảng mới.** Mỗi dòng là một lô nhập.

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| po_detail_id | INT | PK | Khóa chính |
| po_id | INT | NOT NULL, FK → PURCHASE_ORDERS | Phiếu nhập |
| product_id | INT | NOT NULL, FK → PRODUCTS | Sản phẩm |
| lot_code | VARCHAR(50) | NOT NULL | Mã lô |
| mfg_date | DATE | NULL | Ngày sản xuất |
| exp_date | DATE | NOT NULL, > ngày nhập | Hạn dùng |
| quantity | INT | NOT NULL, > 0 | Số lượng thực nhận |
| unit_cost | DECIMAL(12,2) | NOT NULL, ≥ 0 | Giá nhập |
| line_total | DECIMAL(14,2) | NOT NULL | quantity × unit_cost |
| batch_id | INT | NULL, FK → BATCHES | NULL khi phiếu nháp; gán khi xác nhận |

---

## CỤM 4 · KHÁCH HÀNG & KHUYẾN MÃI

### 21. CUSTOMER_TIERS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| tier_id | INT | PK | Khóa chính | |
| tier_code | VARCHAR(20) | UNIQUE, NOT NULL | `BRONZE`, `SILVER`, `GOLD`, `DIAMOND` | Mới |
| tier_name | VARCHAR(50) | UNIQUE, NOT NULL | Tên hiển thị (Đồng, Bạc…) | |
| min_spend | DECIMAL(14,2) | UNIQUE, NOT NULL, ≥ 0 | Mốc chi tiêu lũy kế. Hạng mặc định có min_spend = 0 | Sửa: thêm UNIQUE |
| discount_rate | DECIMAL(4,2) | NOT NULL, DEFAULT 0 | Chiết khấu tự động khi mua (0.03 = 3%) | |

### 22. CUSTOMERS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| customer_id | INT | PK | Khóa chính | |
| customer_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã KH, hệ thống sinh (KH0042) | |
| phone_number | VARCHAR(15) | UNIQUE, NOT NULL | Khóa tra cứu tại POS | |
| full_name | VARCHAR(100) | NOT NULL | Họ tên | |
| email | VARCHAR(100) | NULL | Email | Mới |
| date_of_birth | DATE | NULL | Ngày sinh | Mới |
| gender | VARCHAR(10) | NULL | `MALE`, `FEMALE`, `OTHER` | Mới |
| address | VARCHAR(255) | NULL | Địa chỉ | Mới |
| skin_type | VARCHAR(50) | NULL | Loại da | |
| note | TEXT | NULL | Ghi chú | Mới |
| tier_id | INT | NOT NULL, FK → CUSTOMER_TIERS | Hạng hiện tại | |
| reward_points | INT | NOT NULL, DEFAULT 0, ≥ 0 | Điểm khả dụng | |
| total_spent | DECIMAL(14,2) | NOT NULL, DEFAULT 0 | Chi tiêu lũy kế (trừ khi trả hàng) | |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `INACTIVE` (khóa ưu đãi) | |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Báo cáo khách mới | Mới |
| updated_at | DATETIME | NULL | | Mới |

### 23. POINT_HISTORIES
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| point_id | BIGINT | PK | Khóa chính | |
| customer_id | INT | NOT NULL, FK → CUSTOMERS | Khách hàng | |
| order_id | INT | NULL, FK → ORDERS | Đơn phát sinh (EARN, REDEEM) | |
| return_id | INT | NULL, FK → RETURN_ORDERS | Phiếu trả phát sinh (REVOKE) | Mới |
| action_type | VARCHAR(30) | NOT NULL | `EARN`, `REDEEM`, `REVOKE`, `ADJUST` | Sửa: PDF chỉ có EARNED, REFUNDED |
| points_changed | INT | NOT NULL | Có dấu (+ / −) | |
| balance_after | INT | NOT NULL | Điểm còn lại sau giao dịch | Mới |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | | |

### 24. PROMOTIONS
Chương trình khuyến mãi, gồm 7 loại (cột `promotion_type`) thuộc tầng 1 và tầng 3 của thứ tự 4 tầng cố định. Ưu đãi hạng thẻ (tầng 2) dùng `CUSTOMER_TIERS.discount_rate`, điểm thưởng (tầng 4) dùng `ORDERS.points_used`, hai thứ này không phải bản ghi khuyến mãi. Chi tiết cách tính: [KHUYEN_MAI.md](KHUYEN_MAI.md).

| Cột | Kiểu | Ràng buộc | Dùng cho loại | Mô tả | Thay đổi |
|---|---|---|---|---|---|
| promotion_id | INT | PK | | Khóa chính | |
| promotion_name | VARCHAR(255) | NOT NULL | | Tên chương trình | |
| description | TEXT | NULL | | Mô tả | Mới |
| promotion_type | VARCHAR(30) | NOT NULL | Tất cả | `PRODUCT_DISCOUNT`, `FLASH_PRICE`, `BUY_X_GET_Y`, `COMBO` (tầng 1); `ORDER_TIERED`, `VOUCHER`, `ORDER_GIFT` (tầng 3) | Mới |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'DRAFT' | Tất cả | `DRAFT`, `ACTIVE`, `PAUSED`. "Sắp diễn ra / Đang chạy / Đã kết thúc" tính theo ngày | Sửa |
| voucher_code | VARCHAR(50) | UNIQUE, NULL | VOUCHER | Mã nhập tại POS. Bắt buộc khi `VOUCHER`, NULL với loại khác | Sửa: cho NULL |
| customer_id | INT | NULL, FK → CUSTOMERS | VOUCHER | Voucher riêng 1 khách (quản lý tạo thủ công khi cần). Trả hàng **không** phát voucher bù | Mới |
| apply_scope | VARCHAR(20) | NULL | PRODUCT_DISCOUNT | `ALL`, `PRODUCT`, `BRAND`, `CATEGORY` | Mới |
| discount_type | VARCHAR(20) | NULL | PRODUCT_DISCOUNT, VOUCHER | `PERCENT`, `FIXED_AMOUNT` | Sửa: cho NULL |
| discount_value | DECIMAL(12,2) | NULL, > 0 (PERCENT ≤ 100) | PRODUCT_DISCOUNT, VOUCHER | Mức giảm | Sửa: cho NULL |
| max_discount_amount | DECIMAL(12,2) | NULL | PRODUCT_DISCOUNT, VOUCHER | Trần giảm (loại sản phẩm: trần cho mỗi dòng) | |
| fixed_price | DECIMAL(12,2) | NULL | FLASH_PRICE | Giá đồng giá; chỉ áp khi thấp hơn giá niêm yết | Mới |
| combo_price | DECIMAL(12,2) | NULL | COMBO | Giá cả bộ | Mới |
| buy_quantity | INT | NULL | BUY_X_GET_Y | Số món nhóm "mua" cần có | Mới |
| is_repeatable | BOOLEAN | NOT NULL, DEFAULT FALSE | BUY_X_GET_Y | TRUE: mua 4 tặng 2 (lặp theo bội số) | Mới |
| min_order_value | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | VOUCHER, ORDER_GIFT | Ngưỡng đơn, so với tiền sau tầng 1 và 2 (không tính dòng quà) | |
| min_tier_id | INT | NULL, FK → CUSTOMER_TIERS | Tất cả | Hạng thẻ tối thiểu; khác NULL thì bắt buộc có khách | Mới |
| start_at | DATETIME | NOT NULL | Tất cả | Bắt đầu | Sửa: từ DATE |
| end_at | DATETIME | NOT NULL, ≥ start_at | Tất cả | Kết thúc | Sửa: từ DATE |
| daily_start_time, daily_end_time | TIME | NULL | FLASH_PRICE | Khung giờ trong ngày (giờ vàng) | Mới |
| weekdays | VARCHAR(20) | NULL | FLASH_PRICE | Các thứ áp dụng, vd `1,3,5` (1 = thứ Hai … 7 = Chủ nhật). NULL = mọi ngày | Mới |
| max_quantity | INT | NULL | FLASH_PRICE | Tổng số lượng bán giá khuyến mãi | Mới |
| usage_limit | INT | NULL = không giới hạn | Tất cả | Tổng số đơn được dùng | Sửa: cho NULL |
| limit_per_customer | INT | NULL = không giới hạn, ≤ usage_limit | Tất cả | Số đơn mỗi khách; khác NULL thì bắt buộc có khách | Sửa: cho NULL |
| created_by | INT | NOT NULL, FK → USERS | | Người tạo | Mới |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | | | Mới |
| updated_at | DATETIME | NULL | | | Mới |

Không dùng JSON: mỗi thông số là một cột riêng. `is_stackable` và `priority` đã bỏ (thay bằng thứ tự 4 tầng cố định). Lượt đã dùng đếm từ đơn `COMPLETED` có `ORDERS.order_promotion_id` hoặc `ORDER_DETAILS.line_promotion_id` bằng `promotion_id`.

**Cột bắt buộc theo loại** (kiểm ở tầng ứng dụng khi lưu):

| Loại | Bắt buộc | Bảng con |
|---|---|---|
| `PRODUCT_DISCOUNT` | `apply_scope`, `discount_type`, `discount_value` | `PROMOTION_ITEMS` (role `TARGET`), `PROMOTION_BRANDS` hoặc `PROMOTION_CATEGORIES` theo `apply_scope` |
| `FLASH_PRICE` | `fixed_price` | `PROMOTION_ITEMS` role `TARGET` |
| `BUY_X_GET_Y` | `buy_quantity` | `PROMOTION_ITEMS` role `BUY` và `GIFT` (kèm `quantity` quà) |
| `COMBO` | `combo_price` | `PROMOTION_ITEMS` role `COMBO_ITEM` (kèm `quantity`) |
| `ORDER_TIERED` | | `PROMOTION_STEPS` ít nhất 1 bậc |
| `ORDER_GIFT` | `min_order_value` | `PROMOTION_ITEMS` role `GIFT` |
| `VOUCHER` | `voucher_code`, `discount_type`, `discount_value` | |

Khuyến mãi đã có đơn dùng thì **khóa sửa** mức giảm, điều kiện và bảng con; chỉ được tạm dừng hoặc sửa ngày kết thúc. Nhờ vậy lúc trả hàng dữ liệu khuyến mãi giống hệt lúc bán.

### 24a. PROMOTION_ITEMS
**Bảng mới.** Sản phẩm tham gia khuyến mãi theo vai trò.

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| promotion_id | INT | PK, FK → PROMOTIONS | Khuyến mãi |
| product_id | INT | PK, FK → PRODUCTS | Sản phẩm |
| role | VARCHAR(20) | PK | `TARGET`, `BUY`, `GIFT`, `COMBO_ITEM` |
| quantity | INT | NOT NULL, DEFAULT 1, > 0 | Số lượng (quà tặng, món trong combo) |

Quà có thể là SKU `is_sellable = FALSE` (vd túi tote).

### 24b. PROMOTION_BRANDS
**Bảng mới.** Phạm vi theo thương hiệu.

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| promotion_id | INT | PK, FK → PROMOTIONS | Khuyến mãi |
| brand_id | INT | PK, FK → BRANDS | Thương hiệu |

### 24c. PROMOTION_CATEGORIES
**Bảng mới.** Phạm vi theo danh mục.

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| promotion_id | INT | PK, FK → PROMOTIONS | Khuyến mãi |
| category_id | INT | PK, FK → CATEGORIES | Danh mục |

### 24d. PROMOTION_STEPS
**Bảng mới.** Bậc thang theo giá trị hóa đơn (loại `ORDER_TIERED`).

| Cột | Kiểu | Ràng buộc | Mô tả |
|---|---|---|---|
| step_id | INT | PK | Khóa chính |
| promotion_id | INT | NOT NULL, FK → PROMOTIONS; UNIQUE(promotion_id, min_order_value) | Khuyến mãi |
| min_order_value | DECIMAL(12,2) | NOT NULL | Ngưỡng của bậc |
| discount_type | VARCHAR(20) | NOT NULL | `PERCENT`, `FIXED_AMOUNT` |
| discount_value | DECIMAL(12,2) | NOT NULL, > 0 | Mức giảm |
| max_discount_amount | DECIMAL(12,2) | NULL | Trần giảm |

---

## CỤM 5 · BÁN HÀNG, ĐỔI TRẢ & CA

### 25. CASHIER_SHIFTS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| shift_id | INT | PK | Khóa chính | |
| shift_code | VARCHAR(30) | UNIQUE, NOT NULL | Mã ca (CA20261005-02) | Mới |
| cashier_id | INT | NOT NULL, FK → USERS | Thu ngân | |
| opened_at | DATETIME | NOT NULL | Giờ mở ca | |
| opening_cash | DECIMAL(12,2) | NOT NULL, DEFAULT 0, ≥ 0 | Tiền đầu ca | |
| closed_at | DATETIME | NULL | Giờ chốt ca | |
| expected_cash | DECIMAL(12,2) | NULL | Đầu ca + thu tiền mặt − chi hoàn tiền mặt | |
| actual_cash | DECIMAL(12,2) | NULL, ≥ 0 | Tiền thực đếm | |
| difference_amount | DECIMAL(12,2) | NULL | actual − expected | |
| note | TEXT | NULL, bắt buộc khi difference ≠ 0 | Giải trình | |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'OPEN' | `OPEN`, `CLOSED` | |
| handover_confirmed_by | INT | NULL, FK → USERS | Quản lý xác nhận bàn giao | Mới |

Ràng buộc nghiệp vụ: mỗi `cashier_id` tối đa 1 ca `OPEN` (kiểm ở tầng ứng dụng hoặc unique index có điều kiện nếu CSDL hỗ trợ).

### 26. RETURN_POLICIES
Chỉ còn **thời hạn đổi trả** là tham số chỉnh được. Ai được hoàn tiền, ai chỉ được đổi, có tính phí hay không là **quy tắc cố định** trong code: hàng lỗi hoàn tiền hoặc đổi, không phí; khách đổi ý chỉ đổi hàng.
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| policy_id | INT | PK | Khóa chính | |
| policy_name | VARCHAR(100) | NOT NULL | Tên (Tiêu chuẩn 7 ngày) | |
| max_return_days | INT | NOT NULL, DEFAULT 7 | Số ngày được trả kể từ ngày mua | |
| is_default | BOOLEAN | NOT NULL, DEFAULT FALSE | Chính sách gắn mặc định cho đơn mới (chỉ 1 dòng TRUE) | Mới |
| status | VARCHAR(20) | NOT NULL, DEFAULT 'ACTIVE' | `ACTIVE`, `INACTIVE` | Mới |

### 27. ORDERS
Hóa đơn bán hàng.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| order_id | INT | PK | Khóa chính | |
| order_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã hóa đơn (HD20261005-0015) | |
| shift_id | INT | NOT NULL, FK → CASHIER_SHIFTS | Ca ghi nhận | |
| cashier_id | INT | NOT NULL, FK → USERS | Thu ngân lập đơn | |
| customer_id | INT | NULL, FK → CUSTOMERS | NULL = khách vãng lai | |
| policy_id | INT | NOT NULL, FK → RETURN_POLICIES | Chính sách đổi trả áp cho đơn | |
| order_promotion_id | INT | NULL, FK → PROMOTIONS | Khuyến mãi giảm tiền cấp đơn **đã thắng** ở tầng 3 (bậc thang hoặc voucher). Voucher thua thì không ghi, không bị trừ lượt | Mới (thay PROMOTION_USAGES) |
| subtotal_amount | DECIMAL(12,2) | NOT NULL | Σ unit_price × quantity theo giá niêm yết (dòng quà = 0) | |
| line_discount_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Tổng giảm tầng 1 (dòng hàng) | Mới |
| tier_discount_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Giảm tầng 2 (hạng thẻ) | Mới |
| order_discount_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Giảm tầng 3 (bậc thang hoặc voucher) | Mới |
| points_used | INT | NOT NULL, DEFAULT 0 | Số điểm khách dùng (tầng 4) | Mới |
| points_discount_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Tiền giảm từ điểm | Mới |
| discount_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | `line + tier + order + points` (4 khoản trên) | Sửa: công thức |
| tax_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Tiền VAT | |
| total_amount | DECIMAL(12,2) | NOT NULL | Khách phải trả | |
| exchange_return_id | INT | NULL, FK → RETURN_ORDERS | Đơn này là đơn **mua hàng đổi** của phiếu trả nào | Mới |
| exchange_credit_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Giá trị hàng trả được trừ vào đơn đổi (không phải giảm giá) = **giá trị hàng trả**. Đơn đổi phải có `total_amount ≥ exchange_credit_amount`. Tiền khách phải nộp = `total_amount − exchange_credit_amount` (≥ 0) | Mới |
| payment_method | VARCHAR(20) | **NULL** đến khi hoàn tất | `CASH`, `BANK_TRANSFER`, `CARD` | Sửa: PDF NOT NULL |
| amount_received | DECIMAL(12,2) | NULL | Tiền khách đưa (tiền mặt) | Mới |
| change_due | DECIMAL(12,2) | NULL | Tiền thừa | Mới |
| status | VARCHAR(30) | NOT NULL, DEFAULT 'DRAFT' | `DRAFT`, `COMPLETED`, `CANCELLED` | Sửa: PDF dùng WAITING_FOR_PAYMENT |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Lúc tạo đơn | |
| completed_at | DATETIME | NULL | Lúc hoàn tất; **báo cáo doanh thu tính theo cột này** | Mới |

~~salesperson_id~~ đã bỏ.

### 28. ORDER_DETAILS
Dòng hàng bán, mỗi dòng gắn **một lô**. Lưu giá vốn tại thời điểm bán. Giảm tầng 2, 3, 4 chia xuống từng dòng theo tỷ lệ tiền còn lại của dòng (làm tròn đến đồng, dòng cuối nhận phần dư); tách thành 4 cột để lúc trả hàng lấy chính xác số tiền thực trả, không phải ước lượng.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| order_detail_id | BIGINT | PK | Khóa chính | |
| order_id | INT | NOT NULL, FK → ORDERS | Hóa đơn | |
| product_id | INT | NOT NULL, FK → PRODUCTS | Sản phẩm | |
| batch_id | INT | NOT NULL, FK → BATCHES | Lô xuất theo FEFO | |
| quantity | INT | NOT NULL, > 0 | Số lượng | |
| unit_price | DECIMAL(12,2) | NOT NULL | Giá bán lúc lập đơn | |
| cost_price | DECIMAL(12,2) | NOT NULL | Giá vốn của lô (COGS) | |
| tax_rate | DECIMAL(4,2) | NOT NULL | Thuế suất lúc bán | Mới |
| tax_amount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | VAT của dòng | |
| line_type | VARCHAR(10) | NOT NULL, DEFAULT 'NORMAL' | `NORMAL` (hàng bán), `GIFT` (quà tặng, `unit_price = 0`, vẫn trừ lô và ghi giá vốn) | Mới |
| line_promotion_id | INT | NULL, FK → PROMOTIONS | Khuyến mãi tầng 1 của dòng; với dòng quà: khuyến mãi sinh ra quà (mua X tặng Y hoặc quà theo hóa đơn) | Mới |
| promo_group_no | INT | NULL | Số thứ tự bộ trong đơn: các dòng cùng (`line_promotion_id`, `promo_group_no`) là một combo hoặc một bộ mua-tặng | Mới |
| line_discount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Giảm tầng 1 của dòng (combo: phần giảm đã chia về món này) | Mới |
| tier_discount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Phần giảm tầng 2 phân bổ cho dòng | Mới |
| order_discount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Phần giảm tầng 3 phân bổ cho dòng | Mới |
| points_discount | DECIMAL(12,2) | NOT NULL, DEFAULT 0 | Phần giảm tầng 4 (điểm) phân bổ cho dòng | Mới |
| line_total | DECIMAL(12,2) | NOT NULL | `quantity × unit_price − line_discount − tier_discount − order_discount − points_discount`. Là số tiền khách thực trả của dòng, dùng tính tiền hoàn | Sửa: công thức |

### 29. RETURN_ORDERS
Phiếu đổi trả. Nhận hàng và hoàn tiền là **một giao dịch**.

| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| return_id | INT | PK | Khóa chính | |
| return_code | VARCHAR(50) | UNIQUE, NOT NULL | Mã phiếu (RET20261007-0003) | |
| original_order_id | INT | NOT NULL, FK → ORDERS | Hóa đơn gốc (phải COMPLETED) | |
| shift_id | INT | NOT NULL, FK → CASHIER_SHIFTS | Ca **hiện tại** chi tiền | |
| processed_by | INT | NOT NULL, FK → USERS | Thu ngân xử lý | |
| return_reason | VARCHAR(30) | NOT NULL | `DEFECTIVE` (lỗi, hư, hết hạn), `CHANGE_OF_MIND` (đổi ý, không hợp da). Không có lý do giao nhầm: hàng được quét mã khi thanh toán | Sửa: từ TEXT sang enum |
| return_note | TEXT | NULL | Ghi chú thêm của thu ngân | Mới |
| total_refund_amount | DECIMAL(12,2) | NOT NULL | Σ `refund_amount` (không âm), là tiền hoàn thực tế. Với `EXCHANGE` không chi tiền: bằng `exchange_credit_amount` của đơn đổi (phần được bù vào hàng mới); báo cáo trừ khoản này vào doanh thu | |
| refund_method | VARCHAR(20) | NOT NULL | `CASH`, `BANK_TRANSFER`, `EXCHANGE` (đổi hàng, bắt buộc với `CHANGE_OF_MIND`) | Sửa: thêm EXCHANGE |
| points_revoked | INT | NOT NULL, DEFAULT 0 | Điểm bị thu hồi | Mới |
| status | VARCHAR(30) | NOT NULL, DEFAULT 'COMPLETED' | `COMPLETED`, `CANCELLED` | Sửa: bỏ WAITING_FOR_REFUND, REJECTED |
| created_at | DATETIME | DEFAULT CURRENT_TIMESTAMP | Ngày trả (báo cáo trừ vào ngày này) | |

### 30. RETURN_ORDER_DETAILS
| Cột | Kiểu | Ràng buộc | Mô tả | Thay đổi |
|---|---|---|---|---|
| return_detail_id | INT | PK | Khóa chính | |
| return_id | INT | NOT NULL, FK → RETURN_ORDERS | Phiếu trả | |
| order_detail_id | BIGINT | NOT NULL, FK → ORDER_DETAILS | Dòng hàng gốc | Mới |
| batch_id | INT | NOT NULL, FK → BATCHES | Lô nhận lại (= lô của dòng gốc) | |
| quantity | INT | NOT NULL, > 0 | Σ theo order_detail_id (cả `GIFT_CHARGE`) ≤ quantity dòng gốc | |
| line_kind | VARCHAR(20) | NOT NULL, DEFAULT 'RETURN' | `RETURN` (nhận hàng về), `GIFT_CHARGE` (trừ giá trị quà khách không trả lại, **không nhập kho**) | Mới |
| refund_amount | DECIMAL(12,2) | NOT NULL | Tiền của dòng = `line_total × quantity_trả / quantity` của dòng gốc (lần trả cuối nhận phần còn lại). **Âm** với `GIFT_CHARGE` | Mới (thay refund_unit_price) |
| unit_cost | DECIMAL(12,2) | NOT NULL | cost_price của dòng gốc (trừ COGS) | Mới |
| item_condition | VARCHAR(30) | NOT NULL | `INTACT`, `DAMAGED_OR_OPENED` (→ tăng blocked_qty) | |

---

## 6. Danh sách quan hệ (khóa ngoại)

Tất cả là quan hệ **1 – n** (bảng bên trái là “1”).

| Bảng cha (1) | Bảng con (n) · cột FK |
|---|---|
| USERS | USER_ROLES.user_id · REFRESH_TOKENS.user_id · AUDIT_LOGS.user_id · SYSTEM_SETTINGS.updated_by · CASHIER_SHIFTS.cashier_id · CASHIER_SHIFTS.handover_confirmed_by · ORDERS.cashier_id · RETURN_ORDERS.processed_by · STOCK_MOVEMENTS.created_by · INVENTORY_CHECKS.created_by / approved_by · DISPOSAL_REQUESTS.requested_by / approved_by · PURCHASE_ORDERS.created_by · PROMOTIONS.created_by |
| ROLES | USER_ROLES.role_id · ROLE_PERMISSIONS.role_id |
| PERMISSIONS | ROLE_PERMISSIONS.permission_id |
| CATEGORIES | PRODUCTS.category_id · PROMOTION_CATEGORIES.category_id |
| BRANDS | PRODUCTS.brand_id · PROMOTION_BRANDS.brand_id |
| PRODUCTS | BATCHES.product_id · PURCHASE_ORDER_DETAILS.product_id · ORDER_DETAILS.product_id · PROMOTION_ITEMS.product_id |
| BATCHES | STOCK_MOVEMENTS.batch_id · INVENTORY_CHECK_DETAILS.batch_id · DISPOSAL_DETAILS.batch_id · PURCHASE_ORDER_DETAILS.batch_id · ORDER_DETAILS.batch_id · RETURN_ORDER_DETAILS.batch_id |
| INVENTORY_CHECKS | INVENTORY_CHECK_DETAILS.check_id · BATCHES.locked_by_check_id (0..1) |
| DISPOSAL_REQUESTS | DISPOSAL_DETAILS.disposal_id |
| SUPPLIERS | PURCHASE_ORDERS.supplier_id |
| PURCHASE_ORDERS | PURCHASE_ORDER_DETAILS.po_id |
| CUSTOMER_TIERS | CUSTOMERS.tier_id · PROMOTIONS.min_tier_id |
| CUSTOMERS | ORDERS.customer_id · POINT_HISTORIES.customer_id · PROMOTIONS.customer_id |
| PROMOTIONS | ORDERS.order_promotion_id · ORDER_DETAILS.line_promotion_id · PROMOTION_ITEMS.promotion_id · PROMOTION_BRANDS.promotion_id · PROMOTION_CATEGORIES.promotion_id · PROMOTION_STEPS.promotion_id |
| CASHIER_SHIFTS | ORDERS.shift_id · RETURN_ORDERS.shift_id |
| RETURN_POLICIES | ORDERS.policy_id |
| ORDERS | ORDER_DETAILS.order_id · RETURN_ORDERS.original_order_id · POINT_HISTORIES.order_id |
| ORDER_DETAILS | RETURN_ORDER_DETAILS.order_detail_id |
| RETURN_ORDERS | RETURN_ORDER_DETAILS.return_id · POINT_HISTORIES.return_id · ORDERS.exchange_return_id |

**Liên kết đa hình (không có FK):** `STOCK_MOVEMENTS(reference_type, reference_id)` trỏ tới PURCHASE_ORDERS / ORDERS / RETURN_ORDERS / INVENTORY_CHECKS / DISPOSAL_REQUESTS tùy `reference_type`.

### Sơ đồ quan hệ (mermaid)

```mermaid
erDiagram
  USERS ||--o{ USER_ROLES : ""
  ROLES ||--o{ USER_ROLES : ""
  ROLES ||--o{ ROLE_PERMISSIONS : ""
  PERMISSIONS ||--o{ ROLE_PERMISSIONS : ""
  USERS ||--o{ AUDIT_LOGS : ""
  USERS ||--o{ REFRESH_TOKENS : ""
  USERS ||--o{ CASHIER_SHIFTS : "thu ngân"
  CATEGORIES ||--o{ PRODUCTS : ""
  BRANDS ||--o{ PRODUCTS : ""
  PRODUCTS ||--o{ BATCHES : ""
  BATCHES ||--o{ STOCK_MOVEMENTS : ""
  INVENTORY_CHECKS ||--o{ INVENTORY_CHECK_DETAILS : ""
  BATCHES ||--o{ INVENTORY_CHECK_DETAILS : ""
  INVENTORY_CHECKS |o--o{ BATCHES : "khóa lô"
  DISPOSAL_REQUESTS ||--o{ DISPOSAL_DETAILS : ""
  BATCHES ||--o{ DISPOSAL_DETAILS : ""
  SUPPLIERS ||--o{ PURCHASE_ORDERS : ""
  PURCHASE_ORDERS ||--o{ PURCHASE_ORDER_DETAILS : ""
  PRODUCTS ||--o{ PURCHASE_ORDER_DETAILS : ""
  BATCHES |o--o{ PURCHASE_ORDER_DETAILS : ""
  CUSTOMER_TIERS ||--o{ CUSTOMERS : ""
  CUSTOMER_TIERS |o--o{ PROMOTIONS : "hạng tối thiểu"
  CUSTOMERS ||--o{ POINT_HISTORIES : ""
  CUSTOMERS |o--o{ ORDERS : ""
  PROMOTIONS |o--o{ ORDERS : "khuyến mãi cấp đơn"
  PROMOTIONS |o--o{ ORDER_DETAILS : "khuyến mãi dòng"
  PROMOTIONS ||--o{ PROMOTION_ITEMS : ""
  PRODUCTS ||--o{ PROMOTION_ITEMS : ""
  PROMOTIONS ||--o{ PROMOTION_BRANDS : ""
  BRANDS ||--o{ PROMOTION_BRANDS : ""
  PROMOTIONS ||--o{ PROMOTION_CATEGORIES : ""
  CATEGORIES ||--o{ PROMOTION_CATEGORIES : ""
  PROMOTIONS ||--o{ PROMOTION_STEPS : ""
  CUSTOMERS |o--o{ PROMOTIONS : "voucher riêng"
  CASHIER_SHIFTS ||--o{ ORDERS : ""
  RETURN_POLICIES ||--o{ ORDERS : ""
  ORDERS ||--o{ ORDER_DETAILS : ""
  PRODUCTS ||--o{ ORDER_DETAILS : ""
  BATCHES ||--o{ ORDER_DETAILS : ""
  ORDERS ||--o{ RETURN_ORDERS : ""
  CASHIER_SHIFTS ||--o{ RETURN_ORDERS : ""
  RETURN_ORDERS ||--o{ RETURN_ORDER_DETAILS : ""
  ORDER_DETAILS ||--o{ RETURN_ORDER_DETAILS : ""
  BATCHES ||--o{ RETURN_ORDER_DETAILS : ""
  ORDERS |o--o{ POINT_HISTORIES : ""
  RETURN_ORDERS |o--o{ POINT_HISTORIES : ""
  RETURN_ORDERS |o--o{ ORDERS : "đơn mua hàng đổi"
```

*(Các FK “người thực hiện” như created_by, approved_by… tới USERS được lược bớt khỏi sơ đồ cho dễ đọc; xem bảng ở trên.)*

## 7. Giá trị enum

| Bảng.cột | Giá trị |
|---|---|
| USERS.status | ACTIVE, LOCKED |
| ROLES.role_code | ADMIN, STORE_MANAGER, INVENTORY_STAFF, CASHIER |
| CATEGORIES / BRANDS / SUPPLIERS / RETURN_POLICIES.status | ACTIVE, INACTIVE |
| PRODUCTS.status | ACTIVE, DISCONTINUED |
| STOCK_MOVEMENTS.reference_type | PURCHASE, SALE, RETURN, CHECK_ADJUST, DISPOSAL, TESTER, SUPPLIER_RETURN |
| DISPOSAL_REQUESTS.issue_type | DESTROY, TESTER, RETURN_TO_SUPPLIER |
| DISPOSAL_REQUESTS.reason_code | EXPIRED, NEAR_EXPIRY, DAMAGED, RECALL, RETURN_DEFECT, TESTER_USE |
| RETURN_ORDERS.return_reason | DEFECTIVE, CHANGE_OF_MIND |
| INVENTORY_CHECKS.status | IN_PROGRESS, WAITING_APPROVAL, APPROVED, REJECTED, CANCELLED, NO_DIFFERENCE |
| DISPOSAL_REQUESTS.status | WAITING_APPROVAL, APPROVED, REJECTED |
| PURCHASE_ORDERS.status | DRAFT, COMPLETED, CANCELLED |
| CUSTOMER_TIERS.tier_code | BRONZE, SILVER, GOLD, DIAMOND |
| CUSTOMERS.status | ACTIVE, INACTIVE |
| CUSTOMERS.gender | MALE, FEMALE, OTHER |
| POINT_HISTORIES.action_type | EARN, REDEEM, REVOKE, ADJUST |
| PROMOTIONS.promotion_type | PRODUCT_DISCOUNT, FLASH_PRICE, BUY_X_GET_Y, COMBO, ORDER_TIERED, VOUCHER, ORDER_GIFT |
| PROMOTIONS.apply_scope | ALL, PRODUCT, BRAND, CATEGORY |
| PROMOTIONS.discount_type, PROMOTION_STEPS.discount_type | PERCENT, FIXED_AMOUNT |
| PROMOTIONS.status | DRAFT, ACTIVE, PAUSED |
| PROMOTION_ITEMS.role | TARGET, BUY, GIFT, COMBO_ITEM |
| ORDER_DETAILS.line_type | NORMAL, GIFT |
| RETURN_ORDER_DETAILS.line_kind | RETURN, GIFT_CHARGE |
| CASHIER_SHIFTS.status | OPEN, CLOSED |
| ORDERS.status | DRAFT, COMPLETED, CANCELLED |
| ORDERS.payment_method | CASH, BANK_TRANSFER, CARD |
| RETURN_ORDERS.status | COMPLETED, CANCELLED |
| RETURN_ORDERS.refund_method | CASH, BANK_TRANSFER, EXCHANGE |
| RETURN_ORDER_DETAILS.item_condition | INTACT, DAMAGED_OR_OPENED |

## 8. Index & ràng buộc nghiệp vụ

**Index đề xuất** (ngoài PK / UNIQUE / FK):

| Bảng | Index | Phục vụ |
|---|---|---|
| BATCHES | (product_id, exp_date) | Chọn lô FEFO |
| STOCK_MOVEMENTS | (batch_id, created_at) | Thẻ kho |
| STOCK_MOVEMENTS | (reference_type, reference_id) | Tra từ chứng từ |
| ORDERS | (status, completed_at) | Báo cáo doanh thu |
| ORDERS | (shift_id, status) | Chốt ca |
| ORDERS | (order_promotion_id, customer_id) | Đếm lượt voucher, bậc thang |
| ORDER_DETAILS | (line_promotion_id) | Đếm lượt khuyến mãi dòng hàng, số lượng giờ vàng đã bán |
| RETURN_ORDERS | (created_at) | Báo cáo hàng trả |
| POINT_HISTORIES | (customer_id, created_at) | Lịch sử điểm |
| AUDIT_LOGS | (entity_type, entity_id), (created_at) | Tra nhật ký |

**Ràng buộc nghiệp vụ (kiểm ở tầng ứng dụng):**
1. Mọi thay đổi `BATCHES.on_hand_qty` / `blocked_qty` đi kèm 1 dòng `STOCK_MOVEMENTS` trong **cùng transaction**.
2. `0 ≤ blocked_qty ≤ on_hand_qty`.
3. Mỗi thu ngân tối đa 1 ca `OPEN`; ORDERS/RETURN_ORDERS chỉ gắn ca `OPEN` của người thao tác.
4. Chứng từ `COMPLETED` / `APPROVED` không sửa, không xóa.
5. Tổng `RETURN_ORDER_DETAILS.quantity` theo một `order_detail_id` ≤ `ORDER_DETAILS.quantity`.
6. `ORDERS.discount_amount = line_discount_amount + tier_discount_amount + order_discount_amount + points_discount_amount`; với mỗi khoản, Σ cột tương ứng của `ORDER_DETAILS` = cột của `ORDERS`; `total_amount = Σ ORDER_DETAILS.line_total`.
7. Khuyến mãi có `min_tier_id` hoặc `limit_per_customer` thì đơn bắt buộc có `customer_id`.
8. Chỉ một `RETURN_POLICIES` có `is_default = TRUE`.
9. Phiếu `DISPOSAL_REQUESTS` loại `TESTER`: SKU `is_sellable = FALSE` chỉ được xuất bằng loại `TESTER` hoặc `DESTROY`; POS/FEFO không bao giờ chọn SKU này.
10. `RETURN_ORDERS.refund_method = EXCHANGE` bắt buộc khi `return_reason = CHANGE_OF_MIND`; hàng `DAMAGED_OR_OPENED` + `CHANGE_OF_MIND` bị từ chối (không tạo phiếu). `DEFECTIVE` luôn cộng `blocked_qty`.
11. Tiền mặt phải nộp của đơn = `total_amount − exchange_credit_amount`; chốt ca chỉ tính phần này, và chỉ chi tiền hoàn khi `refund_method = CASH`.
12. Đơn đổi hàng (`exchange_return_id` khác NULL): `exchange_credit_amount = giá trị hàng trả`. **Chặn hoàn tất** khi `total_amount < exchange_credit_amount`: khách phải chọn thêm hàng cho đủ giá trị hàng cũ, phần vượt thì nộp thêm. Không hoàn tiền phần chênh, **không cấp voucher**.
13. Trả hàng **không phát voucher bù** trong mọi trường hợp (kể cả hàng lỗi); điểm đã dùng không hoàn.
14. Mỗi đơn vị sản phẩm hưởng tối đa 1 khuyến mãi tầng 1; tầng 3 chỉ có 1 khoản giảm tiền (`order_promotion_id`); thứ tự tầng cố định, không có `priority` hay `is_stackable`.
15. Khuyến mãi đã có đơn dùng không được sửa mức giảm, điều kiện, bảng con.

## 9. Thay đổi so với PDF

| Loại | Nội dung |
|---|---|
| Bỏ bảng | `DISPOSAL_ALLOCATIONS` → cột `DISPOSAL_DETAILS.blocked_added_qty` · `PROMOTION_USAGES` → `ORDERS.order_promotion_id` và `ORDER_DETAILS.line_promotion_id` · `DAILY_SALES_REPORTS` → báo cáo truy vấn trực tiếp |
| Thêm bảng | `PURCHASE_ORDER_DETAILS` · `SYSTEM_SETTINGS` · `REFRESH_TOKENS` · `PROMOTION_ITEMS` · `PROMOTION_BRANDS` · `PROMOTION_CATEGORIES` · `PROMOTION_STEPS` |
| Bỏ cột | SUPPLIERS.debt_limit · PURCHASE_ORDERS.paid_amount, debt_amount · ORDERS.salesperson_id · PROMOTIONS.is_stackable · RETURN_POLICIES.is_refund_allowed, restocking_fee_percent · RETURN_ORDERS.restocking_fee_amount · ORDER_DETAILS.discount_allocated · RETURN_ORDER_DETAILS.refund_unit_price |
| Thêm / sửa cột | Đánh dấu **Mới** / **Sửa** trong từng bảng ở trên |
| Chốt nghiệp vụ trả hàng & tester | `PRODUCTS.is_sellable` · `DISPOSAL_REQUESTS.issue_type`, `reason_code` · `STOCK_MOVEMENTS.reference_type` thêm `TESTER`, `SUPPLIER_RETURN` · `RETURN_ORDERS.return_reason` (enum), `return_note`, `refund_method = EXCHANGE` · `ORDERS.exchange_return_id`, `exchange_credit_amount` |

Tổng: 30 − 3 + 7 = **34 bảng** (33 nếu không dùng JWT).
