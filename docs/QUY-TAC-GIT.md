# Quy tắc Git

## Sơ đồ luồng merge

```
  feature/auth-01-03-be-...  ──┐
  feature/prod-01-fe-...     ──┤
  fix/pos-03-be-...          ──┤   PR + ít nhất 1 người duyệt
  chore/be-setup             ──┴──────────────────────────────►  develop
                                                                    │
                                                                    │  1. Kiểm tra ổn định (mục 2)
                                                                    │  2. Chỉ LEAD NHÓM tạo PR develop → main
                                                                    ▼
                                                                  main  ──►  tag v0.x ... v1.0

  hotfix/...  (tạo từ main)  ──►  main   rồi merge ngược lại  ──►  develop
```

- **Mọi nhánh** (`feature`, `fix`, `chore`) chỉ được merge vào **`develop`**.
- **`develop` → `main`:** chỉ **lead nhóm** được tạo PR và merge, và chỉ khi `develop` đã qua kiểm tra ổn định.
- **Không ai** được merge nhánh `feature`/`fix`/`chore` thẳng vào `main`.

## 1. Các nhánh chính

| Nhánh | Mục đích | Ai được merge |
|---|---|---|
| `main` | Code ổn định, dùng để demo và bàn giao | Chỉ lead nhóm (từ `develop` hoặc `hotfix`) |
| `develop` | Nhánh tích hợp, chứa code đã xong | Lead (qua PR) |
| `feature/...` | Làm use case mới | Tạo từ `develop` |
| `fix/...` | Sửa bug trên `develop` | Tạo từ `develop` |
| `hotfix/...` | Sửa lỗi gấp trên `main` | Tạo từ `main`, merge vào cả `main` và `develop` |
| `chore/...` | Cấu hình, setup dự án, seed data | Tạo từ `develop` |

## 2. Quy tắc merge `develop` → `main`

**Ai được làm:** chỉ **lead nhóm** được tạo PR `develop` → `main` và merge. Thành viên khác không được tạo PR vào `main`.

**Khi nào:** cuối mỗi sprint, hoặc trước buổi demo hay bàn giao.

**Checklist ổn định, phải đạt hết mới được merge:**

- [ ] Mọi PR của sprint đã merge vào `develop` và đạt Definition of Done
- [ ] `develop` build và chạy được, không lỗi
- [ ] Migration chạy được trên DB trống, seed data chạy được
- [ ] Đã chạy thử các luồng chính (đăng nhập → nhập kho → bán hàng → thanh toán) và không lỗi
- [ ] Không còn bug **Critical/High** trên `develop`
- [ ] Demo cuối sprint đã đạt

**Sau khi merge thì gắn tag phiên bản:**

```bash
git checkout main
git pull origin main
git tag v0.1          # tăng số phiên bản mỗi sprint, bàn giao là v1.0
git push origin v0.1
```

`develop` **chưa ổn định** thì **không merge**. `main` giữ nguyên bản cũ, sửa xong mới merge.

## 3. Đặt tên nhánh theo use case

**Cú pháp:** `<loại>/<mã-uc>-<fe|be>-<mo-ta>`

```
feature/auth-01-be-dang-nhap
feature/auth-01-fe-dang-nhap
fix/pos-03-be-sai-tong-tien
chore/be-setup
```

- Mã UC lấy theo danh sách trong [DANH_GIA_USECASE.md](DANH_GIA_USECASE.md) (mục 3).
- **Viết thường toàn bộ**, không dấu, nối bằng `-`. Windows không phân biệt hoa thường, nên `AUTH-01` và `auth-01` có thể bị coi là cùng một nhánh và gây lỗi.
- **FE và BE luôn tách nhánh riêng**, kể cả khi làm cùng một use case.
- Nếu FE và BE nằm ở 2 repo riêng thì bỏ phần `fe`/`be`.

## 4. Gộp nhiều use case vào một nhánh

Được gộp khi thỏa **đủ cả 4 điều kiện**:

1. Cùng **một nhóm UC** (AUTH, PROD, STK, ...)
2. Cùng **một người** làm
3. Tổng thay đổi **dưới 400 dòng**
4. Làm xong trong **3 đến 5 ngày**

**Đặt tên:**

| Trường hợp | Ví dụ |
|---|---|
| Mã UC liền nhau | `feature/auth-01-03-be-xac-thuc` (gồm AUTH-01, 02, 03) |
| Mã UC không liền nhau | `feature/cus-be-khach-hang`, liệt kê mã UC trong mô tả PR |

**Trong nhánh gộp, mỗi UC ít nhất một commit riêng:**

```
feat(auth-01): đăng nhập
feat(auth-02): đăng xuất
feat(auth-03): đổi mật khẩu
```

**Không gộp:**
- ❌ UC khác nhóm (ví dụ `stk-01` với `pos-02`)
- ❌ FE và BE chung một nhánh
- ❌ UC lớn (POS-02, POS-03, POS-04, STK-01, STK-04): mỗi UC một nhánh riêng. Nếu vẫn quá 400 dòng thì chia nhỏ tiếp: `feature/pos-02-be-fefo`, `feature/pos-02-be-luu-nhap`

> Danh sách nhánh gộp đã được lên sẵn trong [KE-HOACH-THUC-HIEN.md](KE-HOACH-THUC-HIEN.md).

## 5. Quy trình làm một task

```bash
# 1. Cập nhật develop mới nhất
git checkout develop
git pull origin develop

# 2. Tạo nhánh mới
git checkout -b feature/auth-01-03-be-xac-thuc

# 3. Code, commit thường xuyên
git add .
git commit -m "feat(auth-01): đăng nhập"

# 4. Trước khi tạo PR, cập nhật lại develop để tránh conflict
git pull origin develop

# 5. Đẩy nhánh lên và tạo Pull Request vào develop
git push origin feature/auth-01-03-be-xac-thuc
```

## 6. Quy tắc commit

**Cú pháp:** `<loại>(<mã-uc>): <mô tả ngắn>`. Phần `(<mã-uc>)` bỏ được khi commit không thuộc UC nào.

| Loại | Dùng khi |
|---|---|
| `feat` | Thêm tính năng |
| `fix` | Sửa bug |
| `refactor` | Sửa code, không đổi chức năng |
| `style` | Format, khoảng trắng, không đổi logic |
| `docs` | Tài liệu, README, Swagger |
| `test` | Thêm hoặc sửa test |
| `chore` | Cấu hình, thư viện, build |

**Đúng:**
- `feat(prod-01): thêm API tạo sản phẩm`
- `fix(stk-01): sửa giá vốn bình quân khi nhập thêm vào lô cũ`
- `chore: cấu hình ESLint`

**Sai:**
- `update`, `fix bug`, `abc`, `final`, `final2`

> Mỗi commit chỉ làm **một việc**. Không gộp nhiều thay đổi không liên quan.

## 7. Quy tắc Pull Request

- PR từ nhánh `feature`/`fix`/`chore` luôn merge vào **`develop`**, không bao giờ vào `main`.
- Mỗi PR dưới **400 dòng** thay đổi. Task lớn thì chia nhỏ.
- Cần **ít nhất 1 người duyệt** (lead tầng tương ứng) mới được merge.
- Người tạo PR **không tự duyệt** PR của mình.
- Merge bằng **Squash and merge** để lịch sử gọn.
- Merge xong thì **xóa nhánh**.
- **Tiêu đề PR:** `[AUTH-01..03][BE] Xác thực người dùng`

**Mô tả PR theo mẫu:**

```markdown
## Use case
AUTH-01, AUTH-02, AUTH-03 - Xác thực người dùng

## Đã làm
- POST /api/v1/auth/login, khóa tạm 15 phút sau 5 lần sai
- POST /api/v1/auth/logout, thu hồi refresh token
- PUT /api/v1/auth/password

## Cách test
1. Đăng nhập sai 5 lần, lần thứ 6 phải bị khóa
2. ...

## Ảnh chụp (nếu là UI)
```

## 8. Review code

**Người review kiểm tra:**
- Đúng tiêu chí chấp nhận của use case
- Đúng API contract
- Đúng quy tắc nghiệp vụ chung (mục 6 trong DANH_GIA_USECASE.md), nhất là phần tồn kho và transaction
- Không có code thừa, `console.log`, code bị comment bỏ
- Đặt tên rõ ràng, đúng convention
- Không lộ thông tin nhạy cảm (mật khẩu, key)

**Thời hạn:** review trong vòng **24 giờ** kể từ khi PR được tạo.

## 9. Xử lý conflict

- Người tạo PR **tự giải quyết** conflict trên nhánh của mình.
- Conflict ở code của người khác thì **hỏi người đó**, không tự ý xóa code của họ.
- Giải quyết xong phải **chạy lại project** để chắc chắn không lỗi rồi mới push.
- File dễ conflict (migration DB, file route chung, file seed): **báo trong nhóm trước khi sửa**.

## 10. Điều cấm

- ❌ Push thẳng lên `main` hoặc `develop`
- ❌ Merge nhánh `feature`/`fix`/`chore` thẳng vào `main`
- ❌ Thành viên không phải lead nhóm tạo PR vào `main`
- ❌ `git push --force` lên nhánh chung
- ❌ Commit file `.env`, mật khẩu, API key
- ❌ Commit `node_modules/`, `target/`, `build/`, `dist/`, file của IDE
- ❌ Merge PR khi chưa được duyệt hoặc code đang lỗi
- ❌ Để một nhánh tồn tại quá 5 ngày không merge
- ❌ Sửa migration đã merge. Cần đổi DB thì tạo migration mới

## 11. Lưu ý khác

- Luôn có file `.gitignore` và `.env.example` (chỉ chứa tên biến, không chứa giá trị thật).
- Pull `develop` **mỗi ngày** trước khi bắt đầu code.
- Nên bật **Branch protection** trên GitHub:
  - `develop`: bắt buộc có PR và ít nhất 1 người duyệt.
  - `main`: bắt buộc có PR, và chỉ lead nhóm được merge (dùng *Restrict who can push*).
