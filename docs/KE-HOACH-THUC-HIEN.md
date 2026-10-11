# Kế hoạch thực hiện: thứ tự use case và timeline

> Dựa trên danh sách 40 UC trong [DANH_GIA_USECASE.md](DANH_GIA_USECASE.md). Quy tắc đặt tên nhánh xem [QUY-TAC-GIT.md](QUY-TAC-GIT.md).

## 1. Giả định

| Mục | Giả định |
|---|---|
| Thời gian | **12 tuần**, sprint 2 tuần. Tuần 1 tính từ ngày bắt đầu (điền ngày thật vào cột "Ngày") |
| **Team A: Kho** | BE-A + FE-A. Phụ trách AUTH (xác thực), PROD, SUPP, STK |
| **Team B: Bán hàng** | BE-B + FE-B. Phụ trách AUTH (quản trị), CUS, PROM, POS |
| PM | Hỗ trợ BE: setup, seed data, cấu hình hệ thống, một phần báo cáo. **Không nhận task trên đường găng** |

Team A làm trước phần dữ liệu nền (sản phẩm, lô hàng). Trong thời gian đó, Team B làm những phần không cần tồn kho (tài khoản, khách hàng, khuyến mãi). Đến Sprint 3 thì hai bên gặp nhau ở POS.

## 2. Phân tích phụ thuộc: làm cái gì trước

### 2.1. Đường găng

Đây là chuỗi UC mà **trễ một ngày thì cả dự án trễ một ngày**:

```
Setup ─► AUTH-01..03 ─► PROD-02..03 ─► PROD-01 ─► STK-01 ─► POS-02 ─► POS-03 ─► POS-04 ─► REP-02..03
         (đăng nhập)    (danh mục,     (sản phẩm)  (nhập kho  (lập đơn   (thanh    (đổi trả)  (doanh thu,
                         thương hiệu)              tạo lô)    FEFO)      toán)                 lợi nhuận)
```

**STK-01 (Nhập kho) là điểm nghẽn quan trọng nhất.** Chưa có lô hàng thì không bán được, không kiểm kho được, không xuất hủy được, cũng không có dữ liệu cho báo cáo.

### 2.2. Thứ tự ưu tiên theo tầng

| Tầng | Use case | Vì sao làm ở tầng này |
|---|---|---|
| **0. Nền tảng** | Setup, migration DB, seed data | Mọi thứ khác đều cần |
| **1. Xác thực** | AUTH-01..03, AUTH-04..05 | Mọi API đều cần đăng nhập và kiểm quyền |
| **2. Dữ liệu gốc** | PROD-01..03, SUPP-01..02, CUS-01..02, AUTH-07 | Không phụ thuộc nghiệp vụ nào. Các UC sau đều đọc dữ liệu này |
| **3. Cấu hình bán hàng** | CUS-05, PROM-01..02, POS-01 | Cần có trước khi bán: hạng thẻ, voucher, ca làm việc |
| **4. Nghiệp vụ lõi** | STK-01..03, POS-02..03, PROM-03, CUS-03..04 | Luồng chính: nhập hàng, bán hàng |
| **5. Nghiệp vụ phụ** | STK-04..07, POS-04..05, SUPP-03 | Cần có tồn kho và đơn hàng thật |
| **6. Báo cáo** | REP-01..06, PROM-04, AUTH-06 | Chỉ đọc dữ liệu, làm cuối cùng |

### 2.3. Mức ưu tiên khi trễ tiến độ

Khi trễ thì **cắt từ dưới lên**, không cắt chất lượng.

| Mức | Use case |
|---|---|
| **Must** (bắt buộc) | AUTH-01..05, AUTH-07, PROD-01..03, SUPP-01..02, STK-01..03, CUS-01..02, POS-01..05 |
| **Should** (nên có) | STK-04..07, CUS-03..05, PROM-01..03, REP-01..04 |
| **Could** (có thì tốt) | SUPP-03, PROM-04, REP-05, REP-06, AUTH-06 (giao diện xem log; phần ghi log vẫn làm từ đầu) |

## 3. Gom nhóm nhánh

Mỗi nhóm có một nhánh BE và một nhánh FE: `feature/<mã>-be-<mo-ta>` và `feature/<mã>-fe-<mo-ta>`.

| Nhóm | Use case | Tên nhánh (bỏ tiền tố `feature/`, thay `xx` bằng `be` hoặc `fe`) | Phụ thuộc | Người làm | Sprint |
|---|---|---|---|---|---|
| **G00** | Setup dự án | `chore/be-setup`, `chore/fe-setup`, `chore/be-seed-data` | Không | PM + 2 lead | S0 |
| **G01** | AUTH-01..03 | `auth-01-03-xx-xac-thuc` | G00 | Team A | S1 |
| **G02** | AUTH-04..05 | `auth-04-05-xx-tai-khoan-phan-quyen` | G01 | Team B | S1 |
| **G03** | AUTH-07 | `auth-07-xx-cau-hinh-he-thong` | G00 | PM (BE), FE-B | S1 |
| **G04** | PROD-02..03 | `prod-02-03-xx-danh-muc-thuong-hieu` | G01 | Team A | S1 |
| **G05** | PROD-01 | `prod-01-xx-san-pham` | G04 | Team A | S1 |
| **G06** | CUS-01..02 | `cus-01-02-xx-khach-hang` | G01 | Team B | S1 |
| **G07** | SUPP-01..02 | `supp-01-02-xx-nha-cung-cap` | G01 | Team A | S2 |
| **G08** | **STK-01** ★ | `stk-01-xx-nhap-kho` | G05, G07 | Team A | S2 |
| **G09** | STK-02..03 | `stk-02-03-xx-ton-kho-can-han` | G08, G03 | Team A | S2 |
| **G10** | CUS-05 | `cus-05-xx-cau-hinh-diem-hang` | G03 | Team B | S2 |
| **G11** | PROM-01..02 | `prom-01-02-xx-khuyen-mai` | G10 | Team B | S2 |
| **G12** | POS-01 | `pos-01-xx-mo-ca` | G01 | Team B | S2 |
| **G13** | **POS-02** ★ | `pos-02-xx-lap-don-fefo` | G08, G06, G12 | Team B | S3 |
| **G14** | PROM-03 | `prom-03-xx-ap-voucher` | G11, G13 | Team B | S3 |
| **G15** | **POS-03** ★ | `pos-03-xx-thanh-toan` | G13 | Team B | S3 |
| **G16** | STK-04..05 | `stk-04-05-xx-kiem-kho` | G09 | Team A | S3 |
| **G17** | STK-06..07 | `stk-06-07-xx-xuat-huy` | G09 | Team A | S3 |
| **G18** | SUPP-03 | `supp-03-xx-lich-su-nhap` | G08 | Team A | S3 |
| **G19** | CUS-03..04 | `cus-03-04-xx-tich-dung-diem` | G15, G10 | BE-A, FE-B | S4 |
| **G20** | **POS-04** ★ | `pos-04-xx-doi-tra` | G15, G19 | Team B | S4 |
| **G21** | POS-05 | `pos-05-xx-chot-ca` | G15 | Team B | S4 |
| **G22** | REP-04..05 | `rep-04-05-xx-bao-cao-kho-nhap` | G09, G08 | Team A | S4 |
| **G23** | REP-01 | `rep-01-xx-dashboard` | G15, G09 | PM (BE), FE-A | S4 |
| **G24** | AUTH-06 | `auth-06-xx-audit-log` | G00 | Team A | S4 |
| **G25** | REP-02..03 | `rep-02-03-xx-doanh-thu-ban-hang` | G20 | BE-B, FE-A | S5 |
| **G26** | REP-06 | `rep-06-xx-bao-cao-khach-hang` | G19 | PM (BE), FE-B | S5 |
| **G27** | PROM-04 | `prom-04-xx-hieu-qua-km` | G14 | PM (BE), FE-B | S5 |

★ = nằm trên đường găng. Lead phải theo dõi sát, có vấn đề phải báo ngay.

**Ghi chú:**
- **G01 (AUTH-01..03)** gộp được vì cùng nhóm, nhỏ, một người làm. Nên làm **đầu tiên**, vì mọi màn hình khác đều cần đăng nhập.
- **G13, G15, G20 (POS-02, 03, 04)** là UC lớn nhất, **không gộp**. Nếu quá 400 dòng thì chia nhánh nhỏ hơn (ví dụ `pos-02-be-fefo`, `pos-02-be-luu-nhap`).
- **FE không phải chờ BE.** FE làm trên mock server theo API contract, khi BE merge thì chuyển sang API thật.
- **Audit log:** phần helper ghi log làm ngay trong G00 để mọi UC dùng chung. G24 chỉ là màn hình xem log.

## 4. Timeline

| Tuần | Ngày | Sprint | Team A (Kho) | Team B (Bán hàng) | PM | Mốc |
|---|---|---|---|---|---|---|
| 1 | | **S0** | Setup FE/BE, chốt API contract | Setup FE/BE, chốt API contract | Chốt DB, seed data, trả lời 8 câu hỏi mục 8 của DANH_GIA | **M0:** contract, mock server, DB đã chốt |
| 2–3 | | **S1** | G01, G04, G05 | G02, G06 | G03 (BE) | **M1:** đăng nhập, phân quyền, có sản phẩm và khách hàng |
| 4–5 | | **S2** | G07, **G08**, G09 | G10, G11, G12 | Seed data lô hàng cho Team B test POS | **M2:** nhập kho tạo lô, tra tồn, có voucher và ca làm việc |
| 6–7 | | **S3** | G16, G17, G18 | **G13**, G14, **G15** | Hỗ trợ review POS | **M3:** bán hàng chạy từ đầu đến cuối |
| 8–9 | | **S4** | G19 (BE), G22, G24 | **G20**, G21, G19 (FE) | G23 (BE) | **M4:** đổi trả, chốt ca, tích điểm, báo cáo kho |
| 10 | | **S5** | Sửa bug, hỗ trợ báo cáo | G25 | G26, G27 (BE) | **M5:** đủ 40 UC, **code freeze** |
| 11 | | Kiểm thử | Test chéo, sửa bug | Test chéo, sửa bug | UAT theo tiêu chí chấp nhận | **M6:** không còn bug Critical/High |
| 12 | | Bàn giao | Tài liệu, dự phòng | Tài liệu, dự phòng | Tag `v1.0`, tập demo | **M7:** bàn giao |

## 5. Rủi ro cần theo dõi

| Rủi ro | Cách xử lý |
|---|---|
| **STK-01 trễ** thì POS bị chặn | PM tạo seed data lô hàng sẵn trong S2 để Team B code POS mà không phải chờ |
| **POS-02, POS-03 khó nhất** (FEFO, transaction, giảm giá) | BE-B làm POS từ đầu S3. Viết unit test cho FEFO và công thức giảm giá trước khi nối FE |
| **Công thức giảm giá** (hạng → voucher → điểm) ảnh hưởng POS-03, POS-04, CUS, REP | Chốt trong S0, ghi vào API contract. Không đổi sau khi chốt |
| **8 câu hỏi nghiệp vụ** (mục 8 DANH_GIA) chưa trả lời | Bắt buộc chốt trong tuần 1. Câu VAT và câu dùng điểm ảnh hưởng trực tiếp DB |
| **Migration DB conflict** khi nhiều người cùng sửa | Chốt toàn bộ 30 bảng trong G00. Cần sửa thì tạo migration mới và báo trong nhóm |
| **Team B quá tải ở S3 và S4** | BE-A làm phần BE của CUS-03..04. Nếu vẫn trễ thì cắt UC mức Could |
