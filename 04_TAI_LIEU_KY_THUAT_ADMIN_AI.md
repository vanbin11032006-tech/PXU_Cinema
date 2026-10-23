# TÀI LIỆU KỸ THUẬT PHÍA ADMIN (REACT) DÀNH CHO AI – PXU CINEMA

> Phạm vi: khu vực **quản trị `/admin`** (vai trò `admin`, `manager`) và các hành động quản lý liên quan đến `staff`.
> Dùng chung hạ tầng với `03_TAI_LIEU_KY_THUAT_FRONTEND_AI.md` (axios, auth, TanStack Query, form, test). File này chỉ nêu **phần riêng của Admin**.
> Nghiệp vụ gốc: `01_TAI_LIEU_NGHIEP_VU.md`. Giao diện: `02_TAI_LIEU_GIAO_DIEN_AI.md` (mục 6).
> **Quy tắc vàng**: ẩn/ khóa giao diện theo quyền chỉ để tiện dùng – **backend là nơi quyết định quyền**; mọi endpoint admin phải tự kiểm tra quyền và ghi audit.

---

## 1. CÔNG NGHỆ BỔ SUNG CHO ADMIN

| Nhu cầu | Thư viện | Ghi chú |
| --- | --- | --- |
| Bảng dữ liệu | `@tanstack/react-table` v8 | Sort/lọc phía server, chọn hàng, ẩn cột |
| Biểu đồ | `recharts` (hoặc `echarts-for-react`) | Import động theo trang |
| Kéo thả | `@dnd-kit/core` + `sortable` | Sắp xếp banner, kéo trong timeline/sơ đồ ghế |
| Timeline lịch chiếu | Tự dựng bằng CSS grid/SVG (hoặc `react-big-calendar` chế độ resource) | Có chống trùng giờ |
| Rich-text (tin tức/trang) | `tiptap` | Xuất HTML sạch + **DOMPurify** |
| Crop ảnh | `react-easy-crop` | Poster 2:3, banner 16:9 |
| Xem ảnh biên lai | Tự dựng (zoom/pan/rotate) hoặc `react-zoom-pan-pinch` | |
| Xuất file | Backend tạo file (khuyến nghị); FE chỉ tải `blob`. Dự phòng: `xlsx`/`papaparse` cho CSV nhỏ | |
| Diff JSON (audit) | `react-diff-viewer-continued` hoặc tự dựng | |
| Ngày | `react-day-picker` | Khoảng ngày |
| Phím tắt | `react-hotkeys-hook` | Duyệt biên lai |
| Bảng điều khiển lệnh | `cmdk` (Ctrl+K) | MAY |

Bundle admin **tách riêng** (`lazy(() => import('@/features/admin/AdminApp'))`) để không làm nặng web khách.

---

## 2. CẤU TRÚC THƯ MỤC ADMIN

```
src/features/admin/
├─ AdminApp.tsx                 # route con + layout
├─ layout/ AdminLayout, Sidebar, Topbar, CinemaSwitcher, NotificationBell, CommandPalette
├─ lib/
│  ├─ permissions.ts            # mã quyền + helper can()
│  ├─ menu.ts                   # cấu hình menu → quyền
│  ├─ columns/                  # định nghĩa cột bảng dùng lại
│  └─ exportFile.ts             # tải blob, đặt tên file
├─ components/
│  ├─ DataTable/                # bảng chung (server-side)
│  ├─ FilterBar/ · DateRangePicker · StatCard · ChartCard
│  ├─ FormSection · ImageUploader(crop) · RichTextEditor
│  ├─ ConfirmDialog (nhập lý do/từ khóa) · RequirePermission · AuditTimeline
│  └─ ReceiptViewer · SeatBuilder · ShowtimeTimeline · PriceMatrix
├─ modules/
│  ├─ dashboard/ movies/ showtimes/ cinemas/ rooms/ pricing/ combos/
│  ├─ orders/ receipts/ refunds/ exceptions/ vouchers/ members/ users/
│  ├─ payment-settings/ content/ reviews/ reports/ audit/ settings/
│  └─ (mỗi module: api.ts, hooks.ts, schemas.ts, pages/, components/)
└─ index.ts
```

---

## 3. PHÂN QUYỀN (RBAC) PHÍA FRONTEND

### 3.1 Mã quyền (`permissions.ts`)
Định dạng `resource.action`. Danh sách chuẩn (đồng bộ với backend):
```
dashboard.view
movie.read · movie.write · movie.delete
cinema.read · cinema.write
room.read · room.write            # gồm sơ đồ ghế
showtime.read · showtime.write · showtime.cancel
pricing.read · pricing.write
combo.read · combo.write · inventory.write
booking.read · booking.cancel · booking.change · booking.resend
payment.review                    # duyệt/từ chối biên lai
payment.exception.manage
refund.request · refund.approve · refund.process
voucher.read · voucher.write
member.read · member.adjust_points
user.read · user.write · user.lock · user.set_role
paymentsetting.read · paymentsetting.write
content.write · review.moderate
report.view · report.export
audit.view
setting.read · setting.write
checkin.override · checkin.undo
shift.manage
```
- Backend trả `user.permissions: string[]` (đã gộp theo vai trò + ghi đè). `can('x')` kiểm tra thuộc mảng; hỗ trợ wildcard `movie.*`.
- **Phạm vi rạp**: `user.cinema_ids[]` (rỗng + `admin` = tất cả). Dữ liệu danh sách/báo cáo tự lọc theo rạp được phép; `CinemaSwitcher` chỉ hiển thị rạp được gán.

### 3.2 Công cụ giao diện
- `<RequirePermission perm="refund.approve" fallback={null|<Disabled/>}>` bọc nút/khối.
- `menu.ts`: mỗi mục có `perm`; sidebar lọc theo quyền; route con dùng `ProtectedRoute permissions=[...]` → 403.
- Nút bị thiếu quyền: ẩn nếu hành động hiếm; `disabled` + tooltip "Bạn không có quyền" nếu cần giải thích.
- Thao tác nhạy cảm cần **xác nhận lại mật khẩu** (`POST /admin/auth/reauth` trả `reauth_token` sống 5 phút gửi trong header `X-Reauth-Token`): đổi cấu hình thanh toán, đổi vai trò, hoàn tiền > ngưỡng, hoàn tác check-in, xóa dữ liệu.

### 3.3 Ma trận mặc định vai trò → quyền
| Vai trò | Quyền |
| --- | --- |
| `admin` | `*` |
| `manager` | tất cả trừ `user.set_role`, `paymentsetting.write`, `setting.write`, `movie.delete`; giới hạn theo `cinema_ids` |
| `staff` | `booking.read`(phạm vi ca/ngày), `shift.manage`, check-in/POS (endpoint `staff/*`) |
Quyền tùy chỉnh theo người dùng (MAY): bảng `user_permissions` ghi đè.

---

## 4. HẠ TẦNG DÙNG CHUNG TRONG ADMIN

### 4.1 `DataTable` (server-driven)
- Props: `columns`, `queryKey`, `fetcher(params)`, `filters`, `rowActions`, `bulkActions`, `defaultSort`, `enableSelection`.
- **Trạng thái bảng nằm trên URL** (`?page=&per_page=&sort=&q=&status=&from=&to=`) để chia sẻ/ quay lại; dùng `useSearchParams`.
- Tìm kiếm debounce 400ms; `keepPreviousData` khi đổi trang; skeleton hàng; empty/error state.
- Hành động hàng loạt: chọn nhiều → thanh hành động dính; xác nhận; hiển thị kết quả từng dòng (thành công/ thất bại) khi API trả `results[]`.
- Mobile: chuyển thẻ (render prop `mobileCard`).
- Xuất: nút **Xuất** gọi `GET …/export?<cùng bộ lọc>&format=xlsx` (backend stream), FE tải blob; hiện tiến trình.

### 4.2 Form quản trị
- Mọi form: `react-hook-form` + `zod`; lỗi `VALIDATION_ERROR` map vào trường.
- **Optimistic concurrency**: các thực thể sửa được trả `updated_at`/`version`; gửi lại qua `If-Match`/`version`; backend trả `409 CONFLICT_VERSION` → hiển thị "Dữ liệu đã bị người khác thay đổi" + nút **Tải lại** / **Xem khác biệt**.
- Cảnh báo thoát khi `formState.isDirty` (`useBlocker`).
- `ImageUploader`: chọn ảnh → crop theo tỉ lệ → nén → `POST /admin/uploads` (`multipart`, `kind=poster|banner|combo|qr|avatar`) → trả `{ url, width, height }`; hiển thị tiến trình; xóa ảnh cũ do backend dọn.

### 4.3 ConfirmDialog nâng cao
`<ConfirmDialog title description requireReason|requireText="XOA" danger onConfirm(reason)>`; lý do tối thiểu 10 ký tự được gửi kèm API (`reason`) và lưu audit.

### 4.4 Thông báo thời gian thực cho Admin
- SSE `GET /admin/stream` (sự kiện: `receipt.submitted`, `receipt.sla_warning`, `exception.created`, `stock.low`, `job.failed`, `refund.requested`).
- `NotificationBell`: số đếm + danh sách; âm báo tùy chọn (lưu cài đặt cục bộ); `document.title` hiện `(3) Admin` khi có biên lai chờ.
- Fallback: polling `GET /admin/summary/badges` mỗi 30 giây.

### 4.5 Xử lý lỗi/ quyền
- 403 → trang "Không có quyền"; 409 → thông điệp theo mã; 422 → lỗi trường; 5xx → `ErrorState` + `requestId`.
- Mọi mutation hiển thị toast kết quả; hành động quan trọng có thể "Hoàn tác" trong 5 giây nếu backend hỗ trợ (MAY).

---

## 5. HỢP ĐỒNG API ADMIN

**Tiền tố**: `/api/v1/admin` (trừ `staff/*`). Quy ước phân trang/lỗi như file 03 mục 6. Mọi `POST/PUT/PATCH/DELETE` ghi **audit**. Thao tác nhạy cảm cần `X-Reauth-Token`.

### 5.1 Dashboard & báo cáo
| Endpoint | Ghi chú |
| --- | --- |
| `GET /dashboard/summary?cinema_id=&from=&to=` | KPI: `revenue_today`, `revenue_prev_day`, `revenue_month`, `tickets_today`, `occupancy_rate`, `pending_receipts`, `expired_today`, `avg_order_value` |
| `GET /dashboard/revenue?granularity=day|week|month&from=&to=&cinema_id=` | Chuỗi doanh thu |
| `GET /dashboard/top-movies?from=&to=&limit=5` | |
| `GET /dashboard/channels` | Online vs POS, theo phương thức thanh toán |
| `GET /dashboard/upcoming-showtimes` | Suất sắp chiếu + lấp đầy |
| `GET /dashboard/alerts` | Combo thấp, giao dịch bất thường, job lỗi |
| `GET /reports/{type}?…` | `type` ∈ `revenue, movies, showtimes, staff-shifts, vouchers, refunds, customers, concessions, checkin` |
| `GET /reports/{type}/export?format=xlsx|csv|pdf&…` | Stream file |

### 5.2 Phim, rạp, phòng, ghế
| Endpoint | |
| --- | --- |
| `GET/POST /movies` · `GET/PUT/DELETE /movies/:id` · `PATCH /movies/:id/status` | `DELETE` → `409 MOVIE_HAS_SHOWTIMES` nếu có suất/đơn (gợi ý `PATCH status=ended`) |
| `GET/POST /cinemas` · `PUT /cinemas/:id` | |
| `GET/POST /rooms` · `GET/PUT /rooms/:id` · `PATCH /rooms/:id/status` | |
| `GET /rooms/:id/seats` · `PUT /rooms/:id/seats` | **Lưu cả sơ đồ** `{ rows, cols, cells:[{row,col,x,y,type,label}] }`. Backend kiểm tra ghế đã bán: `409 SEAT_LAYOUT_CONFLICT` kèm `details.affected_showtimes` |
| `POST /rooms/:id/seats/generate` | Sinh nhanh theo mẫu `{ rows, cols, vip_rows:[…], couple_rows:[…] }` |

### 5.3 Suất chiếu
| Endpoint | |
| --- | --- |
| `GET /showtimes?cinema_id=&room_id=&date=&movie_id=&status=` | Danh sách / dữ liệu timeline |
| `POST /showtimes` | Tạo; backend tự tính `end_time`; lỗi `SHOWTIME_OVERLAP` (`details.conflicts[]`) |
| `POST /showtimes/validate` | **Kiểm tra trước** (không lưu) – dùng cho cảnh báo thời gian thực |
| `POST /showtimes/bulk` | Tạo hàng loạt; cờ `dry_run=true` trả bản xem trước + xung đột |
| `PUT /showtimes/:id` | Chỉ giá/ghi chú nếu đã có đơn; ngược lại sửa đầy đủ |
| `GET /showtimes/:id/impact` | Danh sách đơn bị ảnh hưởng nếu hủy/dời |
| `POST /showtimes/:id/cancel` | `{ reason, action:'refund'|'reschedule', target_showtime_id? }` |
| `POST /showtimes/:id/block-seats` · `DELETE …/block-seats/:seatId` | Chặn ghế theo suất |

### 5.4 Giá & combo
| Endpoint | |
| --- | --- |
| `GET/PUT /pricing/rules` (bulk replace có version) · `POST /pricing/simulate` | Mô phỏng giá |
| `GET/POST /combos` · `PUT/DELETE /combos/:id` · `POST /combos/:id/stock` | `stock` điều chỉnh có `delta` + lý do |
| `GET /inventory/movements` | Lịch sử kho |

### 5.5 Đơn hàng, thanh toán, hoàn tiền
| Endpoint | |
| --- | --- |
| `GET /orders?status=&payment_status=&channel=&cinema_id=&from=&to=&q=&voucher=` | |
| `GET /orders/:id` | Chi tiết đầy đủ + `timeline[]` |
| `POST /orders/:id/approve` | Duyệt thanh toán (`payment.review`). Idempotent: duyệt lại trả đơn hiện tại |
| `POST /orders/:id/reject` | `{ reason_code, note }` |
| `POST /orders/:id/cancel` | `{ reason, refund:true|false }` |
| `POST /orders/:id/seats/change` · `POST /orders/:id/reschedule` | Đổi ghế/đổi suất |
| `POST /orders/:id/resend` | Gửi lại vé |
| `POST /orders/:id/notes` | Ghi chú nội bộ |
| `GET /receipts/queue?status=proof_submitted&sort=submitted_at` | Hàng đợi duyệt (kèm `sla_deadline_at`, `duplicate_of[]`) |
| `GET /receipts/:paymentId/image` | Ảnh có xác thực (URL ký có hạn) |
| `GET /refunds` · `GET /refunds/:id` · `POST /refunds` | Tạo yêu cầu: `{ booking_id, scope:'full'|'partial', seat_ids?, combo_ids?, amount, reason }` |
| `POST /refunds/:id/approve` · `/reject` · `/complete` | `complete` nhận `{ reference, completed_at }` |
| `GET /payment-exceptions` · `POST /payment-exceptions/:id/resolve` | `{ action:'link'|'restore'|'refund'|'ignore', booking_id?, note }` |

### 5.6 Check-in (quản lý)
`POST /staff/checkin` (staff) · `POST /admin/checkins/:ticketId/undo` (`checkin.undo`, lý do + reauth) · `GET /admin/checkin-logs?showtime_id=&result=`.

### 5.7 Voucher, thành viên, người dùng
| Endpoint | |
| --- | --- |
| `GET/POST /vouchers` · `PUT/DELETE /vouchers/:id` · `POST /vouchers/bulk-generate` · `GET /vouchers/:id/usages` | |
| `GET /members` · `POST /members/:userId/points/adjust` | `{ delta, reason }` |
| `GET /membership-tiers` · `PUT /membership-tiers` | |
| `GET /users?role=&status=&q=` · `GET /users/:id` | |
| `PATCH /users/:id/role` | `user.set_role` + reauth. Chặn: tự hạ quyền mình, xóa admin cuối: `409 LAST_ADMIN` |
| `POST /users/:id/lock` · `/unlock` | `{ reason }` |
| `POST /users/:id/reset-password` | Gửi email đặt lại |
| `POST /users` | Tạo tài khoản staff/manager (gửi lời mời đặt mật khẩu) |

### 5.8 Cấu hình
| Endpoint | |
| --- | --- |
| `GET /payment-settings` · `PUT /payment-settings` | `multipart` khi có ảnh QR. Cần reauth. Kiểm tra: MIME, ≤ 2MB, chống SVG chứa script. Trả bản xem trước |
| `POST /payment-settings/test-qr` | Sinh thử QR (không lưu) |
| `GET/PUT /settings` | Theo nhóm (mục 10 file 01), có `version` |
| `GET /notification-templates` · `PUT /notification-templates/:key` | Mẫu email |
| `GET/POST/PUT/DELETE /banners` · `PUT /banners/order` | Sắp xếp |
| `GET/POST/PUT/DELETE /news` · `/pages` | Nội dung (HTML được sanitize **ở backend**) |
| `GET /reviews` · `PATCH /reviews/:id/visibility` | |
| `GET /audit-logs?actor=&action=&entity=&from=&to=` | Chỉ đọc |
| `POST /uploads` | Tải ảnh chung |
| `GET /admin/summary/badges` | Số đếm cho chuông |

---

## 6. ĐẶC TẢ KỸ THUẬT CÁC MODULE TRỌNG TÂM

### 6.1 Duyệt biên lai (`modules/receipts`) – ưu tiên cao nhất
**Mục tiêu**: duyệt nhanh, chính xác, ít thao tác.
- Dữ liệu: `useInfiniteQuery(qk.receipts.queue)` (poll 15s + SSE `receipt.submitted` → `invalidate`).
- Bố cục hai khung: danh sách (trái) – chi tiết (phải). Chọn đơn bằng ↑/↓.
- **Khối đối chiếu** hiển thị cạnh ảnh: `amount_required` vs `paid_amount` (khách nhập) – tô đỏ nếu lệch; nội dung CK mong đợi; thời gian gửi; thời gian còn đến SLA (đếm ngược, màu cảnh báo < 10 phút); `duplicate_of` (nếu ảnh trùng, link tới đơn kia).
- Hành động:
  - **Duyệt** (`Enter`): `POST /orders/:id/approve` → toast, tự chuyển đơn kế tiếp. Nếu số tiền lệch → hộp thoại xác nhận phải chọn: *Duyệt vì đủ tiền / Duyệt & ghi nhận thừa tiền / Hủy duyệt*.
  - **Từ chối** (`R`): mở modal chọn `reason_code` (bắt buộc) + ghi chú; gửi `POST …/reject`.
  - **Bỏ qua** (`S`): đưa xuống cuối hàng (cục bộ).
- **Khóa bản ghi khi xem (SHOULD)**: `POST /receipts/:id/lock` (TTL 2 phút) để hai người duyệt không trùng; hiển thị "Đang được [tên] xử lý". Nếu không có khóa, backend phải idempotent (duyệt lần 2 trả đơn đã duyệt và FE hiển thị "đã được duyệt bởi …").
- `ReceiptViewer`: zoom (lăn chuột/pinch), kéo, xoay 90°, tải ảnh, chế độ toàn màn hình; không tải ảnh gốc công khai (URL ký hạn 5 phút, tự làm mới khi hết hạn).
- Hiệu năng: prefetch ảnh đơn kế tiếp.

### 6.2 Suất chiếu (`modules/showtimes`)
- `ShowtimeTimeline`: trục giờ 06:00–02:00 (chia 15 phút), hàng = phòng. Mỗi suất là khối `left = (start - dayStart)/range`, `width = duration/range`; màu theo phim; biểu tượng khóa nếu đã có đơn.
- Tạo bằng **kéo trên hàng trống** hoặc nút Thêm → form. Khi thay đổi `movie/room/start`: gọi `POST /showtimes/validate` (debounce 300ms) → hiển thị `end_time` tính bởi server và **xung đột** (đỏ, liệt kê suất trùng + khoảng dọn phòng). **Chặn lưu khi còn xung đột**.
- Kéo-thả để dời suất **chỉ cho suất chưa có đơn**; nếu đã có đơn → mở luồng "Dời/hủy suất" (`/impact`).
- **Tạo hàng loạt**: wizard 3 bước (chọn → xem trước `dry_run` → xác nhận); bảng xem trước có cột trạng thái (OK/xung đột) và cho bỏ dòng.
- Múi giờ: hiển thị/ nhập theo giờ rạp; gửi lên `show_date` + `start_time` địa phương (không chuyển UTC ở FE).

### 6.3 Trình dựng sơ đồ ghế (`SeatBuilder`)
- Mô hình: mảng ô `{ row, col, x, y, type: 'standard'|'vip'|'couple'|'aisle'|'blank'|'disabled', label }`.
- Công cụ: *Chọn*, *Tô* (kéo nhiều ô), *Xóa*, *Đổi nhãn hàng*, *Sinh nhanh*; Undo/Redo (stack lệnh, `Ctrl+Z/Y`).
- Ghế đôi: tô 2 ô liền kề cùng hàng (kiểm tra hợp lệ, tự gộp cặp, ngăn xếp lẻ).
- Kiểm tra trước khi lưu: không trùng `row+col`, nhãn ghế duy nhất, không có ghế đôi lẻ; hiển thị thống kê theo loại + tổng sức chứa.
- Lưu bằng `PUT /rooms/:id/seats`; lỗi `SEAT_LAYOUT_CONFLICT` → modal liệt kê suất/ghế bị ảnh hưởng.
- Xem trước dùng đúng component `SeatMap` của khách (chế độ `readOnly`).
- Render: SVG; ô `memo`; kéo chọn dùng pointer events + `elementFromPoint`.

### 6.4 Đơn hàng (`modules/orders`)
- Danh sách + chi tiết (route `/admin/don-hang/:id` mở **drawer**, deep-link được).
- Tab chi tiết: *Tổng quan · Vé & ghế · Thanh toán · Timeline · Ghi chú*.
- **Nút hành động hiển thị theo ma trận trạng thái** (hàm thuần `orderActions(order, user)` có test):
| Trạng thái | Hành động khả dụng |
| --- | --- |
| `holding` | Hủy |
| `holding` + payment `proof_submitted` | Duyệt, Từ chối, Hủy (đơn giữ ghế theo SLA, chỉ chuyển `confirmed` khi duyệt) |
| `confirmed` (payment `paid`, chưa chiếu) | Gửi lại vé, Đổi ghế, Đổi suất, Hoàn tiền, Hủy |
| `completed` | Gửi lại vé, Hoàn (ngoại lệ, manager) |
| `expired` | Khôi phục (nếu ghế còn & có giao dịch khớp) |
| `cancelled` | – |
- Hoàn tiền: wizard chọn phạm vi (toàn phần / từng ghế / combo / số tiền) → lý do → xác nhận (reauth nếu > ngưỡng) → tạo `refund` ở trạng thái `requested`/`approved`.

### 6.5 Hoàn tiền & giao dịch bất thường
- Bảng `refunds` với tab theo trạng thái; trang chi tiết có **checklist**: duyệt → chuyển tiền ngoài hệ thống → nhập mã tham chiếu + ngày → `complete`.
- Hiển thị tác động: vé sẽ hủy, ghế được nhả, điểm/voucher được hoàn.
- `payment-exceptions`: bảng + modal xử lý; chọn đơn để liên kết bằng ô tìm kiếm đơn (autocomplete theo mã/SĐT).

### 6.6 Cấu hình thanh toán QR (`modules/payment-settings`)
- Form: ngân hàng (select có logo + `bank_code` chuẩn VietQR), số tài khoản (chỉ số, 6–19 ký tự), tên chủ TK (in hoa không dấu – tự chuẩn hóa), tiền tố nội dung (2–6 ký tự A-Z0-9), upload ảnh QR tùy chỉnh, công tắc `use_custom_qr` (chỉ bật được khi đã có ảnh), `confirm_mode` (manual/auto), `webhook_secret` (chỉ hiện một lần khi tạo; sau đó che).
- **Live preview**: dùng chính `PaymentQRPanel` của khách với dữ liệu mẫu (số tiền 100.000đ, nội dung `PXU DEMO01`); khi ở chế độ VietQR, dựng URL ảnh VietQR ngay trên FE để xem.
- Lưu → modal **xác nhận mật khẩu** → `PUT /payment-settings` → toast + ghi audit. Hiển thị cảnh báo đỏ nếu đang dùng thông tin ví dụ mặc định ("Tài khoản mặc định chỉ là ví dụ").
- Bảo vệ: chỉ nhận PNG/JPG/WebP ≤ 2MB; FE kiểm tra kích thước tối thiểu 300×300.

### 6.7 Người dùng & phân quyền
- Chặn trên UI (và backend): không tự đổi vai trò/khóa chính mình; không hạ `admin` cuối cùng; đổi vai trò cần lý do + reauth.
- Chi tiết người dùng: thông tin, lịch sử đơn, lịch sử đăng nhập/thiết bị, hành động quản trị đã thực hiện (nếu là staff).
- Tạo staff: gửi **lời mời** (liên kết đặt mật khẩu), gán `cinema_ids`; không hiển thị mật khẩu tạm.

### 6.8 Báo cáo
- Mỗi báo cáo là cấu hình khai báo `{ filters, columns, charts }` + trang chung `ReportPage` (giảm lặp).
- Biểu đồ nhận dữ liệu đã tổng hợp từ server; FE không tự tổng hợp từ danh sách lớn.
- Xuất file chạy bất đồng bộ nếu lớn: `POST /reports/export-jobs` → poll `GET /export-jobs/:id` → tải khi `ready`.
- Ghi chú định nghĩa chỉ số (tooltip): "Doanh thu tính theo thời điểm xác nhận thanh toán, đã trừ hoàn tiền" (đồng bộ mục 7 file 01).

### 6.9 Nhật ký (audit)
- Bảng chỉ đọc, lọc theo người/hành động/đối tượng/ngày; mở dòng xem **diff** `before/after` (ẩn trường nhạy cảm: mật khẩu, secret, số tài khoản che giữa).
- Liên kết ngược tới đối tượng (đơn, user, suất…).

---

## 7. QUY ƯỚC KỸ THUẬT RIÊNG CỦA ADMIN

1. **Không tự tính dữ liệu tài chính ở FE** (tổng, hoàn, doanh thu) – hiển thị số server trả; FE chỉ định dạng.
2. **Hành động phá hủy/nhạy cảm** → `ConfirmDialog` + lý do + (nếu cần) reauth; gửi `reason` trong body.
3. **Idempotency**: duyệt, hoàn tiền, hủy gửi `Idempotency-Key`; bấm lặp không gây lặp tác dụng.
4. **Cập nhật tức thì**: sau mutation `invalidateQueries` theo nhóm (`orders`, `receipts`, `dashboard`).
5. **Phân trang server-side bắt buộc** cho bảng có thể > 100 dòng.
6. **Dữ liệu nhạy cảm**: che email/SĐT trong danh sách (`a***@gmail.com`, `09*****123`); bấm "Hiện" mới gọi API chi tiết (ghi audit `pii.view`) – SHOULD.
7. **Ảnh/biên lai** không đưa vào `localStorage`/cache bền vững; URL ký hạn ngắn.
8. **Phiên admin**: hết hạn không hoạt động 30 phút → khóa màn hình, yêu cầu đăng nhập lại (đếm theo tương tác); nhiều tab đồng bộ.
9. **Múi giờ & định dạng** thống nhất `dd/MM/yyyy HH:mm` giờ rạp.
10. **Khả năng mở rộng**: thêm module mới chỉ cần (a) thêm quyền, (b) thêm mục `menu.ts`, (c) thêm route lazy, (d) dùng `DataTable`/`FormSection`.

---

## 8. TEST ADMIN

| Cấp | Nội dung bắt buộc |
| --- | --- |
| Unit | `can()` + wildcard, `orderActions`, tính vị trí/độ rộng timeline, kiểm tra hợp lệ `SeatBuilder` (ghế đôi, trùng ô), chuẩn hóa tên chủ TK, che PII |
| Component | `DataTable` (đồng bộ URL, sort, chọn hàng), `ReceiptViewer`, form suất chiếu (hiển thị xung đột), `ConfirmDialog` (bắt buộc lý do) |
| Tích hợp (MSW) | Duyệt/từ chối biên lai, tạo suất xung đột, hoàn tiền một phần, `409 CONFLICT_VERSION`, `LAST_ADMIN` |
| E2E (Playwright) | (1) khách gửi biên lai → admin duyệt bằng phím tắt → khách thấy vé; (2) từ chối kèm lý do → khách gửi lại; (3) tạo suất trùng bị chặn; (4) hủy suất → danh sách đơn ảnh hưởng → hoàn tiền; (5) staff không vào được `/admin`; (6) manager không đổi được vai trò |
| Bảo mật | Truy cập trực tiếp URL/endpoint admin bằng tài khoản không đủ quyền → 403 (kiểm thử API) |

---

## 9. LỘ TRÌNH TRIỂN KHAI ADMIN (THỨ TỰ LÀM)

1. **Khung**: `AdminLayout`, menu theo quyền, `RequirePermission`, `DataTable`, `FormSection`, `ConfirmDialog`, `ImageUploader`.
2. **Danh mục**: Phim → Rạp/Phòng + `SeatBuilder` → Combo.
3. **Suất chiếu**: danh sách → timeline + validate → hàng loạt → hủy/dời + impact.
4. **Đơn & thanh toán**: danh sách/chi tiết đơn → **Duyệt biên lai** → cấu hình QR → hoàn tiền → giao dịch bất thường.
5. **Marketing & người dùng**: voucher, thành viên, người dùng/phân quyền, banner/tin tức.
6. **Báo cáo & dashboard**, xuất file.
7. **Nhật ký, cài đặt, thông báo realtime, phiên admin**.
8. Test E2E, tối ưu, tài liệu.

## 10. TIÊU CHÍ HOÀN THÀNH (ADMIN)
- [ ] Mọi menu/route/nút tuân theo mã quyền; truy cập trái phép trả 403 cả UI lẫn API.
- [ ] Không thể tạo suất trùng giờ/thiếu thời gian dọn phòng (kiểm thử cả FE và BE).
- [ ] Duyệt/từ chối biên lai hoạt động bằng bàn phím, có lý do từ chối, có cảnh báo lệch tiền & ảnh trùng, chịu được duyệt lặp.
- [ ] Hoàn tiền toàn phần/một phần cập nhật đúng vé, ghế, điểm, voucher; có audit.
- [ ] Cấu hình QR: preview khớp giao diện khách; yêu cầu xác nhận mật khẩu; cảnh báo dữ liệu mẫu.
- [ ] Không xóa được phim/suất đã có đơn; hiển thị gợi ý thay thế.
- [ ] Mọi thao tác ghi đều xuất hiện trong nhật ký với diff.
- [ ] Bảng có phân trang server, trạng thái lưu trên URL, xuất file hoạt động.
- [ ] Chỉ số dashboard khớp báo cáo và khớp dữ liệu đơn `paid` trừ hoàn tiền.
- [ ] Bundle admin tách riêng; Lighthouse Accessibility ≥ 90 trên các trang chính.
