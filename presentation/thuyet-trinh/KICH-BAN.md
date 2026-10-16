# KỊCH BẢN THUYẾT TRÌNH — SE100

> Đi kèm trang trình chiếu [index.html](index.html) (14 slide). Mở trang, bấm **F** để toàn màn hình, **→** để chuyển slide, **N** để bật ghi chú ngắn trên màn hình người nói.
> Thời lượng dự kiến: **khoảng 15 phút** trình bày + 5–10 phút hỏi đáp.
> Lời nói viết sẵn để tập cho quen. Khi lên trình bày **không đọc nguyên văn**: nhớ ý in đậm, nói bằng lời của mình.

## Cách bài nói được dựng

Cả bài đi theo **một sơ đồ luồng nghiệp vụ 6 bước**, giới thiệu ở slide 2 và luôn hiện ở thanh trên cùng màn hình:

```
① Nhập hàng → ② Kho & hạn dùng → ③ Chính sách bán → ④ Bán hàng → ⑤ Đổi trả → ⑥ Chốt ca & báo cáo
```

Mỗi khi sang bước mới, người nói **mở đầu bằng tên bước** ("Sang bước 4, bán hàng…"), để người nghe luôn biết mình đang ở đâu trên sơ đồ. Nhân vật chỉ gọi theo **vai trò**: nhân viên kho, quản lý, thu ngân, khách hàng.

## Phân vai gợi ý (5 người)

| Người | Phần | Slide | Thời gian |
|---|---|---|---|
| Người 1 | Mở đầu, sơ đồ luồng tổng, tổng kết | 1, 2, 14 | 3,5 phút |
| Người 2 | Bước 1–2: nhập hàng, kho, kiểm kho, xuất hủy & tester, hàng trên kệ | 3 – 7 | 4,5 phút |
| Người 3 | Bước 3 và phần khuyến mãi 4 tầng | 8, 10 | 2,5 phút |
| Người 4 | Bước 4–5: mở ca & lập đơn, thanh toán, đổi trả | 9, 11, 12 | 3 phút |
| Người 5 | Bước 6: chốt ca & báo cáo; điều phối hỏi đáp | 13 | 1,5 phút |

Khi chuyển người, người trước nói **câu nối** (ghi cuối mỗi slide), người sau bắt đầu ngay, không chào lại.

## Con số cần nhớ

| Con số | Ý nghĩa |
|---|---|
| 3 lô kem chống nắng | Một lô hết hạn (2 tuýp), một lô cận hạn (1 tuýp), một lô mới nhập (24 tuýp) |
| 27 / 25 | Tuýp trong kho / tuýp bán được |
| 1.375.000 → 1.001.690 | Đơn mẫu trước và sau 4 tầng khuyến mãi |
| 327.656 | Tiền hoàn khi khách trả chai sữa rửa mặt bị lỗi (giá niêm yết 385.000) |
| 768.240 < 800.000 | Lý do khách phải trả lại túi tote |
| −20.000 | Chênh lệch tiền két khi chốt ca |

---

## Slide 1 · Bìa — Người 1 (≈ 1 phút)

**Lời nói**

> Em chào thầy cô và các bạn. Nhóm em xây dựng hệ thống quản lý cho một cửa hàng mỹ phẩm.
>
> Mỹ phẩm khác quần áo ở một điểm: **có hạn dùng**. Cùng một loại kem chống nắng, nhưng lần nhập tháng trước và lần nhập hôm nay có hạn khác nhau. Vì vậy hệ thống phải **quản lý theo từng lô**, bán **lô hết hạn trước**, và **mọi lần hàng tăng hay giảm đều có lịch sử**.
>
> Thay vì liệt kê chức năng, nhóm em sẽ đi theo **một tuýp kem chống nắng** qua toàn bộ cửa hàng.

**Câu nối:** "Trước hết, đây là toàn cảnh đường đi của tuýp kem."

## Slide 2 · Sơ đồ luồng tổng — Người 1 (≈ 2 phút)

**Lời nói**

> Đây là toàn cảnh nghiệp vụ. Đọc **từ trái sang phải** là **sáu bước**: nhập hàng, quản lý kho và hạn dùng, cài chính sách bán hàng, bán hàng, đổi trả, và cuối ngày là chốt ca, báo cáo.
>
> Đọc **từ trên xuống** là **ai làm gì**:
> - **Chủ cửa hàng**, tức Admin, tạo tài khoản và phân quyền cho nhân viên ngay từ đầu, và là người duy nhất xem **lợi nhuận** ở cuối.
> - **Nhân viên kho** lo hai bước đầu: nhập hàng, kiểm kho, lập phiếu hủy và tester.
> - **Quản lý** duyệt những việc làm giảm hàng trong kho, cài chính sách bán hàng, xác nhận chốt ca và xem báo cáo doanh thu.
> - **Thu ngân** mở ca, bán hàng, nhận đổi trả, chốt ca.
> - **Khách hàng** mua hàng và có khi mang hàng đến trả.
>
> Hàng cuối cùng là **những việc hệ thống tự làm**: tạo lô, cảnh báo hàng cận hạn, chọn lô để bán, tính khuyến mãi, trừ tồn, cộng điểm, tính tiền két. Đây là phần giá trị nhất của phần mềm.
>
> Thanh trên cùng màn hình chính là sáu bước này, nên lúc nào thầy cô cũng thấy nhóm em đang nói tới bước nào.

**Thao tác:** dùng tay chỉ theo hàng tiêu đề từ trái sang phải, rồi chỉ hàng "Hệ thống tự làm".

**Câu nối:** "Bắt đầu từ bước 1, xin mời bạn [Người 2]."

## Slide 3 · Bước 1 · Nhập hàng — Người 2 (≈ 50 giây)

**Lời nói**

> Bước 1, **nhập hàng**. Nhân viên kho nhận hàng từ nhà cung cấp, đếm thực tế rồi lập phiếu nhập. **Mỗi dòng phiếu là một lô**: mã lô in trên bao bì, hạn dùng, số lượng, giá nhập.
>
> Phiếu có thể lưu nháp; **chỉ khi xác nhận thì tồn kho mới tăng**. Hàng đã hết hạn thì không cho nhập.
>
> Một chi tiết nhỏ: nếu lô mới về có hạn **sớm hơn** lô đang bày trên kệ, hệ thống nhắc **"đưa lô mới ra kệ trước"**. Lát nữa em sẽ giải thích vì sao.

## Slide 4 · Bước 2 · Tồn kho & hạn dùng — Người 2 (≈ 50 giây)

**Lời nói**

> Sang bước 2, **theo dõi kho**. Tra kem chống nắng, nhân viên kho thấy **ba lô, ba trạng thái**: một lô đã hết hạn, một lô chỉ còn 46 ngày, một lô vừa nhập.
>
> Trong kho có 27 tuýp, nhưng **chỉ 25 tuýp bán được**, vì hàng hết hạn không bao giờ được bán. Lô cận hạn được tô vàng để ưu tiên bán trước. Thu ngân chỉ thấy con số 25.

## Slide 5 · Bước 2 · Kiểm kho — Người 2 (≈ 1 phút)

**Lời nói**

> Cũng trong bước 2 là **kiểm kho**. Khi bắt đầu kiểm, hệ thống **chụp lại số tồn** và **khóa các lô đang kiểm**, để quầy không bán hàng từ lô đang đếm.
>
> Lô mới nhập sổ sách ghi 24, nhưng chỉ đếm được 23: **một tuýp đã bị bóc ra làm hàng thử mà không ai ghi lại**.
>
> Tồn kho **chỉ thay đổi khi quản lý duyệt**. Từ chuyện này, nhóm em rút ra: **hàng rời kho mà không qua bán thì phải có phiếu**.

**Câu nối:** "Đó là lý do có phiếu xuất hủy và tester."

## Slide 6 · Bước 2 · Xuất hủy & tester — Người 2 (≈ 1 phút)

**Lời nói**

> Nhóm em gom ba trường hợp hàng rời kho mà không bán vào **một loại phiếu**: tiêu hủy, đưa ra quầy làm tester, và trả nhà cung cấp. Chung một quy trình: **lập phiếu thì hàng bị cách ly ngay**, quản lý **duyệt thì mới trừ tồn**, từ chối thì trả lại.
>
> Ví dụ quản lý duyệt hai phiếu: hủy 2 tuýp hết hạn, và lấy 1 tuýp làm tester.
>
> Riêng tester: **duyệt xong là hệ thống hết trách nhiệm**. Dùng hết hay hết hạn thì nhân viên tự bỏ, không lập phiếu hủy nữa. Tester có hai nguồn: hãng tặng thì nhập như một mã không bán, giá 0 đồng; cửa hàng tự mở thì chi phí bằng giá vốn của lô.

## Slide 7 · Bước 2 · Hàng trên kệ — Người 2 (≈ 1 phút)

**Lời nói**

> Đây là câu hỏi nhóm em đã tranh luận: **có cần bảng riêng cho hàng trưng bày không?**
>
> Tình huống: kệ đang bày lô hạn 12/2027, kho vừa về lô hạn 06/2027. Hệ thống bán theo FEFO sẽ trừ lô 06/2027, nhưng khách lại cầm chai 12/2027 trên kệ.
>
> Mấu chốt là **mã vạch chỉ chứa mã sản phẩm, không chứa số lô**, nên hệ thống không biết khách cầm chai nào. Thêm bảng trưng bày chỉ giúp ghi đúng việc bán sai lô, còn lô 06/2027 vẫn nằm trong kho chờ hết hạn.
>
> Vì vậy nhóm em chọn **xoay kệ**: lô hạn sớm luôn được bày ra trước. Hệ thống hỗ trợ bằng cảnh báo lúc nhập hàng như ở bước 1. Nếu vẫn lệch, kiểm kho theo lô sẽ sửa lại.

**Câu nối:** "Hàng trong kho đã sẵn sàng. Sang bước 3, chuẩn bị chính sách bán hàng, xin mời bạn [Người 3]."

## Slide 8 · Bước 3 · Chính sách bán hàng — Người 3 (≈ 1 phút)

**Lời nói**

> Bước 3, **chính sách bán hàng**. Quản lý cài sẵn, thu ngân chỉ áp dụng chứ không sửa được.
>
> Có **hạng thẻ** theo tổng chi tiêu: Đồng, Bạc 3%, Vàng 5%, Kim Cương 8%. Có **điểm thưởng**: trả 10 nghìn được 1 điểm, 1 điểm dùng được 100 đồng, tối đa nửa giá trị đơn.
>
> Và **tám loại khuyến mãi** phổ biến ở cửa hàng mỹ phẩm, xếp vào **bốn tầng**: theo món, theo khách, theo hóa đơn, và cuối cùng là điểm. Bốn tầng này trả lời câu hỏi khó nhất: **nhiều khuyến mãi cùng áp vào một đơn thì tính cái nào trước**. Em sẽ minh họa ngay ở bước bán hàng.

**Câu nối:** "Sang bước 4, bán hàng, xin mời bạn [Người 4]."

## Slide 9 · Bước 4 · Mở ca & lập đơn — Người 4 (≈ 1 phút)

**Lời nói**

> Bước 4, **bán hàng**. Thu ngân phải **mở ca** trước, khai báo tiền lẻ trong két là 2 triệu. **Chưa mở ca thì chưa bán được**, và mọi hóa đơn đều gắn vào ca để cuối ngày đối chiếu.
>
> Một khách hạng Bạc mua 2 tuýp kem chống nắng và 1 chai sữa rửa mặt. Thu ngân **chỉ quét mã**, hệ thống tự chọn lô: lô hết hạn **bỏ qua**, lô cận hạn còn 1 tuýp **lấy 1**, tuýp còn lại **tự tách** sang lô mới. Đó là FEFO, hết hạn trước xuất trước.
>
> Lúc này đơn mới là nháp, tồn kho chưa bị trừ.

**Câu nối:** "Đơn này dính nhiều khuyến mãi cùng lúc, xin mời bạn [Người 3] giải thích cách tính."

## Slide 10 · Bước 4 · Khuyến mãi 4 tầng — Người 3 (≈ 1,5 phút)

**Lời nói**

> Đơn này gặp năm thứ cùng lúc: giảm 20% hàng La Roche-Posay, hạng Bạc, bậc thang hóa đơn, voucher THU2026, và khách muốn dùng 200 điểm.
>
> Hệ thống tính theo **đúng một thứ tự**:
> - **Tầng 1, theo món**: kem chống nắng giảm 20%. Mỗi món chỉ hưởng một khuyến mãi ở tầng này.
> - **Tầng 2, theo khách**: hạng Bạc giảm thêm 3%.
> - **Tầng 3, theo hóa đơn**: chỉ được **một khoản giảm**. Bậc thang giảm 120 nghìn, voucher chỉ được 50 nghìn vì có mức giảm tối đa, nên hệ thống **lấy bậc thang**. Voucher không dùng thì **không bị trừ lượt**, khách giữ mã cho lần sau.
> - **Tầng 4, điểm**: 200 điểm là 20 nghìn.
>
> Từ 1 triệu 375 còn **1 triệu 1 nghìn 690**. Sau tầng 2 đơn vẫn trên 800 nghìn, nên khách được **tặng túi tote**.
>
> Điểm quan trọng nhất: mọi khoản giảm được **chia xuống từng dòng hàng và lưu lại**. Nhờ vậy lúc trả hàng không phải tính lại khuyến mãi.

**Thao tác:** chỉ lần lượt từng thanh trên biểu đồ.

**Câu nối:** "Giá đã có, xin mời bạn [Người 4] tiếp phần thanh toán."

## Slide 11 · Bước 4 · Thanh toán — Người 4 (≈ 50 giây)

**Lời nói**

> Khách đưa 1 triệu 10 nghìn tiền mặt, thu ngân bấm hoàn tất. Hệ thống kiểm tra lại lô một lần cuối, rồi **trong cùng một lần ghi**: trừ tồn đúng lô, kể cả túi tote tặng kèm; ghi lịch sử kho; chuyển đơn sang hoàn tất và cộng tiền vào ca; trừ 200 điểm, cộng 100 điểm mới. Tiền thừa 8.310 đồng.
>
> Nếu một bước lỗi, ví dụ lô vừa bị khóa kiểm kho, thì **không bước nào được ghi**. Không có chuyện trừ kho rồi mà đơn chưa hoàn tất.

**Câu nối:** "Hai ngày sau, khách quay lại cửa hàng. Sang bước 5, đổi trả."

## Slide 12 · Bước 5 · Đổi trả — Người 4 (≈ 1,5 phút)

**Lời nói**

> Bước 5, **đổi trả**. Mỹ phẩm dùng trên da, đã mở thì không bán lại được, nên nhóm em tách rõ hai trường hợp:
> - **Hàng lỗi**: **hoàn tiền** hoặc đổi. Không có trường hợp giao nhầm, vì thu ngân quét mã từng món khi thanh toán.
> - **Khách đổi ý**, ví dụ không hợp da: **chỉ đổi hàng**, và chỉ khi còn nguyên seal trong 7 ngày. Đã mở seal thì từ chối. Khi đổi, tổng hàng mới **phải lớn hơn hoặc bằng** giá trị hàng cũ; thiếu thì khách chọn thêm. **Không bù voucher.**
>
> Ví dụ khách mang chai sữa rửa mặt **hỏng vòi bơm** đến trả, đây là hàng lỗi. Hoàn **327.656 đồng**, không phải giá niêm yết 385 nghìn, vì đó là số tiền khách thực trả cho chai này sau các khoản giảm. Điểm đã dùng không hoàn và **không bù voucher**.
>
> Còn **túi tote**: trả chai này xong, phần khách giữ lại chỉ còn 768 nghìn, dưới mức 800 nghìn, nên khách trả lại tote, hoặc bị trừ 15 nghìn.

**Câu nối:** "Cuối ngày là bước 6, xin mời bạn [Người 5]."

## Slide 13 · Bước 6 · Chốt ca & báo cáo — Người 5 (≈ 1,5 phút)

**Lời nói**

> Bước 6, **chốt ca và báo cáo**.
>
> Cuối ca, hệ thống tự tính tiền mặt lẽ ra phải có: **đầu ca, cộng tiền mặt thu vào, trừ tiền mặt đã hoàn**. Thu ngân đếm két, thiếu 20 nghìn. Hệ thống không chặn, nhưng **bắt buộc ghi lý do** và quản lý xác nhận. Cuối tháng, ca nào thiếu tiền và vì sao đều có dấu vết.
>
> Mọi thứ ghi ở các bước trước **đổ về báo cáo**. Quản lý xem được doanh thu; **lợi nhuận gộp chỉ chủ cửa hàng xem**. Số liệu lấy từ tiền hàng, giảm giá, hàng trả và giá vốn từng lô, tính thẳng từ chứng từ, không có bảng tổng hợp riêng nên không lệch số. Đơn tính theo **ngày hoàn tất**, hàng trả tính theo **ngày trả**, nên báo cáo tháng trước đã chốt không tự thay đổi.

**Câu nối:** "Xin mời bạn [Người 1] tổng kết."

## Slide 14 · Tổng kết — Người 1 (≈ 30 giây)

**Lời nói**

> Đi hết sáu bước, có năm điều xuyên suốt:
> 1. Quản lý **theo lô và hạn dùng**, bán theo FEFO.
> 2. **Mọi thay đổi tồn kho có một dòng lịch sử**.
> 3. **Hàng rời kho không qua bán phải có phiếu và người duyệt**.
> 4. **Khuyến mãi tính theo bốn tầng cố định**, lưu xuống từng dòng hàng.
> 5. **Chứng từ đã hoàn tất thì không sửa**, và trả hàng không tính lại khuyến mãi.
>
> Em xin cảm ơn thầy cô và các bạn đã lắng nghe. Nhóm em sẵn sàng nhận câu hỏi.

---

## Chuẩn bị hỏi đáp

| Câu hỏi có thể gặp | Gợi ý trả lời |
|---|---|
| Sao không làm bảng riêng cho hàng trưng bày? | Mã vạch không chứa số lô nên hệ thống không biết khách cầm chai nào. Bảng trưng bày chỉ ghi đúng việc bán sai lô, không ngăn hàng hết hạn. Xoay kệ + cảnh báo lúc nhập + kiểm kho theo lô giải quyết tận gốc. |
| Combo sao không làm thành một mặt hàng riêng? | Mỗi món trong combo cần gắn một lô để theo dõi hạn dùng. Làm combo là khuyến mãi thì mỗi món vẫn là một dòng hàng thường; giá bộ chia về từng món theo tỷ lệ giá niêm yết, nên thuế và trả hàng tính đúng. |
| Nhiều khuyến mãi cùng lúc sao không cho cộng dồn tự do? | Cộng dồn tự do khó kiểm soát lỗ và khó giải thích cho khách. Bốn tầng cố định: mỗi món một khuyến mãi, mỗi đơn một khoản giảm cấp đơn, nên luôn biết khách được giảm bao nhiêu và vì sao. |
| Voucher thua bậc thang thì khách có thiệt không? | Không. Hệ thống chọn cái có lợi hơn cho khách, và voucher không dùng thì không bị trừ lượt. |
| Sao trả hàng không tính lại khuyến mãi? | Mọi khoản giảm đã chia xuống từng dòng và lưu lúc bán. Trả hàng lấy đúng số tiền của dòng đó. Tính lại vừa phức tạp vừa dễ sai nếu chương trình đã thay đổi. Hệ thống chỉ xét lại điều kiện nhận quà. |
| Sao đổi ý không được hoàn tiền? | Mỹ phẩm dùng trên da, đã mở thì không bán lại được. Lỗi của cửa hàng thì hoàn tiền; ý muốn của khách thì chỉ đổi hàng còn nguyên seal. |
| Tester duyệt xong không theo dõi nữa, có mất kiểm soát không? | Giá trị tester nhỏ. Kiểm soát nằm ở lúc xuất: phải có phiếu, quản lý duyệt, chi phí được ghi lại. Theo dõi từng chai trên quầy tốn công hơn giá trị mang lại. |
| Thanh toán lỗi giữa chừng thì sao? | Mọi việc khi hoàn tất đơn nằm trong một giao dịch cơ sở dữ liệu; lỗi ở bước nào thì không bước nào được ghi. |
| Hàng trả tháng sau thì báo cáo tháng trước có đổi không? | Không. Hàng trả tính vào ngày trả, như "hàng bán bị trả lại" trong kế toán. |
| Ai xem được lợi nhuận? | Chỉ chủ cửa hàng (Admin). Quản lý xem doanh thu nhưng không xem giá vốn và lợi nhuận. Phân quyền theo vai trò, mỗi vai chỉ thấy phần việc của mình. |
| Hệ thống có cho nhiều chi nhánh không? | Phạm vi đồ án là một cửa hàng. Muốn mở rộng thì thêm mã cửa hàng vào các bảng kho và bán hàng. |

## Lưu ý khi lên trình bày

- **Tập ít nhất 2 lần** với đồng hồ; mỗi người giữ đúng thời gian của mình.
- Mỗi lần sang bước mới, **nói tên bước** và để ý thanh trên cùng đổi màu sang bước đó.
- Khi nói số tiền, có thể **đọc tròn** ("khoảng một triệu", "ba trăm hai mươi bảy nghìn"); số chính xác đã có trên slide.
- Bị hỏi câu chưa chuẩn bị: nói **"nhóm em đã cân nhắc …, và chọn … vì …"**; nếu chưa tính tới thì nhận và ghi lại.
- Kiểm tra trước: máy chiếu mở được trang; có mạng để tải phông chữ (không có mạng trang vẫn chạy, chỉ khác phông chữ).
