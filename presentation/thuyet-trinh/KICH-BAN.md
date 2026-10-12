# KỊCH BẢN THUYẾT TRÌNH — SE100

> Đi kèm trang trình chiếu [index.html](index.html) (17 slide). Mở trang, bấm **F** để toàn màn hình, **→** để chuyển slide, **N** để bật ghi chú ngắn trên màn hình người nói.
> Thời lượng dự kiến: **13–15 phút** trình bày + 5–10 phút hỏi đáp.
> Lời nói viết sẵn để đọc quen, **không đọc nguyên văn khi lên trình bày**: nhớ ý chính in đậm, nói bằng lời của mình.

## Phân vai gợi ý (5 người)

| Người | Phần | Slide | Thời gian |
|---|---|---|---|
| Người 1 | Mở đầu, tổng kết, điều phối hỏi đáp | 1, 2, 17 | 2 phút |
| Người 2 | Kho: nhập hàng → trưng bày | 3 – 7 | 4 phút |
| Người 3 | Khách hàng & khuyến mãi | 8, 11 | 2,5 phút |
| Người 4 | Bán hàng: mở ca → đổi trả | 9, 10, 12 – 14 | 3,5 phút |
| Người 5 | Chốt ca, báo cáo | 15, 16 | 1,5 phút |

Khi chuyển người nói, người trước nói câu nối (ghi ở cuối mỗi slide), người sau bắt đầu ngay, không cần chào lại.

## Nhân vật và con số cần nhớ

| Ai | Vai trò |
|---|---|
| Chị Mai | Quản lý: duyệt kiểm kho, xuất hủy, tester; cài khuyến mãi |
| Anh Tâm | Nhân viên kho |
| Anh Khoa | Thu ngân |
| Chị Hà | Khách hạng Bạc, 320 điểm |

| Con số | Ý nghĩa |
|---|---|
| 3 lô Anthelios | LRP24C02 hết hạn (2 tuýp) · LRP24K07 cận hạn (1) · LRP25B12 mới nhập (24) |
| 495.000 / 385.000 | Giá Anthelios / CeraVe |
| 1.375.000 → 1.001.690 | Đơn của chị Hà trước và sau 4 tầng khuyến mãi |
| 327.656 | Tiền hoàn khi trả chai CeraVe lỗi |
| 768.240 < 800.000 | Lý do chị Hà phải trả lại túi tote |
| −20.000 | Chênh lệch tiền két khi chốt ca |

---

## Slide 1 · Bìa — Người 1 (≈ 40 giây)

**Lời nói**

> Em chào thầy cô và các bạn. Nhóm em xây dựng hệ thống quản lý cho một cửa hàng mỹ phẩm.
>
> Thay vì đọc danh sách chức năng, hôm nay nhóm em kể **câu chuyện của một tuýp kem chống nắng**: nó về kho thế nào, nằm trên kệ ra sao, được bán, có khi bị trả lại, và cuối ngày trở thành con số trên báo cáo. Ở mỗi chặng, nhóm em sẽ cho thấy **ai làm gì** và **hệ thống ghi lại những gì**.

**Câu nối:** "Trước hết, vì sao mỹ phẩm lại cần quản lý khác với một shop quần áo?"

## Slide 2 · Vì sao quản lý theo lô — Người 1 (≈ 1 phút)

**Lời nói**

> Điểm khác biệt lớn nhất là **hạn dùng**. Cùng một mã kem chống nắng, nhưng tuýp về tháng trước và tuýp về hôm nay có hạn khác nhau. Bán nhầm tuýp hết hạn cho khách là mất uy tín ngay.
>
> Vì vậy hệ thống không chỉ đếm "còn bao nhiêu tuýp", mà đếm **"còn bao nhiêu tuýp của từng lô"**. Bán thì lấy lô **hết hạn trước, xuất trước**, gọi là FEFO. Và mọi lần hàng tăng hay giảm đều có **một dòng lịch sử** đi kèm.
>
> Câu chuyện có bốn người làm: chủ cửa hàng, chị Mai quản lý, anh Tâm ở kho, anh Khoa thu ngân. Và một khách quen là chị Hà. Mỗi người chỉ thấy phần việc của mình, nhờ phân quyền theo vai trò.

**Câu nối:** "Câu chuyện bắt đầu ở kho, xin mời bạn [Người 2]."

## Slide 3 · Nhập hàng — Người 2 (≈ 50 giây)

**Lời nói**

> Ngày 28/9, anh Tâm nhận hàng từ nhà cung cấp Hoàng Anh. Anh đếm thực tế rồi nhập vào phiếu nhập, **mỗi dòng là một lô**: mã lô in trên bao bì, hạn dùng, số lượng, giá nhập.
>
> Phiếu có thể lưu nháp; **chỉ khi bấm xác nhận thì tồn kho mới tăng**. Hàng đã hết hạn thì hệ thống không cho nhập.
>
> Có một chi tiết nhỏ nhưng quan trọng: nếu lô mới về lại có hạn **sớm hơn** lô đang bày trên kệ, hệ thống nhắc **"đưa lô mới ra kệ trước"**. Lát nữa em sẽ giải thích vì sao.

**Thao tác:** chỉ vào cột "Mã lô" và "Hạn dùng" trên bảng.

## Slide 4 · Tồn kho & hạn dùng — Người 2 (≈ 50 giây)

**Lời nói**

> Đây là màn hình kho dùng nhiều nhất. Sản phẩm Anthelios có **ba lô, ba trạng thái**: một lô đã hết hạn từ 01/10, một lô chỉ còn 46 ngày, một lô vừa nhập.
>
> Trong kho có 27 tuýp, nhưng **chỉ 25 tuýp bán được**, vì 2 tuýp hết hạn không bao giờ được bán. Lô cận hạn được tô vàng để ưu tiên bán trước. Thu ngân chỉ thấy con số 25.

## Slide 5 · Kiểm kho — Người 2 (≈ 1 phút)

**Lời nói**

> Ngày 3/10, anh Tâm kiểm kệ chống nắng. Lúc bắt đầu, hệ thống **chụp lại số tồn** và **khóa các lô đang kiểm**, để quầy không bán hàng từ lô đang đếm.
>
> Kết quả: lô LRP25B12 sổ sách ghi 24 nhưng chỉ đếm được 23. Một tuýp đã **bị bóc ra làm hàng thử mà không ai ghi lại**.
>
> Tồn kho **chỉ thay đổi khi chị Mai duyệt**. Duyệt xong, lô còn 23 và có thêm một dòng lịch sử "điều chỉnh kiểm kho".
>
> Bài học từ đây: **hàng rời kho mà không qua bán thì phải có phiếu**.

**Câu nối:** "Đó là lý do nhóm em thiết kế phiếu tester."

## Slide 6 · Xuất hủy & tester — Người 2 (≈ 1 phút)

**Lời nói**

> Nhóm em gom ba trường hợp hàng rời kho mà không bán vào **một loại phiếu**: tiêu hủy, đưa ra quầy làm tester, và trả nhà cung cấp. Cùng một quy trình: **lập phiếu thì hàng bị cách ly ngay**, chị Mai **duyệt thì mới trừ tồn**, từ chối thì trả lại.
>
> Sáng 5/10, chị Mai duyệt hai phiếu: hủy 2 tuýp hết hạn, và lấy 1 tuýp làm tester.
>
> Riêng tester, nhóm em chọn cách đơn giản: **duyệt xong là hệ thống hết trách nhiệm**. Tester dùng hết hay hết hạn thì nhân viên tự bỏ, không cần lập phiếu hủy. Tester có hai nguồn: hãng tặng thì nhập như một mã không bán, giá 0 đồng; cửa hàng tự mở thì chi phí bằng giá vốn của lô.

## Slide 7 · Trưng bày & xoay kệ — Người 2 (≈ 1 phút)

**Lời nói**

> Đây là câu hỏi nhóm em đã tranh luận: **có cần một bảng riêng cho hàng trưng bày không?**
>
> Tình huống: kệ đang bày lô hạn 12/2027, kho vừa về lô hạn 06/2027. Hệ thống bán theo FEFO sẽ trừ lô 06/2027, nhưng khách lại cầm chai 12/2027 trên kệ.
>
> Mấu chốt là **mã vạch chỉ chứa mã sản phẩm, không chứa số lô**. Hệ thống không có cách nào biết khách cầm chai nào. Thêm bảng trưng bày chỉ giúp ghi đúng việc bán sai lô, còn lô 06/2027 vẫn nằm trong kho chờ hết hạn.
>
> Nên nhóm em chọn **xoay kệ**: lô hạn sớm luôn được bày ra trước. Hệ thống hỗ trợ bằng cảnh báo lúc nhập hàng như em nói ở đầu. Nếu vẫn lệch thì kiểm kho theo lô sẽ sửa lại.

**Câu nối:** "Hàng đã sẵn sàng. Trước khi mở bán, quản lý cần cài chính sách bán hàng, xin mời bạn [Người 3]."

## Slide 8 · Chuẩn bị chính sách bán hàng — Người 3 (≈ 1 phút)

**Lời nói**

> Chị Mai cài trước ba thứ, thu ngân chỉ áp dụng chứ không sửa được.
>
> Một là **hạng thẻ** theo tổng chi tiêu: Đồng, Bạc 3%, Vàng 5%, Kim Cương 8%.
> Hai là **điểm**: trả 10 nghìn được 1 điểm, 1 điểm dùng được 100 đồng, tối đa nửa giá trị đơn.
> Ba là **khuyến mãi**. Nhóm em chọn tám loại phổ biến ở cửa hàng mỹ phẩm: giảm theo sản phẩm, giờ vàng, mua X tặng Y, combo, ưu đãi theo hạng, bậc thang theo hóa đơn, voucher và quà theo hóa đơn.
>
> Tám loại này được xếp vào **bốn tầng**. Bốn tầng là câu trả lời của nhóm em cho câu hỏi khó nhất: khi nhiều khuyến mãi cùng áp vào một đơn thì tính cái nào trước. Em sẽ quay lại phần này khi chị Hà thanh toán.

**Câu nối:** "Giờ là lúc mở cửa bán hàng, xin mời bạn [Người 4]."

## Slide 9 · Mở ca — Người 4 (≈ 30 giây)

**Lời nói**

> 1 giờ chiều, anh Khoa đếm tiền lẻ trong két, nhập **2 triệu** và mở ca. **Chưa mở ca thì chưa bán được hàng.** Mọi hóa đơn và phiếu trả đều gắn vào ca, để cuối ngày đối chiếu tiền trong két.

## Slide 10 · Bán hàng FEFO — Người 4 (≈ 50 giây)

**Lời nói**

> Chị Hà mua 2 tuýp Anthelios và 1 chai CeraVe. Anh Khoa chỉ việc **quét mã**, hệ thống tự chọn lô.
>
> Lô hết hạn: **bỏ qua**. Lô cận hạn chỉ còn 1 tuýp: **lấy 1**. Tuýp còn lại **tự tách** sang lô mới. Hóa đơn có hai dòng Anthelios, mỗi dòng gắn một lô.
>
> Lúc này đơn mới là **nháp**, tồn kho chưa bị trừ.

**Câu nối:** "Đơn này dính khá nhiều khuyến mãi cùng lúc, xin mời bạn [Người 3] giải thích cách hệ thống tính."

## Slide 11 · Khuyến mãi 4 tầng — Người 3 (≈ 1,5 phút)

**Lời nói**

> Đơn của chị Hà gặp năm thứ cùng lúc: giảm 20% hàng La Roche-Posay, hạng Bạc, bậc thang hóa đơn, voucher THU2026, và chị muốn dùng 200 điểm.
>
> Hệ thống tính theo **đúng một thứ tự**:
> - **Tầng 1, theo món**: Anthelios giảm 20%, còn 1.177.000. Mỗi món chỉ hưởng một khuyến mãi ở tầng này.
> - **Tầng 2, theo khách**: hạng Bạc giảm 3%, còn 1.141.690.
> - **Tầng 3, theo hóa đơn**: chỉ được **một khoản giảm**. Bậc thang giảm 120 nghìn, voucher THU2026 chỉ được 50 nghìn vì bị giới hạn mức giảm tối đa, nên hệ thống **lấy bậc thang**. Voucher không được dùng thì **không bị trừ lượt**, chị Hà giữ mã cho lần sau. Còn 1.021.690.
> - **Tầng 4, điểm**: 200 điểm là 20 nghìn. Chị Hà trả **1.001.690**.
>
> Thêm một điều: sau tầng 2, đơn vẫn trên 800 nghìn, nên chị được **tặng túi tote**.
>
> Điểm quan trọng nhất: mọi khoản giảm được **chia xuống từng dòng hàng và lưu lại**. Nhờ vậy lúc trả hàng, hệ thống không phải tính lại khuyến mãi.

**Thao tác:** chỉ lần lượt từng thanh trên biểu đồ bậc thang.

**Câu nối:** "Giá đã có, xin mời bạn [Người 4] tiếp phần thanh toán."

## Slide 12 · Thanh toán — Người 4 (≈ 40 giây)

**Lời nói**

> Chị Hà đưa 1 triệu 10 nghìn tiền mặt. Anh Khoa bấm hoàn tất. Hệ thống kiểm tra lại lô một lần cuối, rồi **trong cùng một lần ghi**: trừ tồn đúng lô, kể cả túi tote; ghi lịch sử kho; chuyển đơn sang hoàn tất và cộng tiền vào ca; trừ 200 điểm, cộng 100 điểm mới, chị Hà còn 220 điểm. Tiền thừa 8.310 đồng.
>
> Nếu một bước bị lỗi, ví dụ lô vừa bị khóa kiểm kho, thì **không bước nào được ghi**. Không có chuyện trừ kho rồi mà đơn chưa hoàn tất.

## Slide 13 · Đổi trả hàng lỗi — Người 4 (≈ 1 phút)

**Lời nói**

> Hai ngày sau, chị Hà mang chai CeraVe bị **hỏng vòi bơm** đến trả. Đây là lỗi của cửa hàng nên chị **được hoàn tiền**.
>
> Nhưng hoàn bao nhiêu? Không phải giá niêm yết 385 nghìn, mà là **327.656 đồng**, đúng số tiền chị thực trả cho chai đó sau mọi khoản giảm. Con số này đã được lưu sẵn trên hóa đơn.
>
> Chai lỗi về đúng lô cũ và bị cách ly. Điểm đã dùng thì không hoàn, nhưng vì là hàng lỗi nên cửa hàng **bù cho chị một voucher 6.542 đồng**, bằng phần điểm đã dùng cho chai này. Đồng thời thu hồi 32 điểm đã tích.
>
> Còn **túi tote**: trả CeraVe xong, phần chị giữ lại chỉ còn 768 nghìn, dưới mức 800 nghìn. Nên chị mang tote trả lại thì được hoàn đủ; giữ tote thì bị trừ 15 nghìn.

## Slide 14 · Khách đổi ý — Người 4 (≈ 50 giây)

**Lời nói**

> Trường hợp khác: khách mua về thấy **không hợp da**. Đó không phải lỗi của cửa hàng, và mỹ phẩm đã mở thì không bán lại được.
>
> Nên quy tắc là: **đổi ý thì chỉ đổi hàng, không hoàn tiền**, và chỉ khi **còn nguyên seal**, trong 7 ngày. Đã mở seal thì từ chối.
>
> Đổi sang món đắt hơn thì khách trả thêm. Đổi sang món rẻ hơn thì **phần dư không được lấy lại**, cũng không đổi thành voucher. Hệ thống hiện cảnh báo trước để khách biết.

**Câu nối:** "Cuối ngày, thu ngân chốt ca, xin mời bạn [Người 5]."

## Slide 15 · Chốt ca — Người 5 (≈ 40 giây)

**Lời nói**

> 9 rưỡi tối, anh Khoa chốt ca. Hệ thống tự tính tiền mặt lẽ ra phải có: **tiền đầu ca, cộng tiền mặt thu vào, trừ tiền mặt đã hoàn**, ra 10.276.500. Anh Khoa đếm được 10.256.500, **thiếu 20 nghìn**.
>
> Hệ thống không chặn, nhưng **bắt buộc ghi lý do**, và quản lý xác nhận. Cuối tháng, ca nào thiếu tiền và vì sao đều có dấu vết.

## Slide 16 · Báo cáo — Người 5 (≈ 50 giây)

**Lời nói**

> Mọi chặng trước đều đổ về đây. Báo cáo **tính thẳng từ hóa đơn, phiếu trả, lô và phiếu hủy**, không có bảng tổng hợp riêng nên không bị lệch số.
>
> Hai nguyên tắc: đơn tính theo **ngày hoàn tất**, hàng trả tính theo **ngày trả**. Như vậy báo cáo tháng trước đã chốt sẽ không tự thay đổi khi khách trả hàng tháng này.
>
> Lợi nhuận gộp dùng **giá vốn của đúng lô đã bán**, vì mỗi dòng hóa đơn đã ghi lô và giá vốn tại thời điểm bán.

**Câu nối:** "Xin mời bạn [Người 1] tổng kết."

## Slide 17 · Tổng kết — Người 1 (≈ 40 giây)

**Lời nói**

> Nếu chỉ nhớ năm điều về hệ thống của nhóm em, thì đó là:
> 1. Quản lý **theo lô và hạn dùng**, bán theo FEFO.
> 2. **Mọi thay đổi tồn kho có một dòng lịch sử**.
> 3. **Hàng rời kho không qua bán phải có phiếu và người duyệt**.
> 4. **Khuyến mãi tính theo bốn tầng cố định**, lưu xuống từng dòng hàng.
> 5. **Chứng từ đã hoàn tất thì không sửa**; sai thì làm chứng từ ngược, và trả hàng không tính lại khuyến mãi.
>
> Em xin cảm ơn thầy cô và các bạn đã lắng nghe. Nhóm em sẵn sàng nhận câu hỏi.

---

## Chuẩn bị hỏi đáp

| Câu hỏi có thể gặp | Gợi ý trả lời |
|---|---|
| Sao không làm bảng riêng cho hàng trưng bày? | Mã vạch không chứa số lô nên hệ thống không biết khách cầm chai nào. Bảng trưng bày chỉ ghi đúng việc bán sai lô, không ngăn hàng hết hạn. Xoay kệ + cảnh báo lúc nhập + kiểm kho theo lô giải quyết tận gốc. |
| Combo sao không làm thành một mặt hàng riêng? | Mỗi món trong combo cần gắn một lô để theo dõi hạn dùng. Làm combo là khuyến mãi thì mỗi món vẫn là một dòng hàng thường. Giá bộ được chia về từng món theo tỷ lệ giá niêm yết, nên thuế và trả hàng tính đúng. |
| Nhiều khuyến mãi cùng lúc thì sao không cho cộng dồn tự do? | Cộng dồn tự do khó kiểm soát lỗ và khó giải thích cho khách. Bốn tầng cố định: mỗi món một khuyến mãi, mỗi đơn một khoản giảm cấp đơn, nên luôn biết khách được giảm bao nhiêu và vì sao. |
| Voucher thua bậc thang thì khách có thiệt không? | Không. Hệ thống tự chọn cái có lợi hơn cho khách, và voucher không dùng thì không bị trừ lượt. |
| Sao trả hàng không tính lại khuyến mãi? | Mọi khoản giảm đã được chia xuống từng dòng và lưu lại lúc bán. Trả hàng chỉ lấy đúng số tiền của dòng đó. Tính lại vừa phức tạp vừa dễ sai nếu chương trình khuyến mãi đã đổi. Hệ thống chỉ xét lại điều kiện nhận quà. |
| Sao đổi ý không được hoàn tiền? | Mỹ phẩm dùng trên da, đã mở thì không bán lại được. Lỗi của cửa hàng thì hoàn tiền; ý muốn của khách thì chỉ đổi hàng còn nguyên seal. |
| Tester duyệt xong không theo dõi nữa, có mất kiểm soát không? | Giá trị tester nhỏ. Kiểm soát nằm ở lúc xuất: phải có phiếu và quản lý duyệt, chi phí được ghi lại. Theo dõi từng chai trên quầy tốn công hơn giá trị nó mang lại. |
| Thanh toán lỗi giữa chừng thì sao? | Mọi việc khi hoàn tất đơn nằm trong một giao dịch cơ sở dữ liệu. Lỗi ở bước nào thì không bước nào được ghi. |
| Hàng trả tháng sau thì báo cáo tháng trước có đổi không? | Không. Hàng trả tính vào ngày trả, như "hàng bán bị trả lại" trong kế toán. |
| Hệ thống có cho nhiều chi nhánh không? | Phạm vi đồ án là một cửa hàng. Muốn mở rộng thì thêm mã cửa hàng vào các bảng kho và bán hàng. |

## Lưu ý khi lên trình bày

- **Tập ít nhất 2 lần** với đồng hồ; mỗi người giữ đúng thời gian của mình.
- Người không nói thì **đứng yên, nhìn vào màn hình**, sẵn sàng hỗ trợ khi hỏi đáp.
- Khi nói số tiền, **làm tròn bằng lời** ("khoảng một triệu", "ba trăm hai mươi bảy nghìn") nếu không cần chính xác; số chính xác đã có trên slide.
- Bị hỏi câu chưa chuẩn bị: nói **"nhóm em đã cân nhắc …, và chọn … vì …"**; không biết thì nhận là chưa tính tới và ghi lại.
- Kiểm tra trước: máy chiếu mở được trang, có mạng để tải phông chữ (không có mạng trang vẫn chạy, chỉ khác phông chữ).
