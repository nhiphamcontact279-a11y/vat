# Thống kê VAT theo hóa đơn – GitHub Pages + Google Sheet

Gồm 2 phần:

| File | Việc làm |
|---|---|
| `index.html` | Trang web (đăng lên GitHub Pages): tải file Excel lên, lọc trùng, xem bảng theo quý / tháng / hóa đơn, tải Excel |
| `google-sheet-script.gs` | Apps Script gắn với Google Sheet: nhận dữ liệu từ trang web, lọc trùng, ghi vào Sheet, tự ghi ngày giờ cập nhật |

Luồng hoạt động: bạn tải bảng kê lên trang web → trang web lọc trùng → gửi hóa đơn mới sang Google Sheet →
Sheet tự ghi ngày giờ **Cập nhật lần cuối** và sheet **Tổng hợp** tự tính lại bằng công thức.
Mở trang web ở máy khác vẫn thấy cùng dữ liệu vì dữ liệu nằm trong Google Sheet.

---

## Bước 1. Tạo Google Sheet và dán script

1. Vào Google Drive → **Mới → Google Trang tính** → đặt tên, ví dụ `VAT Local Gaming`.
2. Menu **Tiện ích mở rộng → Apps Script**.
3. Xóa hết code mẫu, mở file `google-sheet-script.gs`, **copy toàn bộ** và dán vào. Bấm **Lưu** (biểu tượng đĩa).
4. Ở thanh trên chọn hàm **`setup`** rồi bấm **Chạy**. Lần đầu Google hỏi cấp quyền: **Xem lại quyền → chọn tài khoản → Nâng cao → Đi tới … (không an toàn) → Cho phép** (đây là script của chính bạn).
5. Chạy xong sẽ hiện hộp thoại **TOKEN**. **Copy và cất lại** (nếu quên: quay lại Sheet, menu **VAT → Hiện TOKEN**).
   Sheet có thêm các tab: `Bán ra`, `Mua vào`, `Cấu hình`, `Tổng hợp`.

## Bước 2. Đăng thành web app

1. Trong Apps Script bấm **Triển khai (Deploy) → Tùy chọn triển khai mới**.
2. Biểu tượng bánh răng → chọn **Ứng dụng web (Web app)**.
3. Cài đặt:
   - **Thực thi dưới tên**: Tôi (Me)
   - **Người có quyền truy cập**: **Bất kỳ ai (Anyone)**
4. Bấm **Triển khai**, **copy URL ứng dụng web** (dạng `https://script.google.com/macros/s/…/exec`).

> Dù để "Anyone", mọi yêu cầu đều bị từ chối nếu không có đúng TOKEN, nên người khác biết URL cũng không đọc hay ghi được dữ liệu.

## Bước 3. Đưa trang web lên GitHub

1. Vào github.com → **New repository** → đặt tên (ví dụ `thong-ke-vat`) → **Create repository**.
2. Bấm **uploading an existing file**, kéo thả `index.html` (có thể kèm `README.md`) → **Commit changes**.
3. Vào **Settings → Pages**:
   - **Source**: Deploy from a branch
   - **Branch**: `main` / `(root)` → **Save**
4. Chờ 1–2 phút, trang web có địa chỉ `https://<tên-github-của-bạn>.github.io/thong-ke-vat/`.

> Không đưa URL web app hay TOKEN vào file trên GitHub. Hai thứ này chỉ nhập trên trang web ở bước sau.

## Bước 4. Kết nối và dùng

1. Mở địa chỉ GitHub Pages. Ở mục **0. Kết nối Google Sheet**, dán **URL web app** và **TOKEN** → bấm **Kết nối**.
   Thông tin được nhớ trong trình duyệt đó (máy khác phải nhập lại một lần).
2. Mục **1. Cập nhật dữ liệu**: chọn file Excel (bảng kê bán ra, bảng kê mua vào, hoặc file thống kê có sheet "Bán ra theo đơn", "Mua vào theo đơn"; có thể chọn nhiều file một lúc).
3. Trang báo từng file: bao nhiêu hóa đơn mới, trùng bỏ qua, cập nhật. Dòng trạng thái bên dưới ô kết nối cho biết đã ghi lên Google Sheet lúc nào.
4. **Lần đầu**: tải lên file thống kê hiện có để nạp toàn bộ dữ liệu. Những lần sau chỉ cần tải bảng kê mới, hóa đơn cũ tự bị lọc trùng.

## Cách lọc trùng

| Loại | Khóa trùng |
|---|---|
| Bán ra | Ký hiệu HĐ + số hóa đơn |
| Mua vào | Số hóa đơn + mã số thuế người bán |

- Hóa đơn đã có và số tiền giống nhau → **bỏ qua**.
- Hóa đơn đã có nhưng số tiền khác → **cập nhật** (bỏ tick "Cập nhật hóa đơn đã có…" nếu không muốn). Nếu file mới ghi VAT = 0 mà dữ liệu cũ đã có số thì **giữ số cũ**.
- Ngày bạn đã sửa tay trong danh sách **không bị ghi đè** khi tải lại bảng kê.
- Nhiều dòng hàng của cùng một hóa đơn được gộp thành một hóa đơn; dòng lặp hệt nhau trong một file bị bỏ.
- Mục **5. Kiểm tra trùng** liệt kê hóa đơn nghi trùng (cùng ngày, đối tác, số tiền nhưng khác số HĐ) để bạn xem và xóa.

## Sheet "Tổng hợp" trong Google Sheet

- Bảng theo tháng và bảng theo quý (đầu kỳ – mua – bán – còn khấu trừ – nộp thuế) đều là **công thức**, tự tính lại khi có hóa đơn mới.
- Tab `Cấu hình`: ô **B2** là năm báo cáo (đổi khi sang năm mới); **B1** là ngày giờ cập nhật lần cuối (script tự ghi); các ô **Đã nộp thuế theo quý** (A5:B8) nhập tay hoặc sửa trên trang web.
- Tab `Bán ra` / `Mua vào` có cột **Cập nhật lúc** ghi ngày giờ mỗi lần hóa đơn được thêm hoặc sửa.

## Khi sửa script

Sau khi sửa `google-sheet-script.gs`, phải **Triển khai → Quản lý bản triển khai → biểu tượng bút chì → Phiên bản mới → Triển khai** thì URL cũ mới dùng code mới.

## Xử lý lỗi thường gặp

| Thông báo | Cách xử lý |
|---|---|
| "Sai TOKEN hoặc chưa chạy setup()" | Chạy lại `setup`, lấy TOKEN bằng menu **VAT → Hiện TOKEN** và nhập lại |
| "Không tải được từ Google Sheet" | Kiểm tra URL kết thúc bằng `/exec` và quyền truy cập là **Anyone** |
| "Chưa có sheet … – hãy chạy setup()" | Chạy hàm `setup` một lần |
| Số hóa đơn mất số 0 đầu trong Sheet | Dùng `setup` để định dạng lại cột (cột Số HĐ đặt dạng văn bản) |
| Tải file lên nhưng báo "không nhận ra" | File cần có cột Ký hiệu HĐ / Số hóa đơn / Ngày hóa đơn / Thuế GTGT (bán ra) hoặc Tên người bán / Giá trị HHDV mua vào chưa có thuế (mua vào) |

## Lưu ý

- Chưa nhập URL/TOKEN thì trang vẫn chạy, nhưng dữ liệu chỉ lưu trong trình duyệt đang dùng.
- Trang web dùng thư viện SheetJS tải từ `cdnjs.cloudflare.com` nên máy cần có internet.
- Số liệu chưa kiểm tra điều kiện khấu trừ đầu vào.
