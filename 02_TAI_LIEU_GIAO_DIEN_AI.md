# TÀI LIỆU GIAO DIỆN (UI/UX) DÀNH CHO AI THỰC THI – PXU CINEMA

> Dành cho AI agent dựng giao diện **React + Vite + Tailwind CSS**. Đọc kèm `01_TAI_LIEU_NGHIEP_VU.md` (tên trạng thái, mã lỗi, quy tắc) và `03_TAI_LIEU_KY_THUAT_FRONTEND_AI.md` (cấu trúc code, API).
> Phạm vi: **Website khách hàng** và **giao diện Admin/Staff** (cùng design system). Chi tiết kỹ thuật Admin xem file 04.
> Quy ước: **MUST** bắt buộc · **SHOULD** nên · **MAY** tùy chọn.

---

## 1. NGUYÊN TẮC THIẾT KẾ

1. **Phong cách**: Cinema Dark Mode sang trọng, lấy cảm hứng Galaxy Cinema nhưng **không sao chép nguyên bản** (dùng thương hiệu PXU, không dùng logo/ảnh của bên thứ ba).
2. **Mobile-first**: thiết kế từ 360px lên; ≥ 70% lượt đặt vé đến từ điện thoại. Mọi thao tác chính dùng được bằng một tay.
3. **Rõ ràng trước, đẹp sau**: luồng đặt vé ít bước, luôn hiển thị *đang ở bước nào*, *tổng tiền*, *thời gian giữ ghế còn lại*.
4. **Phản hồi tức thì**: skeleton khi tải, trạng thái nút `loading`, thông báo toast thành công/thất bại, không để người dùng đoán.
5. **Phòng lỗi**: vô hiệu hóa hành động không hợp lệ kèm lý do; xác nhận trước thao tác không hoàn tác (hủy, hoàn tiền, khóa tài khoản).
6. **Nhất quán**: dùng component chung, không viết kiểu inline ngẫu hứng. Mọi màu/khoảng cách lấy từ **design token**.
7. **Truy cập (a11y)**: tương phản đủ, điều khiển bằng bàn phím, nhãn cho icon/ảnh, không chỉ dùng màu để biểu thị trạng thái.

---

## 2. DESIGN SYSTEM

### 2.1 Màu sắc (token – đặt trong `tailwind.config.js`/CSS variables)
| Token | Giá trị | Dùng cho |
| --- | --- | --- |
| `bg-base` | `#0b0f19` | Nền trang |
| `bg-surface` | `#0f172a` | Thẻ, panel |
| `bg-elevated` | `#1e293b` | Modal, dropdown, hàng hover |
| `border-subtle` | `#334155` | Viền |
| `text-primary` | `#f8fafc` | Chữ chính |
| `text-secondary` | `#cbd5e1` | Chữ phụ (≥ 4.5:1 trên nền) |
| `text-muted` | `#94a3b8` | Chú thích (chỉ cỡ ≥ 14px) |
| `brand` | `#ff6600` (hover `#ea580c`, active `#c2410c`) | CTA chính, nhấn mạnh |
| `gold` | `#fbbf24` | VIP, đánh giá sao, điểm thưởng |
| `success` | `#22c55e` | Thành công, vé hợp lệ |
| `warning` | `#f59e0b` | Cảnh báo, sắp hết giờ |
| `danger` | `#ef4444` | Lỗi, hủy, vé không hợp lệ |
| `info` | `#38bdf8` | Thông tin, ghế đang chọn |
| `couple` | `#ec4899` | Ghế đôi |
Chế độ sáng cho khu vực **Admin** (SHOULD): cung cấp `data-theme="light"` với bộ token tương ứng; khách hàng mặc định tối.
Hiệu ứng nhấn: glow cam nhẹ `0 0 24px rgba(255,102,0,.25)` cho CTA/poster hover; **không** lạm dụng neon ở vùng đọc nhiều chữ.

### 2.2 Chữ
- Font: `Be Vietnam Pro` hoặc `Inter` (hỗ trợ tiếng Việt đủ dấu), tiêu đề `Barlow Condensed`/`Oswald` tùy chọn cho tên phim/hero. Tải bằng `font-display: swap`.
- Thang cỡ chữ: `xs 12 · sm 14 · base 16 · lg 18 · xl 20 · 2xl 24 · 3xl 30 · 4xl 36 · 5xl 48`. Đoạn văn `line-height 1.6`, tiêu đề `1.2`.
- Số tiền: dùng `tabular-nums`, định dạng `85.000đ` (`Intl.NumberFormat('vi-VN')`).

### 2.3 Khoảng cách, bo góc, đổ bóng
- Lưới 4px: `1(4) 2(8) 3(12) 4(16) 6(24) 8(32) 12(48) 16(64)`.
- Bo góc: nút/input `8px`, thẻ `12px`, modal `16px`, ghế `6px` trên / `2px` dưới.
- Đổ bóng: `sm` cho thẻ, `lg` cho modal; tránh bóng đậm trên nền tối (dùng viền + glow).

### 2.4 Breakpoint & lưới
| Tên | Rộng | Cột | Ghi chú |
| --- | --- | --- | --- |
| mobile | < 640 | 4 | Một cột, thanh điều hướng đáy cho khách (SHOULD) |
| tablet | 640–1023 | 8 | Lưới phim 3 cột |
| desktop | 1024–1439 | 12 | Container tối đa 1200px |
| wide | ≥ 1440 | 12 | Container 1320px |

### 2.5 Component dùng chung (thư mục `components/ui`)
`Button` (primary/secondary/ghost/danger, sizes, loading, icon) · `IconButton` · `Input` · `Textarea` · `Select` · `Checkbox` · `Radio` · `Switch` · `DatePicker`/`DateStrip` · `Badge` · `Chip` · `Card` · `Modal`/`Drawer`(bottom sheet trên mobile) · `Tabs` · `Accordion` · `Tooltip` · `Toast` · `Skeleton` · `Spinner` · `EmptyState` · `ErrorState` · `Pagination` · `Table` (sort, chọn, sticky header, responsive → thẻ trên mobile) · `Stepper` · `Countdown` · `QRImage` · `FileUploader`(kéo thả + xem trước) · `ConfirmDialog` · `Breadcrumb` · `Avatar` · `Rating` · `PriceText` · `AgeBadge` · `StatusBadge`.

**Bảng màu `StatusBadge`** (dùng thống nhất):
| Giá trị | Nhãn tiếng Việt | Màu |
| --- | --- | --- |
| `holding` | Đang giữ ghế | warning |
| `proof_submitted` | Chờ duyệt | info |
| `confirmed` / `paid` | Đã thanh toán | success |
| `rejected` | Biên lai bị từ chối | danger |
| `expired` | Hết hạn | slate |
| `cancelled` | Đã hủy | danger-nhạt |
| `completed` | Đã hoàn thành | slate-success |
| `refunded` / `partially_refunded` | Đã hoàn tiền / Hoàn một phần | purple |
| `valid` | Chưa sử dụng | success |
| `used` | Đã sử dụng | slate |
**AgeBadge**: `P` xanh lá, `K` xanh dương, `T13` vàng, `T16` cam, `T18` đỏ. Luôn kèm chữ, không chỉ màu.

### 2.6 Icon & hình ảnh
- Icon: `lucide-react` (stroke 1.75). Không dùng emoji làm icon chức năng trong UI chính (chỉ trong nội dung/khuyến mãi nếu cần).
- Poster tỉ lệ **2:3**, banner **16:9** (hero 21:9 trên desktop). Dùng `loading="lazy"`, `srcset`/ảnh WebP, `aspect-ratio` để không nhảy layout, ảnh dự phòng khi lỗi.

### 2.7 Chuyển động
- Thời lượng 150–250ms, easing `ease-out`. Hiệu ứng: hover nâng poster, trượt modal, fade trang. Tôn trọng `prefers-reduced-motion` (tắt chuyển động lớn, confetti, auto-slide).
- Confetti chỉ khi đặt vé thành công, chạy một lần.

---

## 3. SƠ ĐỒ TRANG (SITEMAP) & ĐIỀU HƯỚNG

### 3.1 Website khách hàng (`/`)
| Route | Trang | Yêu cầu đăng nhập |
| --- | --- | :-: |
| `/` | Trang chủ | ✖ |
| `/phim` | Danh sách phim (đang chiếu/sắp chiếu, lọc, tìm kiếm) | ✖ |
| `/phim/:slug` | Chi tiết phim + lịch chiếu | ✖ |
| `/lich-chieu` | Lịch chiếu theo ngày/rạp | ✖ |
| `/rap` , `/rap/:id` | Danh sách rạp/chi tiết rạp (nếu đa chi nhánh) | ✖ |
| `/bap-nuoc` | Menu bắp nước | ✖ |
| `/khuyen-mai` , `/khuyen-mai/:slug` | Tin tức/ưu đãi | ✖ |
| `/dat-ve/:showtimeId/ghe` | Chọn ghế | ✔ (hoặc chuyển đăng nhập kèm `returnTo`) |
| `/dat-ve/:bookingCode/bap-nuoc` | Chọn combo | ✔ |
| `/dat-ve/:bookingCode/xac-nhan` | Xem lại đơn + voucher | ✔ |
| `/thanh-toan/:bookingCode` | Thanh toán QR | ✔ |
| `/ve/:bookingCode` | Kết quả/Vé điện tử | ✔ |
| `/tai-khoan` | Hồ sơ | ✔ |
| `/tai-khoan/ve-cua-toi` | Lịch sử vé | ✔ |
| `/tai-khoan/diem-thuong` | Điểm & hạng | ✔ |
| `/tai-khoan/thong-bao` | Thông báo | ✔ |
| `/dang-nhap` `/dang-ky` `/quen-mat-khau` `/dat-lai-mat-khau/:token` `/xac-thuc-email/:token` | Xác thực | ✖ |
| `/trang/:slug` | Trang tĩnh (điều khoản, chính sách, FAQ, liên hệ, quy chế) | ✖ |
| `*` | 404 | – |

### 3.2 Staff (`/staff`)
`/staff/quet-ve` (Scanner) · `/staff/ban-ve` (POS) · `/staff/ve-trong-ngay` · `/staff/ca-lam-viec` (mở/đóng ca)

### 3.3 Admin (`/admin`)
`/admin` (Dashboard) · `/admin/phim` · `/admin/phim/:id` · `/admin/suat-chieu` · `/admin/rap` · `/admin/phong` · `/admin/phong/:id/so-do-ghe` · `/admin/bang-gia` · `/admin/combo` · `/admin/don-hang` · `/admin/don-hang/:id` · `/admin/duyet-bien-lai` · `/admin/hoan-tien` · `/admin/giao-dich-bat-thuong` · `/admin/voucher` · `/admin/thanh-vien` · `/admin/nguoi-dung` · `/admin/thanh-toan-qr` · `/admin/banner-tin-tuc` · `/admin/danh-gia` · `/admin/bao-cao` · `/admin/nhat-ky` · `/admin/cai-dat`

### 3.4 Điều hướng
- **Navbar khách (desktop)**: Logo · Phim · Lịch chiếu · Rạp · Bắp nước · Khuyến mãi · [Tìm kiếm] · [Chuông thông báo] · Avatar/Đăng nhập. Mobile: logo + hamburger (drawer) + thanh tab đáy: *Trang chủ, Phim, Vé của tôi, Tài khoản*.
- Avatar menu: Hồ sơ, Vé của tôi, Điểm thưởng, (Quét vé nếu staff), (Quản trị nếu admin/manager), Đăng xuất.
- **Admin layout**: Sidebar trái thu gọn được (mobile: drawer) + Topbar (breadcrumb, chọn rạp, tìm nhanh, thông báo, avatar) + vùng nội dung. Mục menu ẩn theo quyền.
- Sau đăng nhập: customer → quay lại `returnTo` hoặc `/`; staff → `/staff/quet-ve`; admin/manager → `/admin`.

---

## 4. ĐẶC TẢ TỪNG MÀN HÌNH – KHÁCH HÀNG

> Mỗi màn hình MUST có đủ 4 trạng thái: **loading (skeleton)**, **empty**, **error (có nút thử lại)**, **success/data**.

### 4.1 Trang chủ `/`
**Khối theo thứ tự:**
1. **Hero slider**: banner 16:9 (desktop 21:9), tự chuyển 6 giây, dừng khi hover/focus, có chấm điều hướng + nút trước/sau; mỗi slide có tên phim, nhãn tuổi, thể loại, nút **Mua vé** (cam) và **Xem trailer**. Ảnh đầu tiên preload.
2. **Quick Booking** (đè lên đáy hero trên desktop; thẻ riêng trên mobile): Chọn *Phim → Rạp → Ngày → Suất* → nút **Mua vé ngay**. Mỗi ô phụ thuộc ô trước, disable khi chưa chọn, tải dữ liệu theo cấp.
3. **Tabs Phim đang chiếu / Sắp chiếu** + bộ lọc thể loại (chip cuộn ngang) + nút "Xem tất cả".
   - Desktop lưới 5 cột, tablet 3, mobile 2 (hoặc carousel 1,5 thẻ).
4. **Ưu đãi/Combo nổi bật** (3–4 thẻ).
5. **Tin tức & khuyến mãi** (3 thẻ).
6. **Lợi ích thành viên/CTA tải/ đăng ký** (SHOULD).
7. **Footer**.

**MovieCard**: poster 2:3, `AgeBadge` góc trái trên, điểm sao góc phải; tên phim (2 dòng, cắt `…`), thể loại · thời lượng; hover (desktop): lớp phủ tối + nút **Mua vé** & **Trailer**; mobile: nút **Mua vé** cố định dưới thẻ. Toàn thẻ là link tới chi tiết.

### 4.2 Danh sách phim `/phim`
- Tabs trạng thái, ô tìm kiếm (debounce 300ms), lọc: thể loại (nhiều), nhãn tuổi, định dạng, sắp xếp (mới nhất, điểm cao, A-Z). Giữ bộ lọc trên URL query.
- Phân trang hoặc "Tải thêm" (infinite scroll có nút dự phòng). Empty: "Không tìm thấy phim phù hợp" + nút xóa bộ lọc.

### 4.3 Chi tiết phim `/phim/:slug`
- **Hero mờ**: banner làm nền (blur + gradient tối), poster trái, thông tin phải: tên, nhãn tuổi, điểm, thời lượng, ngày khởi chiếu, thể loại, ngôn ngữ, đạo diễn, diễn viên, nút **Xem trailer** (modal YouTube, lazy tải iframe), **Mua vé** (cuộn tới lịch chiếu), nút yêu thích.
- **Tab**: *Nội dung* · *Lịch chiếu* · *Đánh giá* (P3).
- **Lịch chiếu**: `DateStrip` 14 ngày (Hôm nay, Ngày mai, T2…), dính (sticky) khi cuộn; lọc rạp; nhóm theo **định dạng** (2D Phụ đề/Lồng tiếng/3D…) → hàng nút giờ.
  - **Nút giờ**: hiển thị `19:30`, dòng nhỏ giá "từ 85.000đ" và số ghế còn; màu theo mức lấp đầy: nhiều ghế (viền trung tính) → sắp hết (< 20%, viền cam) → hết (mờ, gạch, không bấm) → đã qua/đóng bán (mờ, tooltip lý do).
  - Click nút giờ: nếu chưa đăng nhập → modal "Đăng nhập để tiếp tục" giữ `returnTo`; nếu T16/T18 → modal cảnh báo độ tuổi (xác nhận); rồi vào chọn ghế.
- Phim liên quan ở cuối.

### 4.3b Lịch chiếu `/lich-chieu`
Chọn ngày + rạp, danh sách phim theo ngày, mỗi phim một hàng gồm poster nhỏ + các nút giờ. Lọc theo định dạng/thể loại.

### 4.4 Chọn ghế `/dat-ve/:showtimeId/ghe`
**Bố cục desktop**: trái 2/3 sơ đồ; phải 1/3 **thẻ tóm tắt dính (sticky)**. **Mobile**: sơ đồ full width, thẻ tóm tắt **bottom bar dính** (tổng tiền + nút Tiếp tục; chạm mở bottom sheet chi tiết).
- **Header bước**: `Stepper` 5 bước: *Chọn ghế → Bắp nước → Xác nhận → Thanh toán → Nhận vé*.
- **Thông tin suất**: poster nhỏ, tên phim, nhãn tuổi, rạp • phòng, ngày • giờ • định dạng. Nút "Đổi suất".
- **Màn hình (screen)**: đường cong phát sáng cam/trắng phía trên, chữ "MÀN HÌNH".
- **Sơ đồ ghế**: render theo `grid_x/grid_y` từ API (có lối đi/ô khuyết). Nhãn hàng bên trái & phải, số cột. Cho phép **pinch-zoom/kéo** trên mobile (SHOULD), nút +/−.
- **Chú giải (legend)**: Thường · VIP · Đôi · Đang chọn · Đã bán · Đang được giữ · Không khả dụng.
| Loại | Màu | Ghi chú |
| --- | --- | --- |
| Thường (available) | `#475569` nền, viền `#64748b` | |
| VIP | nền cam, viền `gold` | |
| Đôi (couple) | `couple` hồng, rộng gấp đôi | |
| Đang chọn | `info` sáng + dấu ✓ | |
| Đang được giữ (người khác) | gạch chéo, mờ | Cập nhật realtime |
| Đã bán | `#1e293b`, icon khóa/X | Không bấm |
| Hỏng/không khả dụng | gạch ngang, nhạt | Không bấm |
- **Tương tác**: chạm/click chọn-bỏ chọn; **hover/focus** hiện tooltip *"Ghế C05 · VIP · 105.000đ"*; điều khiển bằng bàn phím (mũi tên di chuyển, Space/Enter chọn); `aria-label` mỗi ghế ("Ghế C05, VIP, còn trống").
- **Quy tắc UI**: tối đa 8 ghế (toast + vô hiệu thêm); cảnh báo ghế lẻ (banner vàng, không chặn); ghế đôi chọn cả cặp.
- **Thẻ tóm tắt**: danh sách ghế đã chọn (chip, bấm × để bỏ), từng loại × giá, **Tạm tính**. Nút **Tiếp tục** (disabled nếu 0 ghế). Khi bấm: gọi tạo giữ ghế; nếu `SEAT_UNAVAILABLE` → highlight ghế bị mất, tải lại sơ đồ, thông báo "Rất tiếc, ghế A5 vừa có người khác chọn".
- **Realtime**: ghế đổi trạng thái cập nhật ngay (SSE/WebSocket hoặc polling 5 giây) không làm mất lựa chọn.
- **Xác nhận rời trang** khi đã có ghế đã giữ (đơn `holding`).

### 4.5 Chọn bắp nước `/dat-ve/:bookingCode/bap-nuoc`
- **Đồng hồ giữ ghế dính trên cùng** (xem 4.8) hiển thị ở mọi bước sau khi giữ ghế.
- Tabs danh mục: Combo · Bắp · Nước · Snack. Thẻ combo: ảnh, tên, mô tả, giá, **bộ đếm − 0 +** (tối đa theo giới hạn), nhãn "Hết hàng" nếu hết.
- Thanh tổng dính dưới: *Vé + Combo = Tạm tính* và nút **Tiếp tục** / **Bỏ qua**.

### 4.6 Xác nhận đơn `/dat-ve/:bookingCode/xac-nhan`
- Thẻ tóm tắt: phim/suất/rạp/phòng, ghế (kèm loại + giá), combo (số lượng × giá), phụ phí/thuế.
- **Voucher**: ô nhập mã + nút Áp dụng; kết quả hiển thị *−30.000đ* (xanh) hoặc lỗi rõ ràng theo mã `VOUCHER_*`; có thể gỡ mã. Gợi ý các voucher khả dụng (danh sách chip) (SHOULD).
- **Điểm thưởng**: công tắc "Dùng X điểm (−Y đ)" (SHOULD).
- **Thông tin liên hệ**: tên, email, SĐT (điền sẵn, có thể sửa), validate theo định dạng VN.
- **Chấp nhận điều khoản** (checkbox bắt buộc) + cảnh báo độ tuổi nếu có.
- **Tổng thanh toán** lớn, nổi bật. Nút **Thanh toán** (disabled đến khi hợp lệ).

### 4.7 Thanh toán QR `/thanh-toan/:bookingCode`
**Bố cục desktop 2 cột** (trái: QR; phải: thông tin & hành động). Mobile: dọc.
- **Banner trên cùng**: đồng hồ **MM:SS** đếm ngược lớn; ≤ 2 phút đổi sang màu `warning` và nhấp nháy nhẹ (không với reduced-motion); nút "Gia hạn +5 phút" (nếu cho phép).
- **Khung QR**: ảnh QR vuông, nền trắng viền bo, ≥ 240px; nhấp để phóng to toàn màn hình; nút **Tải ảnh QR**. Dưới QR: logo ngân hàng + "Quét bằng app ngân hàng".
- **Bảng thông tin** (mỗi dòng có nút **Sao chép** + toast "Đã sao chép"): Ngân hàng · Số tài khoản · Chủ tài khoản · **Số tiền** (nổi bật) · **Nội dung chuyển khoản** (nổi bật, cảnh báo "Giữ nguyên nội dung").
- **Hướng dẫn ngắn 3 bước** (accordion trên mobile).
- **Khu upload biên lai** (`FileUploader`): kéo thả/chọn ảnh/chụp (mobile), xem trước, nút xóa, kiểm tra loại/kích thước ngay (≤ 5MB), tiến trình tải.
- **Nút chính**: **Tôi đã chuyển khoản** (disabled nếu chưa có ảnh). Sau khi gửi → chuyển sang trạng thái **"Đang chờ xác nhận"** (xem 4.7b).
- **Mở app ngân hàng (mobile, SHOULD)**: nút deep link VietQR nếu hỗ trợ.
- Link phụ: "Hủy đơn & chọn lại ghế" (xác nhận), "Cần hỗ trợ?" (hotline).

### 4.7b Trạng thái sau khi gửi biên lai
Cùng trang `/thanh-toan/:bookingCode` hoặc `/ve/:bookingCode` hiển thị theo `payment.status`:
| Trạng thái | Giao diện |
| --- | --- |
| `proof_submitted` | Biểu tượng đồng hồ, "Chúng tôi đang xác nhận thanh toán của bạn (thường < X phút)"; hiển thị ghế được giữ; tự **polling/SSE**; cho xem ảnh đã gửi; có thể **thay ảnh** khi chưa duyệt |
| `paid` | Chuyển sang trang vé + confetti |
| `rejected` | Khung đỏ: **lý do từ chối** + hướng dẫn, nút **Gửi lại biên lai**, thời gian còn lại |
| `expired` | "Đơn đã hết hạn" + nút "Chọn lại ghế"; nếu đã chuyển tiền: "Liên hệ hỗ trợ kèm mã đơn" |
| Đơn 0đ | Bỏ qua QR, chỉ nút **Xác nhận đặt vé** |

### 4.8 Đồng hồ giữ ghế (component `HoldCountdown`)
- Nhận `hold_expires_at` + `server_time` → tính độ lệch → đếm theo giờ server. Hiển thị trong thanh dính (header của quy trình đặt vé).
- Còn ≤ 2 phút: đổi màu + toast nhắc một lần. Hết giờ: **modal chặn** "Đã hết thời gian giữ ghế" → nút "Chọn lại ghế" (về trang chọn ghế cùng suất).
- Tab ẩn rồi quay lại: tính lại theo giờ thực.

### 4.9 Vé điện tử `/ve/:bookingCode`
- **Đầu trang**: icon ✓ + "Đặt vé thành công!" + confetti (một lần) + mã đơn.
- **Vé** (mỗi ghế một vé, carousel trên mobile hoặc xếp dọc): thiết kế vé có **viền răng cưa/khuyết bán nguyệt hai bên**, nửa trên poster + thông tin, nửa dưới **QR** (≥ 200px, nền trắng) + mã vé dạng chữ.
  - Hiển thị: tên phim, nhãn tuổi, ngày giờ, rạp • phòng, ghế, định dạng, ghi chú (ưu đãi HSSV...), mã đơn.
  - **Trạng thái vé** (`StatusBadge`): Chưa sử dụng / Đã sử dụng (làm mờ + watermark "ĐÃ SỬ DỤNG") / Hết hạn / Đã hủy.
- **Combo**: khối "Nhận bắp nước tại quầy" kèm QR đơn (nếu có).
- **Hành động**: **Lưu vé (ảnh/PDF)**, **In vé**, **Gửi lại email**, **Thêm vào lịch** (.ics), **Chỉ đường** (SHOULD), **Về trang chủ**.
- Tăng độ sáng màn hình khi mở QR (SHOULD, `Screen Wake Lock`).
- Nếu đơn chưa `paid`: hiển thị trạng thái tương ứng (4.7b) thay vì vé.

### 4.10 Tài khoản
- **Hồ sơ**: avatar, tên, email (kèm trạng thái xác thực), SĐT, ngày sinh; form sửa; **Đổi mật khẩu** (modal); xóa/ẩn danh tài khoản (MAY).
- **Vé của tôi**: tabs *Sắp tới · Đã xem · Đã hủy/Hết hạn*; mỗi thẻ: poster nhỏ, tên phim, suất, ghế, `StatusBadge`, nút **Xem vé**; đơn đang chờ thanh toán có nút **Tiếp tục thanh toán** + đồng hồ. Lọc theo ngày, tìm theo mã đơn.
- **Điểm thưởng**: điểm hiện có, hạng + thanh tiến độ lên hạng, lịch sử cộng/trừ, quyền lợi.
- **Thông báo**: danh sách, đánh dấu đã đọc, lọc theo loại.

### 4.11 Đăng nhập / Đăng ký / Quên mật khẩu
- Bố cục: ảnh poster mờ bên trái (ẩn trên mobile), form bên phải trong thẻ bo góc.
- Đăng nhập: email, mật khẩu (hiện/ẩn), "Ghi nhớ", link quên mật khẩu, (MAY) nút đăng nhập Google; hiển thị lỗi chung "Email hoặc mật khẩu không đúng", thông báo khóa tạm kèm thời gian còn lại.
- Đăng ký: tên, email, SĐT, mật khẩu (thanh độ mạnh), nhập lại mật khẩu, đồng ý điều khoản. Validate thời gian thực (onBlur) + tóm tắt lỗi khi submit.
- Quên mật khẩu: nhập email → màn hình xác nhận trung tính. Đặt lại: mật khẩu mới + xác nhận, xử lý token hết hạn.

### 4.12 Trang tĩnh, 404, lỗi
- Trang tĩnh: bố cục đọc (max 720px), mục lục tự sinh. 404: minh họa vé/ghế rỗng + nút về trang chủ. 500/offline: thông báo thân thiện + thử lại. Banner offline khi mất mạng.

---

## 5. ĐẶC TẢ GIAO DIỆN STAFF

### 5.1 Quét vé `/staff/quet-ve`
- Bố cục tối ưu **điện thoại/máy tính bảng ngang**. Camera chiếm phần lớn màn hình với khung ngắm, đèn flash/đổi camera, nút **Tạm dừng**.
- Ô **nhập mã thủ công** + nút Kiểm tra. Chọn **suất chiếu hiện tại** (mặc định các suất trong ±2 giờ) để lọc và cảnh báo vé sai suất.
- **Kết quả toàn màn hình hoặc panel lớn** (kèm âm thanh/rung):
  - ✅ **HỢP LỆ** (nền xanh): tên phim, phòng, ghế, giờ chiếu, số vé còn lại trong đơn; nút **Cho vào**(tự động nếu bật chế độ nhanh) và **Check-in cả đơn**.
  - ⚠️ **ĐÃ SỬ DỤNG** (nền vàng): hiện giờ + người check-in.
  - ⏱ **CHƯA ĐẾN GIỜ / QUÁ GIỜ** (nền cam): kèm khung giờ hợp lệ; manager có nút Override (cần PIN).
  - ❌ **KHÔNG HỢP LỆ / CHƯA THANH TOÁN / ĐÃ HỦY / SAI RẠP** (nền đỏ).
  - ⚠️ **KIỂM TRA GIẤY TỜ**: modal nhắc kiểm tra (HSSV/độ tuổi) + nút **Đã kiểm tra**.
- Tự động đóng kết quả sau 3 giây và sẵn sàng quét tiếp (cấu hình được). **Lịch sử quét** gần đây (20 dòng) bên dưới/ bên phải. Bộ đếm: đã vào/tổng vé của suất đang chọn.
- Yêu cầu quyền camera: màn hướng dẫn nếu bị từ chối; nhắc cần HTTPS.

### 5.2 POS `/staff/ban-ve`
- **Bố cục 3 vùng (tablet/desktop)**: (1) chọn phim/suất (cột trái), (2) sơ đồ ghế/combo (giữa), (3) **giỏ hàng + thanh toán** (phải). Mobile: từng bước.
- Phím tắt (desktop): `F2` tìm phim, `F4` combo, `F8` thanh toán, `Esc` hủy.
- **Thanh toán**: chọn *Tiền mặt / Chuyển khoản QR / Thẻ / Hỗn hợp*; tiền mặt có ô "Khách đưa" tự tính **tiền thừa** và nút mệnh giá nhanh (50k/100k/200k/500k); QR hiển thị mã tương tự 4.7 nhưng chờ staff bấm **Đã nhận tiền**.
- **Sau khi bán**: màn hình thành công + **In vé** (khổ 80mm) + **Giao dịch mới**. Chế độ "In lại".
- Thanh trạng thái: tên nhân viên, **ca hiện tại**, tổng bán trong ca. Chưa mở ca → modal **Mở ca**.

### 5.3 Vé trong ngày `/staff/ve-trong-ngay`
Bảng đơn/vé của ngày, lọc theo suất, trạng thái, tìm theo mã/SĐT; xem chi tiết; in lại vé.

### 5.4 Ca làm việc
Mở ca (tiền đầu ca) → bảng tóm tắt → Đóng ca (đếm tiền theo mệnh giá, hệ thống tính chênh lệch, ghi chú) → in/xuất báo cáo ca.

---

## 6. ĐẶC TẢ GIAO DIỆN ADMIN

### 6.1 Khung chung
- **Layout**: sidebar nhóm menu *Tổng quan · Bán hàng · Danh mục · Marketing · Hệ thống*; topbar có **bộ chọn rạp** (nếu nhiều rạp), tìm nhanh (Ctrl+K), chuông thông báo (số biên lai chờ), avatar.
- **Mẫu trang danh sách** (dùng cho mọi module): tiêu đề + nút **Thêm mới** → thanh lọc (tìm kiếm, lọc nhanh, khoảng ngày, trạng thái) → bảng (sort, chọn nhiều, thao tác hàng loạt, cột có thể ẩn) → phân trang + chọn số dòng. Mobile: bảng chuyển thành thẻ.
- **Mẫu form**: bố cục 1–2 cột, nhóm trường có tiêu đề, validate inline, nút **Lưu / Hủy** dính cuối, cảnh báo thay đổi chưa lưu, hiển thị lỗi từ server theo trường.
- **Xác nhận thao tác nguy hiểm**: `ConfirmDialog` mô tả hậu quả; thao tác rất nguy hiểm (xóa, khóa, hoàn tiền) yêu cầu nhập lại từ khóa hoặc lý do.
- **Quyền**: phần tử/nút không có quyền thì ẩn (menu) hoặc disable + tooltip (nút).

### 6.2 Dashboard `/admin`
- Hàng thẻ KPI: Doanh thu hôm nay (so với hôm qua %), Doanh thu tháng, Vé bán hôm nay, Tỷ lệ lấp đầy, Đơn chờ duyệt (nhấp → hàng đợi), Đơn hết hạn.
- Biểu đồ: **đường** doanh thu 7/30 ngày (chuyển đổi), **cột** vé theo khung giờ, **tròn/donut** online vs POS, **cột ngang** top 5 phim.
- Bảng: đơn mới nhất, suất chiếu sắp diễn ra (lấp đầy), cảnh báo (combo sắp hết, giao dịch bất thường, job lỗi).
- Bộ lọc: rạp, khoảng ngày. Có skeleton + làm mới tự động mỗi 60 giây.

### 6.3 Phim `/admin/phim`
Bảng: poster nhỏ, tên, thể loại, thời lượng, nhãn tuổi, ngày khởi chiếu, trạng thái (toggle), thao tác. Form: các trường 3.2 nghiệp vụ, **upload poster/banner** với crop theo tỉ lệ + xem trước, trình xem trước trailer YouTube, chọn thể loại nhiều, chọn trạng thái. Xóa: nếu có suất/đơn → thông báo và đề nghị "Ẩn/Kết thúc".

### 6.4 Suất chiếu `/admin/suat-chieu`
- Hai chế độ xem: **Lịch (timeline theo phòng)** – trục ngang là giờ, mỗi hàng một phòng, khối màu theo phim, kéo để tạo, nhấp để sửa; và **Danh sách**.
- Form tạo: phim, rạp/phòng, ngày, giờ bắt đầu → **tự tính giờ kết thúc**, định dạng, giá nền, ngày mở bán. **Cảnh báo trùng/thiếu thời gian dọn phòng ngay khi chọn** (đỏ + chi tiết suất xung đột), chặn lưu.
- **Tạo hàng loạt**: chọn phim, phòng, khoảng ngày, nhiều giờ → bảng xem trước + đánh dấu xung đột → xác nhận.
- Hủy/dời suất: mở trang **ảnh hưởng** liệt kê đơn bị tác động + hành động hàng loạt (3.12 nghiệp vụ).

### 6.5 Rạp & Phòng & Sơ đồ ghế
- Danh sách rạp/phòng; trang **Trình dựng sơ đồ ghế**: lưới có thể kéo thả; công cụ chọn *Thường / VIP / Đôi / Lối đi / Khuyết / Hỏng*; tô nhiều ô bằng kéo; đặt tên hàng; sinh nhanh theo số hàng × cột; **xem trước như khách**; thống kê số ghế theo loại. Cảnh báo ảnh hưởng đơn đã bán khi sửa.

### 6.6 Bảng giá `/admin/bang-gia`
Ma trận giá: loại ngày × khung giờ × loại phòng/định dạng; phụ thu loại ghế; đối tượng ưu đãi; **mô phỏng giá** (chọn suất + ghế → hiển thị giá). Xem trước ngày hiệu lực.

### 6.7 Combo `/admin/combo`
Bảng + form (ảnh, giá, danh mục, tồn kho, giới hạn, trạng thái). Cảnh báo tồn kho thấp. Lịch sử nhập/xuất (P2).

### 6.8 Đơn hàng `/admin/don-hang`
- Lọc: trạng thái đơn, trạng thái thanh toán, kênh, rạp, khoảng ngày, voucher, tìm (mã, email, SĐT).
- Chi tiết đơn (drawer rộng/ trang): thông tin khách, phim/suất, ghế & vé (trạng thái từng vé), combo, thanh toán (các lần), biên lai (xem phóng to, tải), **dòng thời gian (timeline)** mọi sự kiện, ghi chú nội bộ.
- Hành động theo trạng thái: **Duyệt**, **Từ chối** (chọn lý do), **Hủy đơn**, **Hoàn tiền (toàn phần/ một phần theo ghế)**, **Gửi lại vé/email**, **Đổi ghế/đổi suất**, **Hoàn tác check-in** (manager).

### 6.9 Duyệt biên lai `/admin/duyet-bien-lai`
Màn hình **hàng đợi 2 khung**: trái danh sách đơn chờ (sắp theo cũ nhất, hiện thời gian chờ và thời hạn SLA, màu cảnh báo khi sắp hết); phải **ảnh biên lai** phóng to/xoay + khối đối chiếu (số tiền cần, nội dung, ngân hàng, thời gian gửi, cảnh báo ảnh trùng) + nút **Duyệt (Enter)** / **Từ chối (R)** / **Bỏ qua**. Phím tắt, tự chuyển sang đơn kế tiếp. Âm báo khi có biên lai mới (tùy chọn).

### 6.10 Hoàn tiền & Giao dịch bất thường
Bảng yêu cầu hoàn theo trạng thái; trang chi tiết: số tiền, lý do, người yêu cầu, người duyệt, mã giao dịch hoàn (nhập khi hoàn xong). Bảng giao dịch bất thường với hành động *Liên kết đơn / Khôi phục / Hoàn tiền / Bỏ qua*.

### 6.11 Voucher `/admin/voucher`
Bảng + form đầy đủ điều kiện (3.9); hiển thị lượt đã dùng/tổng; tạo mã hàng loạt; xem lịch sử sử dụng.

### 6.12 Thành viên / Người dùng
- `/admin/nguoi-dung`: tìm, lọc theo vai trò/trạng thái; đổi vai trò (xác nhận), khóa/mở khóa (nhập lý do), đặt lại mật khẩu (gửi email), xem lịch sử đơn/đăng nhập; **không cho tự hạ quyền/khóa chính mình**, không xóa admin cuối cùng.
- `/admin/thanh-vien`: hạng, điểm, điều chỉnh điểm thủ công (kèm lý do).

### 6.13 Thanh toán QR `/admin/thanh-toan-qr`
Hai cột: **Form** (chọn ngân hàng có logo, số tài khoản, tên chủ tài khoản, tiền tố nội dung, upload ảnh QR tùy chỉnh, công tắc "Dùng ảnh QR tải lên"/"VietQR tự động", chế độ xác nhận manual/auto, secret webhook) và **Xem trước trực tiếp** đúng giao diện khách thấy (QR + bảng thông tin với dữ liệu mẫu). Nút **Kiểm tra QR** (sinh thử). Mọi thay đổi cần xác nhận mật khẩu admin và ghi audit.

### 6.14 Banner / Tin tức / Trang tĩnh / Đánh giá
Quản lý nội dung (trình soạn thảo rich-text an toàn – sanitize), lịch đăng/hết hạn, sắp xếp banner kéo thả, kiểm duyệt đánh giá (ẩn/hiện).

### 6.15 Báo cáo `/admin/bao-cao`
Tabs theo mục 7 nghiệp vụ; bộ lọc chung; biểu đồ + bảng; nút **Xuất Excel/CSV/PDF**; so sánh kỳ trước (SHOULD).

### 6.16 Nhật ký `/admin/nhat-ky`
Bảng audit: thời gian, người thực hiện, hành động, đối tượng, IP; lọc; xem **diff trước/sau** (JSON dễ đọc). Chỉ đọc.

### 6.17 Cài đặt `/admin/cai-dat`
Nhóm: Chung (tên rạp, logo, hotline, email), Đặt vé (thông số mục 10 nghiệp vụ), Thông báo (mẫu email, SMTP), Thanh toán, Bảo mật (chính sách mật khẩu, thời hạn phiên), Giao diện (màu thương hiệu, banner).

---

## 7. MẪU & TRẠNG THÁI GIAO DIỆN CHUNG

| Tình huống | Cách hiển thị |
| --- | --- |
| Đang tải danh sách | Skeleton giống bố cục thật (không dùng spinner trống) |
| Đang gửi form | Nút `loading` + khóa form, tránh gửi đúp |
| Rỗng | `EmptyState` có minh họa + câu hướng dẫn + CTA |
| Lỗi mạng/server | `ErrorState` + nút Thử lại + mã yêu cầu (request id) |
| 401 | Làm mới token ngầm; thất bại → chuyển đăng nhập kèm `returnTo` |
| 403 | Trang "Bạn không có quyền truy cập" |
| Mất kết nối | Banner cố định phía trên, tự ẩn khi có mạng |
| Thành công | Toast xanh 3–4s (có thể kèm hoàn tác) |
| Xung đột (409) | Thông báo rõ nguyên nhân + hành động đề xuất (chọn lại ghế, tải lại) |
| Dữ liệu cũ (stale) | Tự làm mới khi tab được focus lại |

**Văn phong**: tiếng Việt, xưng "bạn", ngắn gọn, nói rõ hành động tiếp theo. Ví dụ lỗi: "Ghế A5 vừa có người khác chọn. Vui lòng chọn ghế khác." (không dùng "Đã xảy ra lỗi" chung chung). Định dạng: ngày `dd/MM/yyyy`, giờ `HH:mm` (24h), tiền `85.000đ`, thứ `T2…CN`.

---

## 8. RESPONSIVE – QUY TẮC CỤ THỂ

- Vùng bấm tối thiểu **44×44px**; khoảng cách giữa ghế ≥ 4px trên mobile (ghế ≥ 28px, cho phép zoom).
- Bảng admin: dưới 768px chuyển sang thẻ, hành động trong menu "⋯".
- Modal → bottom sheet trên mobile; form dài → các bước/accordion.
- Hero giảm chiều cao (aspect 4:5 hoặc 16:9) trên mobile; Quick Booking xếp dọc.
- Ẩn thao tác hover-only trên thiết bị cảm ứng (`@media (hover: hover)`).
- In vé: stylesheet `@media print` chỉ in vé (ẩn nav/footer), khổ A5/80mm.
- Tránh `100vh` lỗi trên mobile: dùng `100dvh`.

---

## 9. TRUY CẬP (A11Y) & HIỆU NĂNG GIAO DIỆN

- Cấu trúc tiêu đề h1→h3 đúng; landmark (`header/nav/main/footer`); skip link "Bỏ qua tới nội dung".
- Modal: bẫy focus, đóng bằng `Esc`, trả focus về nút gọi; `aria-modal`.
- Biểu mẫu: `label` gắn với input, thông báo lỗi `aria-describedby`, `aria-live="polite"` cho toast và kết quả check-in.
- Sơ đồ ghế: `role="grid"`/`gridcell` hoặc nút có `aria-pressed`; trạng thái không chỉ bằng màu (icon/hoa văn).
- Tương phản ≥ 4.5:1 cho chữ thường, 3:1 cho thành phần UI.
- Hiệu năng: code-splitting theo route (Admin/Staff tách bundle), ảnh lazy/WebP, font subset tiếng Việt, tránh re-render sơ đồ ghế (memo từng ghế), mục tiêu Lighthouse mobile: Performance ≥ 80, A11y ≥ 90. LCP < 2.5s trên trang chủ.

---

## 10. DANH SÁCH BÀN GIAO (DELIVERABLES) CỦA AI GIAO DIỆN

1. Cấu hình Tailwind + token màu/chữ/khoảng cách; file `index.css` (biến CSS, font, reset, print).
2. Bộ component `components/ui` đầy đủ mục 2.5 kèm trang xem mẫu `/_ui` (Storybook hoặc trang nội bộ chỉ chạy ở dev).
3. Layout: `PublicLayout`, `BookingLayout` (stepper + hold countdown), `AccountLayout`, `StaffLayout`, `AdminLayout`.
4. Toàn bộ trang mục 4–6 với **dữ liệu giả (mock)** trước khi nối API thật (giao diện chạy được độc lập).
5. Xử lý đủ 4 trạng thái cho mỗi màn hình, responsive 360/768/1024/1440.
6. Kiểm tra a11y cơ bản (axe) và Lighthouse; ảnh chụp màn hình các màn chính.

## 11. CHECKLIST NGHIỆM THU GIAO DIỆN
- [ ] Mọi màn hình dùng đúng token, không hard-code màu rải rác.
- [ ] Luồng đặt vé hoàn tất được **chỉ bằng điện thoại 360px**.
- [ ] Sơ đồ ghế rõ ràng, phân biệt được 7 trạng thái (kể cả với người mù màu).
- [ ] Đồng hồ giữ ghế xuất hiện ở mọi bước sau khi giữ ghế và hành xử đúng khi hết giờ.
- [ ] Trang thanh toán đủ: QR, thông tin sao chép được, upload biên lai, trạng thái chờ/từ chối/hết hạn.
- [ ] Vé điện tử đẹp, QR quét được ở độ sáng thấp, in được.
- [ ] Màn hình quét vé phân biệt tức thì 5 loại kết quả bằng màu + chữ + âm thanh.
- [ ] Admin: mọi danh sách có lọc/sắp xếp/phân trang; form có validate lỗi từ server; thao tác nguy hiểm có xác nhận.
- [ ] Hàng đợi duyệt biên lai thao tác được bằng bàn phím.
- [ ] Không có chữ tiếng Anh lẫn trong UI (trừ thuật ngữ thương hiệu), định dạng ngày/tiền đúng.
