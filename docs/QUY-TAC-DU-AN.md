# Quy tắc dự án: Quản lý mỹ phẩm

## 1. Vai trò và trách nhiệm

| Vai trò | Trách nhiệm |
|---|---|
| **PM** | Chốt yêu cầu, chốt API contract, nghiệm thu (UAT). Khi phụ BE thì chỉ nhận task không gấp |
| **FE lead / BE lead** | Chịu trách nhiệm chất lượng và tiến độ tầng mình, review code, báo cáo cho PM |
| **Thành viên** | Mỗi tính năng có **1 cặp FE + BE** chịu trách nhiệm từ đầu đến cuối |

## 2. Yêu cầu và API contract

- Ưu tiên use case theo **Must / Should / Could / Won't**. Khi trễ thì cắt Could trước.
- Mỗi use case phải có **tiêu chí chấp nhận** đo được.
- Viết API bằng **Swagger**, có **mock server** để FE làm song song.
- **Thống nhất chung:** cấu trúc response, mã lỗi, phân trang, định dạng ngày và tiền, xác thực.
- **Contract đã chốt thì không tự ý sửa.** Muốn đổi phải có 2 lead đồng ý, báo trong nhóm và cập nhật Swagger.

## 3. Definition of Done

Task chỉ được tính là **xong** khi:

- [ ] Đã merge qua PR được duyệt
- [ ] Đạt tiêu chí chấp nhận
- [ ] Đúng API contract, đã nối API thật
- [ ] Một người khác đã test lại
- [ ] Không còn bug Critical/High
- [ ] Đã cập nhật tài liệu (nếu có thay đổi)

## 4. Giao tiếp và báo cáo

- **Bị kẹt quá 1 ngày thì báo ngay**, không chờ đến buổi họp.
- Trao đổi công việc trong nhóm chung, không nhắn riêng. Phản hồi trong vòng **24 giờ**.
- Mọi quyết định đều ghi vào **Decision log**.

**Mẫu báo cáo của lead:**

1. Đã xong (đạt DoD)
2. Đang làm (dự kiến xong ngày nào)
3. Đang kẹt hoặc rủi ro
4. Cần PM quyết định
5. Kế hoạch tới

## 5. Họp cố định

| Buổi | Khi nào | Thời lượng |
|---|---|---|
| Daily async (nhắn tin: hôm qua làm gì, hôm nay làm gì, đang kẹt gì) | Mỗi ngày, trước 22h | |
| Planning | Đầu sprint | 45 phút |
| Sync với lead | Giữa sprint | 15 phút |
| Demo và retro | Cuối sprint | 60 phút |

## 6. Quy tắc code và Git

> Chi tiết xem [QUY-TAC-GIT.md](QUY-TAC-GIT.md)

- Linter và formatter chạy tự động trước khi commit.
- Không push thẳng vào `main`.
- Đặt tên nhánh: `feature/<mã-task>-mo-ta`.
- Commit theo dạng `feat:`, `fix:`, `refactor:`, `docs:`...
- PR dưới **400 dòng**, có mô tả, cần **ít nhất 1 người duyệt**.

## 7. Kiểm thử

- **BE:** unit test cho logic chính (tồn kho, hạn sử dụng, tính tiền) và Postman collection.
- **FE:** test case theo từng use case.
- **Test chéo:** không tự test code của mình.
- Bug ghi lên board, có mức độ (Critical / High / Medium / Low) và bước tái hiện.
- PM làm UAT cuối mỗi sprint.

## 8. Bàn giao

- [ ] Code gắn tag `v1.0`
- [ ] README cài đặt, `.env.example`, seed data
- [ ] Swagger, tài liệu use case và ERD
- [ ] Báo cáo test
- [ ] Hướng dẫn sử dụng
- [ ] Kịch bản demo đã tập trước
