# TÀI LIỆU NGHIỆP VỤ – HỆ THỐNG QUẢN LÝ ĐẶT VÉ XEM PHIM PXU CINEMA

> Phiên bản: 2.0 · Phạm vi: toàn bộ nghiệp vụ của một chuỗi rạp chiếu phim (đặt vé online, bán tại quầy, soát vé, vận hành, báo cáo).
> Đây là **nguồn sự thật duy nhất (single source of truth)** về nghiệp vụ. Ba tài liệu còn lại (Giao diện, Kỹ thuật Frontend, Kỹ thuật Admin) phải bám theo tên trạng thái, mã lỗi, quy tắc trong file này.
> Quy ước: **MUST** = bắt buộc, **SHOULD** = nên có, **MAY** = tùy chọn/giai đoạn sau. Tiền tệ: VND (số nguyên, không có phần thập phân). Múi giờ nghiệp vụ: `Asia/Ho_Chi_Minh` (UTC+7). Lưu DB theo UTC cho trường thời điểm (`DATETIME` UTC), riêng `show_date`/`start_time` lưu theo giờ địa phương của rạp.

---

## 1. TỔNG QUAN

### 1.1 Mục tiêu
1. Khách đặt vé nhanh, chọn ghế trực quan, thanh toán an toàn, nhận vé điện tử có mã QR.
2. Rạp quản lý phim, suất chiếu, giá, phòng/ghế, combo, khuyến mãi, doanh thu.
3. Nhân viên bán vé tại quầy (POS) và soát vé chính xác, chống dùng lại/gian lận.
4. Mọi thao tác nhạy cảm đều có phân quyền và nhật ký (audit).

### 1.2 Thuật ngữ
| Thuật ngữ | Ý nghĩa |
| --- | --- |
| Cinema (Rạp/Chi nhánh) | Một địa điểm vật lý có nhiều phòng chiếu |
| Room (Phòng chiếu) | Phòng trong một rạp, có sơ đồ ghế |
| Showtime (Suất chiếu) | Một lần chiếu của 1 phim trong 1 phòng tại 1 thời điểm |
| Seat hold (Giữ ghế) | Khóa tạm ghế cho một đơn trong thời gian giới hạn |
| Booking (Đơn đặt vé) | Giao dịch gồm ghế (+ combo) của 1 suất chiếu |
| Ticket (Vé) | 1 vé = 1 ghế, có mã QR duy nhất |
| Voucher | Mã giảm giá |
| POS | Bán hàng tại quầy |
| Check-in | Soát vé vào phòng chiếu |

### 1.3 Phạm vi chức năng theo giai đoạn
| Giai đoạn | Nội dung |
| --- | --- |
| **P1 – Lõi (MUST)** | Tài khoản, phim, suất chiếu, ghế, giữ ghế, đặt vé, thanh toán QR + duyệt tay, vé điện tử, check-in, POS, voucher, dashboard cơ bản, phân quyền, audit |
| **P2 – Vận hành (SHOULD)** | Hoàn/hủy vé, đổi suất, thông báo email, giá động, điểm thành viên, báo cáo xuất file, ca làm việc POS, kho bắp nước, banner/tin tức |
| **P3 – Mở rộng (MAY)** | Đối soát ngân hàng tự động (webhook), cổng thanh toán (VNPay/MoMo/ZaloPay), đa chi nhánh đầy đủ, đánh giá phim, đa ngôn ngữ, ví vé (Wallet), hóa đơn điện tử |

---

## 2. VAI TRÒ VÀ QUYỀN HẠN

### 2.1 Danh sách vai trò
| Vai trò | Mô tả | Phạm vi dữ liệu |
| --- | --- | --- |
| `guest` | Chưa đăng nhập | Xem phim, lịch chiếu, combo, tin tức |
| `customer` | Khách hàng đăng ký | Đặt vé, xem/hủy vé của mình, hồ sơ, điểm |
| `staff` | Nhân viên quầy/soát vé | POS, check-in, xem vé trong ngày của rạp được gán |
| `manager` | Quản lý rạp (SHOULD) | Toàn quyền vận hành trong rạp được gán: suất chiếu, đơn, báo cáo rạp, duyệt hoàn tiền |
| `admin` | Quản trị hệ thống | Toàn quyền, mọi rạp, cấu hình thanh toán, phân quyền |

### 2.2 Ma trận quyền (permission)
Ký hiệu: ✔ có, ✖ không, 🔸 chỉ dữ liệu của mình/rạp mình.

| Chức năng | guest | customer | staff | manager | admin |
| --- | :-: | :-: | :-: | :-: | :-: |
| Xem phim/lịch chiếu/combo | ✔ | ✔ | ✔ | ✔ | ✔ |
| Đăng ký/đăng nhập/quên mật khẩu | ✔ | ✔ | ✔ | ✔ | ✔ |
| Đặt vé online | ✖ | ✔ | ✖ | ✖ | ✖ |
| Xem/hủy đơn của mình | ✖ | 🔸 | ✖ | ✖ | ✖ |
| Bán vé POS | ✖ | ✖ | ✔ | ✔ | ✔ |
| Check-in vé | ✖ | ✖ | ✔ | ✔ | ✔ |
| Xem danh sách đơn | ✖ | 🔸 | 🔸 (ca/ngày) | 🔸 | ✔ |
| Duyệt/từ chối biên lai | ✖ | ✖ | ✖ | ✔ | ✔ |
| Hủy đơn/hoàn tiền | ✖ | 🔸 (theo chính sách) | ✖ | ✔ | ✔ |
| CRUD phim | ✖ | ✖ | ✖ | ✖ | ✔ |
| CRUD suất chiếu | ✖ | ✖ | ✖ | 🔸 | ✔ |
| CRUD phòng/ghế | ✖ | ✖ | ✖ | 🔸 | ✔ |
| CRUD combo, giá, voucher | ✖ | ✖ | ✖ | 🔸 | ✔ |
| Cấu hình thanh toán QR | ✖ | ✖ | ✖ | ✖ | ✔ |
| Báo cáo/dashboard | ✖ | ✖ | 🔸 (ca của mình) | 🔸 | ✔ |
| Quản lý người dùng/phân quyền | ✖ | ✖ | ✖ | ✖ | ✔ |
| Xem audit log | ✖ | ✖ | ✖ | 🔸 | ✔ |

> Nếu P1 chưa có `manager`, gộp quyền manager vào `admin`. Cấu trúc permission phải thiết kế theo **mã quyền** (vd `booking.refund`, `showtime.write`) để thêm vai trò sau này không phải sửa code.

---

## 3. CÁC QUY TRÌNH NGHIỆP VỤ CHÍNH

### 3.1 Đăng ký / đăng nhập / tài khoản
1. **Đăng ký**: họ tên, email (duy nhất), số điện thoại (SHOULD duy nhất), mật khẩu ≥ 8 ký tự gồm chữ + số. Tùy chọn: ngày sinh (cho voucher sinh nhật và kiểm tra độ tuổi).
2. **Xác thực email (SHOULD)**: gửi liên kết/OTP, tài khoản chưa xác thực vẫn đăng nhập được nhưng không được thanh toán (hoặc theo cấu hình).
3. **Đăng nhập**: email + mật khẩu → access token (15 phút) + refresh token (7–30 ngày, xoay vòng). Sai quá **5 lần/15 phút** → khóa tạm 15 phút theo (email + IP).
4. **Quên mật khẩu**: nhập email → gửi liên kết dùng 1 lần, hiệu lực 30 phút → đặt mật khẩu mới → mọi phiên cũ bị thu hồi. Phản hồi luôn trung tính ("Nếu email tồn tại, chúng tôi đã gửi hướng dẫn") để không lộ email.
5. **Đổi mật khẩu**: yêu cầu mật khẩu cũ; thu hồi các phiên khác.
6. **Hồ sơ**: sửa tên, SĐT, ngày sinh, avatar. Đổi email cần xác thực lại.
7. **Khóa tài khoản** (admin): `status = locked` → không đăng nhập được, token hiện có bị vô hiệu ở lần làm mới kế tiếp. Đơn đang giữ ghế của tài khoản đó vẫn tự hết hạn theo quy tắc.
8. **Xóa tài khoản (MAY)**: ẩn danh hóa dữ liệu cá nhân, giữ đơn hàng để đối soát (không xóa cứng).
9. Tài khoản mẫu (seed) **MUST** bị buộc đổi mật khẩu ở lần đăng nhập đầu hoặc bị vô hiệu ở môi trường production.

### 3.2 Quản lý danh mục phim (Admin)
- Trường: tiêu đề, slug (duy nhất), mô tả, đạo diễn, diễn viên, thể loại (nhiều), thời lượng (phút), ngày khởi chiếu, ngôn ngữ, **nhãn tuổi** (`P`, `K`, `T13`, `T16`, `T18`), poster, banner, trailer (YouTube), điểm đánh giá, trạng thái.
- Trạng thái phim: `coming_soon` → `now_showing` → `ended`. Chuyển tự động theo `release_date` (job hằng ngày) và có thể chỉnh tay.
- **Không xóa cứng** phim đã có suất chiếu/đơn: dùng `deleted_at` (soft delete) hoặc `ended`. Chỉ cho xóa cứng nếu chưa có suất chiếu nào.
- Phim `ended` hoặc ẩn không hiển thị ở trang đặt vé nhưng vẫn xuất hiện trong lịch sử đơn.
- Hiển thị nhãn tuổi theo quy định: P (mọi lứa tuổi), K (dưới 13 có người bảo hộ), T13, T16, T18 (cấm dưới độ tuổi tương ứng).

### 3.3 Rạp, phòng chiếu, ghế
**Cấu trúc**: `cinemas (1) – (N) rooms (1) – (N) seats`.
- Phòng: tên, loại (`2D`, `3D`, `IMAX`, `4DX`, `Gold Class`), sức chứa, trạng thái (`active`, `maintenance`).
- Sơ đồ ghế **tùy biến theo ô lưới** (không chỉ ma trận cố định): mỗi ô có `row_label`, `col_number`, `grid_x`, `grid_y`, loại ô: `standard`, `vip`, `couple`, `aisle` (lối đi), `blank` (khuyết), `disabled` (hỏng).
- Ghế đôi (`couple`) chiếm 2 ô liền kề, **bán nguyên cặp** (1 ghế logic = 2 chỗ).
- Phụ thu theo loại ghế cấu hình theo **bảng giá** (xem 3.5), không hard-code.
- Ghế `disabled`/bảo trì: không bán. Có thể đặt bảo trì **theo phòng** hoặc **theo suất chiếu** (chặn 1 ghế riêng cho 1 suất).
- Khi sửa sơ đồ phòng đã có suất chiếu tương lai có đơn: MUST cảnh báo, và không cho xóa ghế đã được bán; chỉ cho đổi sang `disabled` nếu không ảnh hưởng đơn đã thanh toán (nếu có thì phải đi qua quy trình đổi ghế/hoàn tiền).

### 3.4 Suất chiếu
- Trường: phim, phòng, `show_date`, `start_time`, `end_time`, định dạng (`2D Phụ đề`, `2D Lồng tiếng`, `3D`…), trạng thái.
- `end_time = start_time + duration phim + thời gian quảng cáo (cấu hình, mặc định 15 phút)`.
- **Chống trùng**: trong cùng một phòng, hai suất chiếu không được chồng lấn, giữa 2 suất phải có **thời gian dọn phòng** (mặc định 15 phút, cấu hình được). Kiểm tra ở backend trong transaction; trả lỗi `SHOWTIME_OVERLAP` kèm suất xung đột.
- Không tạo suất chiếu trong quá khứ; không tạo suất cho phim `ended`, phòng `maintenance`.
- Có thể **tạo hàng loạt** (chọn phim, phòng, khoảng ngày, danh sách giờ) với xem trước xung đột trước khi lưu.
- Trạng thái: `scheduled` → `selling` (mở bán) → `closed` (đóng bán) → `finished`; ngoài ra `cancelled`.
  - Mặc định mở bán khi tạo (hoặc theo `sale_open_at`), **đóng bán online** trước giờ chiếu `X` phút (mặc định 15, cấu hình), **POS** vẫn bán đến trước giờ chiếu `Y` phút (mặc định 0 hoặc 10, cấu hình).
- **Sửa suất chiếu** đã có đơn: chỉ cho sửa giá/ghi chú; đổi giờ/phòng/phim bắt buộc đi qua quy trình **dời/hủy suất** (3.12).
- Không xóa suất có đơn; chỉ `cancelled`.

### 3.5 Giá vé
Giá một vé = `giá nền suất chiếu` + `phụ thu loại ghế` + `phụ thu định dạng/phòng` (nếu có), sau đó áp **quy tắc giá động**.
- **Bảng giá (`price_rules`)** (SHOULD): theo loại ngày (`weekday`, `weekend`, `holiday`), khung giờ (sáng sớm, ngày thường, giờ vàng, khuya), đối tượng (`student`, `senior`, `member`), loại phòng. Thứ tự ưu tiên: ngày lễ > cuối tuần > ngày thường; quy tắc cụ thể hơn thắng quy tắc chung.
- Giá **chốt tại thời điểm tạo đơn** và lưu vào `booking_seats.price` (snapshot) – đổi bảng giá sau đó không ảnh hưởng đơn đã tạo.
- Làm tròn đến 1.000 VND (cấu hình được).
- Giá hiển thị cho khách luôn là giá đã gồm thuế (VAT 8%/10% theo quy định hiện hành, cấu hình được); hóa đơn tách dòng thuế nếu cần.
- Đối tượng ưu đãi (HSSV, người cao tuổi) cần **xuất trình giấy tờ khi nhận vé**; ghi chú lên vé và cho staff cảnh báo khi check-in.

### 3.6 Combo bắp nước
- Trường: tên, mô tả, giá, hình ảnh, trạng thái, danh mục (`combo`, `popcorn`, `drink`, `snack`), tồn kho (SHOULD), giới hạn mua mỗi đơn (mặc định 10).
- Combo có thể gồm nhiều thành phần (`combo_items`) để trừ kho từng món (P2).
- Hết hàng (`stock = 0`) → tự động không cho đặt; hiển thị "Hết hàng".
- Combo mua cùng đơn: khách nhận **tại quầy** bằng mã đơn/QR combo (xem 3.10). Combo **chỉ có mã nhận hàng**, không có check-in như vé.
- Có thể đặt combo ở bước riêng hoặc **chỉ mua combo tại POS**.

### 3.7 QUY TRÌNH ĐẶT VÉ ONLINE (trọng tâm)

```
Chọn phim → Chọn suất → Chọn ghế → [Giữ ghế] → Chọn combo → Voucher
→ Xem lại đơn → Thanh toán → Chờ xác nhận → Vé điện tử → Check-in
```

**Bước 1 – Chọn phim & suất**: lọc theo ngày, rạp, định dạng. Chỉ hiển thị suất `selling`, chưa qua giờ đóng bán. Hiển thị số ghế còn lại (cache ≤ 30 giây).

**Bước 2 – Chọn ghế**: xem sơ đồ với trạng thái từng ghế: `available`, `held` (người khác giữ), `sold`, `disabled`, `selected` (của mình). Quy tắc:
- Tối đa **8 ghế/đơn** (cấu hình).
- **Quy tắc ghế trống lẻ (SHOULD)**: không để lại đúng 1 ghế trống kẹp giữa hàng (cảnh báo, không chặn cứng).
- Ghế đôi chọn theo cặp.
- Phim có nhãn T16/T18: hiển thị cảnh báo độ tuổi; bắt buộc tích "Tôi xác nhận đủ độ tuổi và sẽ xuất trình giấy tờ khi cần".

**Bước 3 – Giữ ghế (seat hold)**: khi bấm "Tiếp tục", backend tạo đơn `holding` và khóa ghế **atomic**:
- Giữ **10 phút** kể từ lúc tạo đơn (`hold_expires_at`). Đồng hồ đếm ngược dựa trên **giờ máy chủ** (frontend nhận `server_time` để bù lệch).
- Một người dùng chỉ có tối đa **1 đơn `holding` cho mỗi suất** và tối đa **3 đơn `holding` đồng thời** (chống găm ghế). Tạo đơn mới cho cùng suất sẽ hủy/gộp đơn cũ.
- Ghế đã `held` bởi người khác → lỗi `SEAT_UNAVAILABLE` kèm danh sách ghế bị mất; UI yêu cầu chọn lại.
- Có thể **gia hạn giữ ghế 1 lần (+5 phút)** khi đang ở trang thanh toán (SHOULD).
- Hết hạn: job mỗi 30–60 giây chuyển đơn `holding` quá hạn sang `expired` và nhả ghế. Khi khách quay lại: hiển thị "Đơn đã hết hạn" và nút "Chọn lại ghế".

**Bước 4 – Combo & voucher**: thêm/bớt combo (tính lại tổng ở backend). Voucher kiểm tra ở backend (xem 3.9). Tổng tiền hiển thị luôn là kết quả **backend tính** (frontend chỉ tạm tính).

**Bước 5 – Xem lại & xác nhận**: tóm tắt phim/suất/ghế/combo/giảm giá/tổng, điều khoản, chọn phương thức thanh toán, nhập/ xác nhận thông tin liên hệ (tên, email, SĐT).

**Bước 6 – Thanh toán** (xem 3.8). **Bước 7 – Phát hành vé** sau khi thanh toán được xác nhận. **Bước 8 – Nhận vé** qua màn hình, email, và mục "Vé của tôi".

### 3.8 THANH TOÁN QR NGÂN HÀNG

**Mã đơn & nội dung chuyển khoản**
- `booking_code`: `PXU-YYMMDD-XXXXXX` (6 ký tự ngẫu nhiên không nhập nhằng, vd bỏ `0/O/1/I`), duy nhất.
- Nội dung chuyển khoản: `{transfer_prefix} {mã rút gọn}` (vd `PXU A7K9QX`), phải **duy nhất theo đơn** để đối soát.

**Hiển thị**
- Ưu tiên 1: ảnh QR tùy chỉnh do admin tải lên (`use_custom_qr = true`). Ưu tiên 2: VietQR sinh động (đã nhúng số tiền, nội dung).
- Hiển thị: ngân hàng, số tài khoản, chủ tài khoản, **số tiền chính xác**, nội dung chuyển khoản, nút sao chép từng trường, đồng hồ đếm ngược.
- Cảnh báo rõ: "Chuyển đúng số tiền và nội dung để được xác nhận nhanh".

**Hai chế độ xác nhận** (cấu hình `payment_confirm_mode`):
1. **`manual` (P1)**: khách tải biên lai (ảnh ≤ 5 MB, jpg/png/webp, bắt buộc) → bấm "Tôi đã chuyển khoản" → `payment.status = proof_submitted`, đơn **tiếp tục giữ ghế** → admin/manager duyệt.
2. **`auto` (P3)**: webhook ngân hàng/cổng thanh toán (SePay, Casso, payOS, VNPay…) gửi giao dịch → hệ thống **khớp theo nội dung + số tiền** → tự `paid` và phát hành vé trong vài giây. Webhook phải: xác thực chữ ký/secret, **idempotent** (lưu `provider_txn_id` duy nhất), chống replay.

**Quy tắc duyệt thủ công**
- Hàng đợi duyệt sắp xếp theo thời gian gửi cũ nhất; hiển thị: ảnh biên lai (phóng to), số tiền, nội dung, thời gian gửi, thời gian còn lại.
- **Duyệt** → `paid`, phát hành vé, gửi thông báo.
- **Từ chối** (bắt buộc chọn lý do: `unclear_proof`, `wrong_amount`, `wrong_content`, `not_received`, `other` + ghi chú) → `payment.status = rejected`; khách nhận thông báo và **được gửi lại biên lai** trong thời hạn gia hạn (xem bên dưới).
- **Đối chiếu số tiền**: nếu khách có thể nhập "số tiền đã chuyển" (SHOULD), hệ thống cảnh báo khi ≠ `final_amount`. Chuyển **thiếu** → từ chối/yêu cầu bổ sung; chuyển **thừa** → duyệt và ghi nhận phần dư để hoàn (3.11).
- **Chống biên lai trùng**: tính hash ảnh (SHA-256); cảnh báo nếu ảnh đã dùng cho đơn khác.
- **SLA giữ ghế khi chờ duyệt**: đơn `proof_submitted` giữ ghế tối đa **60 phút** (cấu hình) hoặc đến **trước giờ chiếu 30 phút** (lấy mốc sớm hơn). Quá hạn mà chưa có xử lý → tự `expired`, nhả ghế, gửi thông báo cho khách và cảnh báo admin. Nếu khách đã thực sự chuyển tiền sau đó → quy trình hoàn tiền/khôi phục thủ công (3.11).
- Admin được nhắc (thông báo trong app + email tùy chọn) khi có biên lai chờ quá 10 phút.

**Thanh toán sau khi hết hạn giữ ghế**: nếu khách chuyển khoản sau khi `expired`, giao dịch rơi vào **"Giao dịch bất thường"** (3.11) để admin quyết định: khôi phục đơn (nếu ghế còn trống), đổi suất, hoặc hoàn tiền.

**Cổng thanh toán khác (P3)**: tạo `payment` theo `provider` (`bank_qr`, `vnpay`, `momo`, `zalopay`, `cash`); luồng IPN/return URL; cùng một máy trạng thái thanh toán.

### 3.9 Voucher / khuyến mãi
Trường: mã (không phân biệt hoa thường), loại (`percent`/`fixed`), giá trị, đơn tối thiểu, giảm tối đa, ngày bắt đầu/hết hạn, **tổng lượt dùng**, **lượt/người dùng**, áp dụng theo (phim, rạp, định dạng, khung giờ, ngày trong tuần), đối tượng (`all`, `new_member`, `member_tier`, `birthday`), cộng dồn hay không, trạng thái.
Quy tắc:
- Kiểm tra ở backend khi áp mã **và** lúc thanh toán (kiểm tra lại, tránh hết lượt/hết hạn giữa chừng).
- Mỗi đơn **1 voucher** (mặc định). Voucher giảm **chỉ áp lên vé** hay **vé + combo** theo cấu hình (`apply_to`).
- Lượt dùng chỉ **tính khi đơn `paid`**; đơn hết hạn/hủy thì **trả lại lượt**. Dùng bảng `voucher_usages` để chặn vượt giới hạn (có khóa/transaction).
- Tổng thanh toán tối thiểu **0 đồng**; nếu đơn 0 đồng thì bỏ qua bước QR (xác nhận thẳng).
- Mã lỗi: `VOUCHER_INVALID`, `VOUCHER_EXPIRED`, `VOUCHER_EXHAUSTED`, `VOUCHER_MIN_ORDER`, `VOUCHER_NOT_APPLICABLE`, `VOUCHER_ALREADY_USED`.

### 3.10 Vé điện tử & soát vé (Check-in)
**Phát hành vé**: khi đơn `paid`, tạo **1 vé cho mỗi ghế** (`tickets`) với `ticket_code` ngẫu nhiên khó đoán (≥ 128 bit, dạng chuỗi/ JWT/HMAC ký). QR chứa mã này (không chứa dữ liệu cá nhân). Vé không đổi mã khi gửi lại.

**Nội dung vé**: tên phim, nhãn tuổi, ngày giờ, phòng, ghế, định dạng, rạp, mã đơn, mã QR, ghi chú (HSSV/người cao tuổi, combo nhận tại quầy).

**Check-in (staff)**: quét QR (camera) hoặc nhập mã.
Kết quả và mã lỗi:
| Kết quả | Mã | Điều kiện |
| --- | --- | --- |
| ✅ Hợp lệ | `OK` | Vé `valid`, đúng rạp, trong khung giờ cho phép → chuyển `used`, ghi `checked_in_at`, `checked_in_by` |
| ⚠️ Đã dùng | `ALREADY_USED` | Vé `used` → hiện giờ & người check-in trước đó |
| ⏱ Chưa đến giờ | `TOO_EARLY` | Sớm hơn `start_time - 60 phút` (cấu hình) |
| ⏱ Quá giờ | `TOO_LATE` | Sau `end_time` (cho phép vào trễ đến X phút sau giờ chiếu, mặc định 30, manager có thể override) |
| ❌ Không tồn tại | `NOT_FOUND` | Mã sai |
| ❌ Chưa thanh toán/hủy | `NOT_PAID` / `CANCELLED` | Đơn chưa `paid` hoặc vé `cancelled` |
| ❌ Sai rạp/suất | `WRONG_CINEMA` | Vé thuộc rạp khác |
| ⚠️ Cần kiểm tra giấy tờ | `ID_CHECK_REQUIRED` | Vé ưu đãi/nhãn T16/T18 → staff xác nhận thủ công rồi bấm "Đã kiểm tra" |

- Check-in là thao tác **atomic** (`UPDATE ... WHERE status='valid'` kiểm tra số dòng ảnh hưởng) để 2 máy quét không cùng cho vào 1 vé.
- Ghi log mọi lần quét (kể cả thất bại) vào `checkin_logs`.
- Cho phép **check-in theo đơn** (cả nhóm) để vào nhanh; mỗi vé vẫn là 1 bản ghi `used`.
- Vé không dùng sau `end_time` → job chuyển `expired`.
- **Nhận combo**: quét QR đơn tại quầy bắp nước → đánh dấu `combo_picked_up_at` (một lần).
- **Chế độ offline (MAY)**: staff app tải sẵn danh sách vé của suất sắp chiếu, đồng bộ lại khi có mạng, xử lý xung đột theo thời gian.

### 3.11 Hủy vé, hoàn tiền, giao dịch bất thường
**Hủy bởi khách**
- Cho phép tự hủy khi đơn `holding` (miễn phí) hoặc đơn `paid` nếu **còn ≥ 2 giờ trước giờ chiếu** (cấu hình) – tùy chính sách rạp: hoàn 100% / hoàn theo % / **không hoàn** (mặc định rạp VN thường không hoàn sau khi thanh toán; vì vậy chính sách này **cấu hình được**, mặc định: *không cho khách tự hủy vé đã thanh toán*, chỉ liên hệ hỗ trợ).
- Vé đã `used` không hủy được.

**Hủy bởi rạp** (hủy suất, sự cố, nhầm lẫn): tất cả đơn `paid` của suất → tự tạo `refund` **100%** hoặc đề xuất **đổi suất**; thông báo khách ngay.

**Quy trình hoàn tiền** (`refunds`): `requested` → `approved` / `rejected` → `processing` → `completed` (ghi mã giao dịch hoàn, thời gian, người thực hiện) hoặc `failed`.
- Hoàn qua **chuyển khoản ngược** thủ công (admin chuyển rồi nhập mã tham chiếu) ở P1/P2; hoàn qua cổng ở P3.
- Hoàn một phần: hoàn **một số ghế**, hoàn **combo**, hoàn **phần thừa**; cập nhật `payment_status = partially_refunded/refunded`; vé tương ứng `cancelled`; ghế nhả lại (nếu còn trước giờ chiếu).
- Trả lại **lượt voucher** và **trừ điểm thưởng** đã cộng (nếu có).
- Mọi hoàn tiền **bắt buộc** có lý do + người duyệt + audit; hoàn trên ngưỡng (vd > 2.000.000đ) cần **manager/admin**.

**Giao dịch bất thường** (`payment_exceptions`): chuyển khoản không khớp đơn nào; chuyển sau hết hạn; chuyển trùng; chuyển thừa/thiếu. Admin xử lý: `link_to_booking`, `restore_booking`, `refund`, `ignore` (kèm lý do).

### 3.12 Đổi suất / đổi ghế / dời suất
- **Đổi ghế** trong cùng suất (khách hoặc staff): chỉ khi vé chưa `used`, còn ≥ 30 phút trước giờ chiếu; ghế mới cùng hoặc rẻ hơn (chênh lệch hoàn/thu thêm).
- **Đổi suất**: cho phép 1 lần, ≥ 2 giờ trước giờ chiếu (cấu hình), cùng phim, chênh giá thu thêm/hoàn lại; tạo bản ghi `booking_changes`, vé cũ `cancelled`, vé mới phát hành.
- **Rạp dời/hủy suất**: công cụ admin chọn suất → xem danh sách đơn bị ảnh hưởng → chọn hành động hàng loạt (đổi sang suất khác / hoàn tiền) → gửi thông báo.

### 3.13 Bán vé tại quầy (POS)
- Staff chọn phim → suất → ghế (sơ đồ giống online) → combo → thanh toán `cash` / `bank_qr` / `card` (máy POS thẻ, nhập mã giao dịch) / `mixed`.
- Giữ ghế POS: 3 phút; thanh toán xong **xuất vé in ngay** (khổ 80mm) hoặc gửi mã vé qua SĐT/email nếu khách muốn.
- Không bắt buộc thông tin khách (`customer_name = "Khách lẻ"`); nhập SĐT để tích điểm.
- Mỗi giao dịch gắn **ca làm việc** (`shift_id`) và `staff_id`.
- **Ca làm việc (SHOULD)**: mở ca (tiền đầu ca) → bán hàng → đóng ca (đếm tiền, so với hệ thống, ghi chênh lệch, lý do). Báo cáo ca xuất được. Không bán nếu chưa mở ca (cấu hình).
- Hủy/hoàn tại quầy cần mật khẩu/ PIN **manager**.
- In lại vé: ghi log "reprint".

### 3.14 Điểm thành viên & hạng
- Cộng điểm khi đơn `paid`: ví dụ **1 điểm / 10.000đ** thực trả (cấu hình), cộng sau khi vé được phát hành; POS cộng theo SĐT.
- Đổi điểm: 1 điểm = X đồng giảm giá khi thanh toán, hoặc đổi quà/voucher; giới hạn tối đa % đơn.
- Hạng: `member` → `silver` → `gold` → `diamond` theo **tổng chi tiêu 12 tháng** (cấu hình ngưỡng); quyền lợi: tích điểm nhanh hơn, giảm giá bắp nước, voucher sinh nhật.
- Điểm có hạn sử dụng (vd 12 tháng) (MAY). Mọi biến động ghi `point_transactions` (`earn`, `redeem`, `expire`, `adjust`, `refund_revert`).

### 3.15 Thông báo
| Sự kiện | Kênh | Người nhận |
| --- | --- | --- |
| Đăng ký, xác thực email, quên mật khẩu | Email | Khách |
| Đặt giữ ghế sắp hết hạn (còn 2 phút) | Trên màn hình | Khách |
| Đã nhận biên lai, chờ duyệt | Email + in-app | Khách |
| Thanh toán thành công + vé | Email (kèm vé) + in-app | Khách |
| Biên lai bị từ chối | Email + in-app | Khách |
| Nhắc giờ chiếu (trước 2 giờ) | Email/SMS/Zalo (MAY) | Khách |
| Hủy/dời suất, hoàn tiền | Email + in-app | Khách |
| Biên lai mới/chờ lâu, giao dịch bất thường, hết kho, lỗi job | In-app + email | Admin/Manager |
Có **mẫu nội dung (template)** quản lý được, gửi qua **hàng đợi** với retry/backoff; lưu `notifications` (đã đọc/chưa đọc).

### 3.16 Đánh giá & nội dung (P2/P3)
- Đánh giá phim (1–5 sao + bình luận): chỉ khách có vé **đã check-in** của phim đó; mỗi phim 1 đánh giá; admin kiểm duyệt/ẩn.
- Banner trang chủ, tin tức/khuyến mãi, trang tĩnh (điều khoản, chính sách, FAQ, liên hệ) quản lý từ admin; có lịch đăng/hết hạn.
- Phim yêu thích, nhận thông báo khi phim sắp chiếu mở bán.

---

## 4. MÁY TRẠNG THÁI (STATE MACHINE)

### 4.1 Đơn đặt vé `bookings.status`
```
holding ──(thanh toán xác nhận)──▶ confirmed ──(suất kết thúc)──▶ completed
   │                                   │
   ├─(hết hạn)──▶ expired              ├─(hủy/hoàn toàn bộ)──▶ cancelled
   └─(khách hủy)──▶ cancelled          └─(hoàn một phần)──▶ confirmed (partially_refunded)
```
| Trạng thái | Giữ ghế | Ghi chú |
| --- | :-: | --- |
| `holding` | ✔ (≤ 10 phút) | Chưa thanh toán |
| `confirmed` | ✔ (vĩnh viễn) | Đã thanh toán, có vé |
| `completed` | – | Suất đã chiếu |
| `expired` | ✖ | Hết hạn giữ ghế / SLA duyệt |
| `cancelled` | ✖ | Hủy bởi khách/rạp |

### 4.2 Thanh toán `payments.status`
```
pending ──(gửi biên lai)──▶ proof_submitted ──(duyệt)──▶ paid ──▶ partially_refunded / refunded
   │                              │
   │                              └─(từ chối)──▶ rejected ──(gửi lại)──▶ proof_submitted
   └─(hết hạn)──▶ expired        (rejected quá hạn ──▶ expired)
failed (cổng thanh toán lỗi) · cancelled
```
Một đơn có thể có **nhiều bản ghi `payments`** (thử lại, đa phương thức POS). Đơn `confirmed` khi **tổng `paid` ≥ `final_amount`**.

### 4.3 Vé `tickets.status`
`valid` → `used` | `cancelled` | `expired`. (Không có đường quay lại `valid` trừ thao tác **hoàn tác check-in** của manager, có lý do + audit.)

### 4.4 Suất chiếu `showtimes.status`
`scheduled` → `selling` → `closed` → `finished`; `cancelled` từ bất kỳ trạng thái nào trước `finished`.

### 4.5 Hoàn tiền `refunds.status`
`requested` → `approved` → `processing` → `completed`; nhánh `rejected`, `failed`.

### 4.6 Chuyển trạng thái tự động (jobs)
| Job | Tần suất | Việc làm |
| --- | --- | --- |
| `expire-holds` | 30–60 giây | `holding` quá hạn → `expired`, nhả ghế, trả lượt voucher |
| `expire-proof-pending` | 1 phút | Áp SLA 3.8 |
| `close-sales` | 1 phút | Suất đến giờ đóng bán → `closed` |
| `finish-showtimes` | 5 phút | Suất qua `end_time` → `finished`, đơn → `completed`, vé `valid` → `expired` |
| `movie-status-sync` | Hằng ngày 00:05 | Cập nhật `now_showing`/`ended` |
| `remind-showtime` | 10 phút | Gửi nhắc giờ chiếu |
| `cleanup` | Hằng ngày | Xóa token hết hạn, ảnh mồ côi |
Jobs phải **idempotent** và dùng khóa phân tán/advisory lock để không chạy trùng khi có nhiều instance.

---

## 5. QUY TẮC NGHIỆP VỤ XUYÊN SUỐT

1. **Không bao giờ bán trùng ghế**: ràng buộc `UNIQUE(showtime_id, seat_id)` trên bảng ghế đã giữ/bán (xem 6) + transaction + khóa hàng.
2. **Tổng tiền do backend quyết định**; client chỉ hiển thị.
3. **Snapshot dữ liệu giá/tên** vào đơn (giá ghế, giá combo, tên phim, tên phòng…) để lịch sử không đổi khi danh mục thay đổi.
4. **Không xóa cứng** dữ liệu có liên quan tài chính; dùng soft delete / trạng thái.
5. **Idempotency**: tạo đơn, thanh toán, webhook, hoàn tiền dùng `Idempotency-Key`.
6. **Audit log** mọi thao tác: đăng nhập admin, sửa giá, duyệt/từ chối, hoàn tiền, đổi quyền, khóa tài khoản, cấu hình thanh toán, hoàn tác check-in (ai, khi nào, IP, giá trị trước/sau).
7. **Bảo mật dữ liệu cá nhân**: chỉ chủ đơn/nhân viên có quyền xem biên lai và thông tin liên hệ; ảnh biên lai không công khai (URL ký có hạn hoặc cần xác thực); che một phần email/SĐT trong danh sách.
8. **Múi giờ**: hiển thị theo giờ rạp; so sánh thời gian ở backend.
9. **Giới hạn tần suất (rate limit)**: đăng nhập, quên mật khẩu, áp voucher, tạo đơn, check-in.
10. **Chính sách & pháp lý**: điều khoản sử dụng, chính sách hoàn/hủy, bảo mật (theo Nghị định 13/2023/NĐ-CP về bảo vệ dữ liệu cá nhân), quy chế rạp; khách phải đồng ý khi đăng ký/đặt vé.

---

## 6. MÔ HÌNH DỮ LIỆU (TỔNG QUAN)

### 6.1 Danh sách bảng
**Người dùng & bảo mật**: `users`, `roles`, `permissions`, `role_permissions`, `user_roles`(hoặc `users.role`), `refresh_tokens`, `password_resets`, `email_verifications`, `audit_logs`
**Danh mục rạp**: `cinemas`, `rooms`, `seats`, `movies`, `genres`, `movie_genres`, `showtimes`, `showtime_seat_blocks` (chặn ghế theo suất), `price_rules`
**Bán hàng**: `bookings`, `booking_seats`, `booking_combos`, `payments`, `payment_proofs`, `payment_settings`, `payment_exceptions`, `refunds`, `booking_changes`, `tickets`, `checkin_logs`, `shifts`, `shift_cash_counts`
**Marketing**: `vouchers`, `voucher_usages`, `point_transactions`, `membership_tiers`, `banners`, `news`, `pages`, `reviews`, `favorites`
**Kho & combo**: `combos`, `combo_items`, `products`, `inventory_movements`
**Hệ thống**: `notifications`, `notification_templates`, `settings` (key/value), `jobs`/queue, `files`

### 6.2 Các ràng buộc then chốt
- `booking_seats`: cột `showtime_id`, `seat_id`, `booking_id`, `price`, **`active TINYINT`** (1 khi đang giữ/bán, NULL khi nhả) và `UNIQUE(showtime_id, seat_id, active)` – hoặc bảng `seat_locks` riêng có `UNIQUE(showtime_id, seat_id)` và xóa dòng khi nhả. Cách nào cũng phải **chặn ở mức DB**.
- `tickets.ticket_code UNIQUE`; `bookings.booking_code UNIQUE`; `movies.slug UNIQUE`; `vouchers.code UNIQUE`; `payments(provider, provider_txn_id) UNIQUE`.
- Khóa ngoại dùng `ON DELETE RESTRICT` cho dữ liệu tài chính (phim→suất→đơn); `CASCADE` chỉ cho bảng con thuần túy (vd `booking_seats` theo `booking`).
- Các cột tiền: `INT UNSIGNED`/`BIGINT` (VND, không dùng FLOAT). `movies.rating DECIMAL(3,1)`.
- Mọi bảng chính có `created_at`, `updated_at`; bảng danh mục có `deleted_at`.
- Chỉ mục: `showtimes(room_id, show_date, start_time)`, `bookings(user_id, created_at)`, `bookings(status, hold_expires_at)`, `tickets(booking_id)`, `payments(booking_id, status)`.

### 6.3 Trường chính của `bookings`
`id, booking_code, user_id (nullable), cinema_id, showtime_id, channel (online|pos), staff_id, shift_id, status, subtotal_tickets, subtotal_combos, discount_amount, points_discount, tax_amount, final_amount, voucher_id, customer_name, customer_email, customer_phone, hold_expires_at, confirmed_at, cancelled_at, cancel_reason, combo_picked_up_at, idempotency_key, created_at, updated_at`.

### 6.4 `payments`
`id, booking_id, provider, method (bank_qr|cash|card|vnpay|momo...), amount, status, provider_txn_id, transfer_content, proof_image_hash, submitted_at, reviewed_by, reviewed_at, reject_reason_code, reject_note, raw_payload (JSON), created_at`.

---

## 7. BÁO CÁO VÀ DASHBOARD

| Báo cáo | Chỉ số | Bộ lọc |
| --- | --- | --- |
| Tổng quan | Doanh thu hôm nay/tuần/tháng, số vé, số đơn, tỷ lệ lấp đầy, giá trị đơn trung bình, đơn chờ duyệt, đơn hết hạn | Rạp, khoảng ngày |
| Doanh thu | Theo ngày/tuần/tháng; theo kênh (online/POS); theo phương thức thanh toán; vé vs combo | Rạp, kênh |
| Phim | Doanh thu, số vé, tỷ lệ lấp đầy từng phim; top phim | Khoảng ngày |
| Suất chiếu | Lấp đầy theo khung giờ/phòng/định dạng | Rạp, phòng |
| Nhân viên/Ca | Doanh thu theo ca, chênh lệch tiền mặt, số vé bán | Nhân viên, ca |
| Voucher | Lượt dùng, tổng giảm, doanh thu phát sinh | Voucher |
| Hoàn/hủy | Số đơn hủy, tiền hoàn, lý do | Khoảng ngày |
| Khách hàng | Khách mới, khách quay lại, top chi tiêu | Hạng |
| Bắp nước | Số lượng bán, doanh thu, tồn kho | Danh mục |
| Soát vé | Tỷ lệ vé đã dùng/không đến (no-show), số lần quét lỗi | Suất |
Xuất **CSV/Excel/PDF**. Số liệu doanh thu tính theo **thời điểm thanh toán được xác nhận** (và trừ hoàn tiền theo thời điểm hoàn).
**Tỷ lệ lấp đầy** = ghế bán (`confirmed/completed`) ÷ tổng ghế bán được của các suất đã/đang chiếu trong kỳ.

---

## 8. YÊU CẦU PHI CHỨC NĂNG

| Nhóm | Yêu cầu |
| --- | --- |
| Hiệu năng | Trang danh sách < 2 giây (p95); sơ đồ ghế < 1 giây; chịu 500 người đặt đồng thời một suất hot |
| Tính nhất quán | Không bán trùng ghế dưới tải đồng thời (có test) |
| Sẵn sàng | Job và webhook chịu được khởi động lại; xử lý lại an toàn |
| Bảo mật | Băm mật khẩu bcrypt/argon2; HTTPS; CORS chặt; header bảo mật; validate đầu vào; chống CSRF khi dùng cookie; upload kiểm tra MIME + kích thước + đổi tên ngẫu nhiên; không lưu secret trong repo |
| Riêng tư | Che dữ liệu cá nhân, quyền truy cập biên lai, xóa/ẩn danh khi có yêu cầu |
| Khả dụng | Responsive, hỗ trợ trình duyệt hiện đại (Chrome/Safari/Edge/Firefox 2 phiên bản gần nhất), a11y WCAG 2.1 AA ở mức hợp lý |
| Quan sát | Log có cấu trúc, request-id, cảnh báo job lỗi, endpoint `/health` và `/ready` |
| Sao lưu | Backup DB hằng ngày + thư mục upload; thử khôi phục định kỳ |
| i18n | Chuỗi giao diện tách file; mặc định `vi`, MAY thêm `en` |

---

## 9. DANH MỤC MÃ LỖI NGHIỆP VỤ (DÙNG CHUNG FE/BE)

Định dạng phản hồi lỗi: `{ "error": { "code": "SEAT_UNAVAILABLE", "message": "…", "details": { … } } }`

| Mã | HTTP | Ý nghĩa |
| --- | :-: | --- |
| `VALIDATION_ERROR` | 422 | Dữ liệu không hợp lệ (kèm `details.fields`) |
| `UNAUTHENTICATED` | 401 | Chưa/không hợp lệ token |
| `FORBIDDEN` | 403 | Thiếu quyền |
| `ACCOUNT_LOCKED` | 423 | Tài khoản bị khóa |
| `TOO_MANY_REQUESTS` | 429 | Vượt giới hạn tần suất |
| `NOT_FOUND` | 404 | Không tìm thấy |
| `SHOWTIME_CLOSED` | 409 | Suất đã đóng bán/đã chiếu |
| `SHOWTIME_OVERLAP` | 409 | Trùng giờ phòng chiếu |
| `SEAT_UNAVAILABLE` | 409 | Ghế đã bị giữ/bán |
| `SEAT_LIMIT_EXCEEDED` | 422 | Vượt số ghế tối đa |
| `HOLD_EXPIRED` | 410 | Đơn giữ ghế hết hạn |
| `HOLD_LIMIT_EXCEEDED` | 429 | Quá số đơn giữ đồng thời |
| `BOOKING_INVALID_STATE` | 409 | Thao tác không hợp lệ với trạng thái đơn |
| `PAYMENT_PROOF_REQUIRED` | 422 | Thiếu biên lai |
| `PAYMENT_PROOF_DUPLICATE` | 409 | Biên lai trùng |
| `PAYMENT_AMOUNT_MISMATCH` | 409 | Sai số tiền |
| `VOUCHER_*` | 422 | Xem 3.9 |
| `COMBO_OUT_OF_STOCK` | 409 | Hết hàng |
| `TICKET_*` | 200/409 | Xem 3.10 (kết quả check-in trả 200 với `result`) |
| `REFUND_NOT_ALLOWED` | 409 | Ngoài chính sách hoàn |
| `IDEMPOTENCY_CONFLICT` | 409 | Cùng khóa nhưng khác nội dung |

---

## 10. CẤU HÌNH HỆ THỐNG (bảng `settings`, sửa được từ Admin)

| Khóa | Mặc định | Ý nghĩa |
| --- | --- | --- |
| `hold_minutes` | 10 | Thời gian giữ ghế |
| `hold_extend_minutes` | 5 | Gia hạn 1 lần |
| `proof_pending_max_minutes` | 60 | SLA chờ duyệt |
| `max_seats_per_booking` | 8 | |
| `max_active_holds_per_user` | 3 | |
| `online_close_before_minutes` | 15 | Đóng bán online trước giờ chiếu |
| `pos_close_before_minutes` | 0 | |
| `room_cleaning_minutes` | 15 | |
| `ads_minutes` | 15 | |
| `checkin_open_before_minutes` | 60 | |
| `checkin_late_allow_minutes` | 30 | |
| `customer_cancel_allowed` | false | |
| `customer_cancel_before_hours` | 2 | |
| `customer_refund_percent` | 0 | |
| `points_per_vnd` | 1/10000 | |
| `vat_percent` | 8 | |
| `price_rounding` | 1000 | |
| `payment_confirm_mode` | manual | manual / auto |
| `receipt_max_mb` | 5 | |
| `login_max_attempts` | 5 | |

---

## 11. DANH SÁCH KIỂM TRA NGHIỆP VỤ (ACCEPTANCE)

- [ ] Hai khách cùng chọn một ghế: chỉ một người giữ được, người kia nhận `SEAT_UNAVAILABLE`.
- [ ] Đơn `holding` quá 10 phút tự `expired` và ghế được nhả; đồng hồ FE khớp giờ server.
- [ ] Đơn đã gửi biên lai giữ ghế đến khi duyệt/từ chối/hết SLA.
- [ ] Từ chối biên lai có lý do; khách gửi lại được; từ chối quá hạn → `expired`.
- [ ] Duyệt đơn → sinh đúng N vé, mỗi vé 1 QR duy nhất, email gửi thành công.
- [ ] Check-in lần 2 cùng mã trả `ALREADY_USED` + thời điểm; hai máy quét đồng thời chỉ 1 thành công.
- [ ] Check-in ngoài khung giờ trả `TOO_EARLY`/`TOO_LATE`.
- [ ] Tạo suất trùng giờ/thiếu thời gian dọn phòng bị chặn.
- [ ] Xóa phim có suất/đơn bị chặn (chỉ ẩn/`ended`).
- [ ] Voucher: hết lượt, hết hạn, sai điều kiện đều bị từ chối; hủy đơn trả lượt.
- [ ] Hủy suất → danh sách đơn bị ảnh hưởng, hoàn tiền hàng loạt, thông báo.
- [ ] Hoàn tiền một phần cập nhật vé/ghế/điểm/voucher đúng.
- [ ] Mỗi thao tác nhạy cảm có dòng audit log.
- [ ] Báo cáo doanh thu khớp tổng đơn `paid` trừ hoàn tiền.
- [ ] Quên mật khẩu, khóa đăng nhập sau 5 lần sai hoạt động đúng.
- [ ] POS: mở/đóng ca, in vé, tiền mặt khớp báo cáo ca.
