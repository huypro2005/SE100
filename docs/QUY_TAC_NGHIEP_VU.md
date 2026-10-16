# QUY TẮC NGHIỆP VỤ CỐ ĐỊNH — SE100

> Luật cố định là luật viết thẳng vào code: Owner và Manager không chỉnh trên giao diện được.
> Tham số chỉnh được nằm ở [mục 9](#9-tham-số-chỉnh-được-không-phải-luật-cố-định).
> Chi tiết: khuyến mãi xem [KHUYEN_MAI.md](KHUYEN_MAI.md); bảng và ràng buộc xem [DATABASE.md](DATABASE.md); lý do từng quyết định xem [DANH_GIA_USECASE.md](DANH_GIA_USECASE.md).

## 1. Khuyến mãi

1. Có 4 tầng, thứ tự cố định:
   - **Tầng 1, dòng hàng:** giảm theo sản phẩm, thương hiệu hoặc danh mục; đồng giá giờ vàng; mua X tặng Y; combo.
   - **Tầng 2, hạng thẻ:** giảm theo `discount_rate` của hạng, áp cả lên món đã giảm ở tầng 1.
   - **Tầng 3, đơn hàng:** bậc thang, voucher, quà theo hóa đơn.
   - **Tầng 4, điểm thưởng:** trừ trên phần tiền còn lại.
2. Mỗi đơn vị sản phẩm hưởng tối đa 1 khuyến mãi tầng 1. Hệ thống tự chọn khuyến mãi lợi nhất cho khách; bằng nhau thì lấy khuyến mãi tạo trước.
3. Tầng 3 chỉ có 1 khoản giảm tiền: bậc thang và voucher, lấy cái lợi hơn. Bằng nhau thì lấy bậc thang, voucher không bị trừ lượt.
4. Quà theo hóa đơn được cộng thêm. Đạt nhiều chương trình quà thì lấy chương trình có ngưỡng cao nhất.
5. Ngưỡng "đơn từ … đ" xét trên tiền sau tầng 1 và 2, không tính dòng quà.
6. Giảm giá tầng 2, 3, 4 chia xuống từng dòng theo tỷ lệ tiền dòng; dòng cuối nhận phần dư do làm tròn.
7. Giá combo chia về từng món theo tỷ lệ giá niêm yết.
8. Giờ vàng chỉ áp khi rẻ hơn giá niêm yết.
9. Giá trị quà tính theo giá niêm yết; quà không bán được (`is_sellable = FALSE`) thì lấy giá vốn.
10. Khuyến mãi đã có đơn dùng thì khóa sửa mức giảm và điều kiện; chỉ được tạm dừng hoặc gia hạn.
11. Voucher có `min_tier_id` hoặc `limit_per_customer` thì đơn bắt buộc có khách hàng.
12. Lượt dùng chỉ tính đơn `COMPLETED`, và kiểm tra lại lúc hoàn tất.
13. Không làm các loại: món thứ N+1, nhân điểm, chi tiêu đổi voucher, giảm hàng cận date.

## 2. Đổi trả

14. Chỉ có 2 lý do trả hàng:
    - **Hàng lỗi** (`DEFECTIVE`: lỗi, hư, hết hạn): hoàn tiền hoặc đổi hàng, không tính phí. Hàng nhận về luôn bị cách ly (`blocked_qty`).
    - **Khách đổi ý** (`CHANGE_OF_MIND`): chỉ đổi hàng, không hoàn tiền. Điều kiện: còn nguyên seal và còn trong thời hạn đổi trả. Mất seal thì từ chối, không lập phiếu.
15. Không có lý do giao nhầm, vì thu ngân quét mã từng sản phẩm khi thanh toán.
16. Quá thời hạn đổi trả, hoặc trả vượt số lượng đã mua, thì chặn.
17. Đổi hàng: tổng hàng mới phải lớn hơn hoặc bằng giá trị hàng cũ, thiếu thì hệ thống chặn. Phần vượt khách nộp thêm. Không hoàn tiền phần chênh.
18. Không bù voucher trong mọi trường hợp, kể cả hàng lỗi.
19. Tiền hoàn tính theo số tiền khách thực trả cho dòng hàng (`line_total`), không theo giá niêm yết. Không tính lại khuyến mãi; không hoàn lại lượt voucher.
20. Nếu trả hàng làm mất điều kiện nhận quà mà khách không mang quà trả lại, thì trừ giá trị quà vào tiền hoàn.
21. Điểm:
    - Điểm đã dùng không hoàn.
    - Điểm đã tích bị thu hồi theo tỷ lệ tiền hoàn.
    - Không hạ hạng thẻ, chỉ trừ tổng chi tiêu (`total_spent`).
22. Trả hàng không cần Manager duyệt.
23. Phiếu trả gắn vào ca hiện tại. Két không đủ tiền mặt thì hoàn bằng chuyển khoản.

## 3. Kho, lô, hạn dùng

24. Tồn bán được = `on_hand − blocked`, chỉ tính lô còn hạn và không bị khóa kiểm kho.
25. FEFO: bán lô hết hạn sớm nhất trước, thiếu thì tự tách sang lô kế tiếp.
26. Được bán đến hết ngày ghi trên hạn dùng; không chặn bán sớm, chỉ cảnh báo trước.
27. Thu ngân đổi lô thủ công chỉ là ngoại lệ: lô phải còn hạn, và hệ thống ghi nhật ký (`OVERRIDE_FEFO`).
28. Mọi thay đổi tồn phải ghi kèm thẻ kho (`STOCK_MOVEMENTS`) trong cùng 1 giao dịch. Luôn đảm bảo `0 ≤ blocked ≤ on_hand`.
29. Nhập thêm vào lô đã có thì giá vốn tính bình quân gia quyền.
30. Bấm xác nhận nhập kho 2 lần không cộng tồn 2 lần.
31. Lô mới nhập có hạn sớm hơn lô đang tồn thì cảnh báo xoay kệ. Hệ thống không quản lý vị trí kệ hay kho.
32. Kiểm kho:
    - Bắt đầu kiểm thì chụp tồn và khóa lô.
    - Ô trống không hiểu là 0.
    - Duyệt thì cộng phần chênh lệch vào tồn hiện tại.
    - Không có chênh lệch thì phiếu tự hoàn tất.
33. Xuất hủy, xuất tester, trả NCC: lập phiếu thì cách ly hàng ngay; Manager duyệt mới trừ tồn; từ chối thì trả lại phần đã cách ly.
34. Tester:
    - Sau khi duyệt, hệ thống không theo dõi tiếp.
    - Không phát sinh doanh thu hay điểm.
    - Hàng tester của hãng không bao giờ được bán.

## 4. Bán hàng và ca

35. Mỗi thu ngân tối đa 1 ca đang mở.
36. Ca đã chốt thì không phát sinh chứng từ. Còn đơn nháp thì không chốt ca được.
37. Chốt ca mà tiền lệch thì bắt buộc nhập giải trình.
38. Mỗi đơn chỉ 1 phương thức thanh toán.
39. Khách đưa thiếu tiền thì chặn hoàn tất. Lỗi máy in thì đơn vẫn hoàn tất.
40. Hoàn tất đơn là 1 giao dịch: kiểm tra lại lô và lượt voucher, trừ tồn, ghi điểm. Lỗi ở bất kỳ bước nào thì không ghi gì cả.
41. Chứng từ đã hoàn tất không sửa, không xóa; sai thì lập chứng từ ngược.

## 5. Khách hàng và điểm

42. Chỉ tích điểm khi đơn hoàn tất và khách đang hoạt động.
43. Tích điểm chạy ngay lúc hoàn tất đơn, không có job chạy nền.
44. Điểm dùng không vượt mức tối đa theo % giá trị đơn.
45. Hạng thẻ xét theo tổng chi tiêu, không theo điểm. Hạng mặc định là hạng có mốc chi tiêu 0.

## 6. Tài khoản

46. Sai mật khẩu 5 lần thì khóa 15 phút.
47. Bắt đổi mật khẩu ở lần đăng nhập đầu.
48. Không được tự khóa chính mình.
49. ADMIN không bị bỏ quyền quản trị.
50. Role hệ thống không xóa được.
51. Danh sách quyền có sẵn trong code, không thêm hay xóa trên giao diện.
52. Khóa tài khoản thì thu hồi phiên đăng nhập ngay.

## 7. Báo cáo

53. Doanh thu tính theo ngày hoàn tất đơn (`completed_at`).
54. Hàng trả trừ vào kỳ phát sinh trả hàng; báo cáo kỳ cũ không đổi.
55. Doanh thu thuần = tiền hàng − giảm giá − hàng bán bị trả.
56. Lợi nhuận gộp = doanh thu thuần chưa VAT − giá vốn.

## 8. Phạm vi

57. Hệ thống cho 1 cửa hàng.
58. Không có công nợ nhà cung cấp, không có nhân viên bán hàng (SALES).
59. Không quản lý vị trí trưng bày; không giảm giá cho hàng trưng bày móp hộp.

## 9. Tham số chỉnh được (không phải luật cố định)

| Tham số | Lưu ở | Giá trị hiện tại |
|---|---|---|
| Số ngày cảnh báo cận hạn | `SYSTEM_SETTINGS.EXPIRY_WARNING_DAYS` | 60 |
| Thời hạn đổi trả | `RETURN_POLICIES.max_return_days` | 7 ngày |
| Tích điểm | `SYSTEM_SETTINGS.POINT_EARN_AMOUNT` | 10.000đ = 1 điểm |
| Giá trị 1 điểm | `SYSTEM_SETTINGS.POINT_REDEEM_VALUE` | Chủ cửa hàng tự đặt (mẫu 100đ) |
| Điểm dùng tối đa | `SYSTEM_SETTINGS.POINT_MAX_REDEEM_PERCENT` | 50% đơn |
| Mốc chi tiêu và % giảm từng hạng | `CUSTOMER_TIERS` | Theo bảng hạng |
| Thông số từng khuyến mãi | `PROMOTIONS` và bảng con | Mức giảm, ngưỡng, thời gian, số lượt |
| Ngưỡng tồn tối thiểu | `PRODUCTS.min_stock_alert` | Theo từng sản phẩm |

## 10. Còn chờ chốt

| # | Câu hỏi | Đang ghi |
|---|---|---|
| 1 | Đổi hàng có cho **bằng giá** không? | Có (luật 17: lớn hơn hoặc bằng) |
| 2 | `RETURN_POLICIES.is_refund_allowed`, `restocking_fee_percent` cho bật hoàn tiền khi đổi ý và đặt phí trả hàng, mâu thuẫn với luật 14. Có bỏ 2 cột này không? | Chưa bỏ |
