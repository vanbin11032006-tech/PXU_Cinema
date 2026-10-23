# TÀI LIỆU KỸ THUẬT FRONTEND (REACT) DÀNH CHO AI – PXU CINEMA

> Phạm vi: ứng dụng **khách hàng + staff** (`/`, `/staff`). Phần quản trị (`/admin`) xem `04_TAI_LIEU_KY_THUAT_ADMIN_AI.md` (dùng chung hạ tầng ở file này).
> Đọc kèm: `01_TAI_LIEU_NGHIEP_VU.md` (quy tắc, trạng thái, mã lỗi) và `02_TAI_LIEU_GIAO_DIEN_AI.md` (giao diện, component).
> Quy ước: **MUST / SHOULD / MAY**. Mọi chuỗi hiển thị bằng tiếng Việt; mã, tên biến, commit bằng tiếng Anh.

---

## 1. CÔNG NGHỆ

| Nhóm | Lựa chọn | Ghi chú |
| --- | --- | --- |
| Build | **Vite 5+**, React **18/19**, **TypeScript** (MUST – dùng `strict`) | Nếu bắt buộc JS thuần, vẫn dùng JSDoc + `// @ts-check` |
| Router | `react-router-dom` v6 (data router: `createBrowserRouter`) | Lazy route, `errorElement`, loader nhẹ |
| Dữ liệu server | **TanStack Query v5** | Cache, retry, invalidate, optimistic |
| State client | **Zustand** (booking draft, UI) + React Context cho Auth | Tránh nhét state server vào Zustand |
| HTTP | `axios` (instance + interceptor) | |
| Form | `react-hook-form` + `zod` (`@hookform/resolvers`) | Schema dùng chung validate |
| Style | Tailwind CSS 3.4+ (`@tailwindcss/forms`), `clsx`, `tailwind-merge`, `class-variance-authority` | |
| Icon | `lucide-react` | |
| Ngày giờ | `dayjs` (plugin `utc`, `timezone`, `customParseFormat`, `relativeTime`) | Múi giờ `Asia/Ho_Chi_Minh` |
| QR hiển thị | `qrcode.react` | |
| Quét QR | `html5-qrcode` (hoặc `@yudiel/react-qr-scanner`) | Chỉ staff, import động |
| Hiệu ứng | `canvas-confetti` (import động) | Tôn trọng reduced-motion |
| Realtime | **SSE** (`EventSource`) ưu tiên; dự phòng polling | Có thể nâng cấp WebSocket |
| In vé/PDF | `html-to-image` + `jspdf` (hoặc `react-to-print`) | Import động |
| i18n | `i18next` + `react-i18next` (MAY, mặc định `vi`) | |
| Test | **Vitest** + Testing Library + **MSW** (mock API) + **Playwright** (E2E) | |
| Chất lượng | ESLint (+ `jsx-a11y`), Prettier, Husky + lint-staged, `tsc --noEmit` | |
| Theo dõi lỗi | Sentry (MAY) | |

---

## 2. CẤU TRÚC THƯ MỤC (feature-based)

```
frontend/
├─ public/                      # favicon, manifest, robots.txt
├─ src/
│  ├─ app/
│  │  ├─ main.tsx               # entry
│  │  ├─ App.tsx                # Providers: QueryClient, Auth, Toaster, Router
│  │  ├─ router.tsx             # định nghĩa route (lazy)
│  │  └─ providers/             # QueryProvider, AuthProvider, ThemeProvider
│  ├─ components/
│  │  ├─ ui/                    # Button, Input, Modal, Table, Toast...  (file 02, mục 2.5)
│  │  ├─ layout/                # PublicLayout, BookingLayout, AccountLayout, StaffLayout
│  │  └─ common/                # Navbar, Footer, MovieCard, TrailerModal, QuickBooking,
│  │                            # SeatMap, SeatLegend, HoldCountdown, PriceText, StatusBadge,
│  │                            # AgeBadge, QRImage, FileUploader, ProtectedRoute, ErrorBoundary
│  ├─ features/
│  │  ├─ auth/                  # api, hooks, schemas, pages (Login, Register, Forgot, Reset, VerifyEmail)
│  │  ├─ movies/                # api, hooks, pages (Home, MovieList, MovieDetail), components
│  │  ├─ showtimes/             # api, hooks, ShowtimeGrid, DateStrip
│  │  ├─ booking/               # store, api, hooks, pages (Seats, Combos, Review), components
│  │  ├─ payment/               # api, hooks, PaymentQRPage, ProofUploader, PaymentStatus
│  │  ├─ tickets/               # api, hooks, TicketPage, TicketCard, ticketExport.ts
│  │  ├─ account/               # Profile, MyTickets, Points, Notifications
│  │  ├─ content/               # news, banners, static pages
│  │  ├─ staff/                 # ScannerPage, PosPage, TodayTicketsPage, ShiftPage
│  │  └─ admin/                 # (xem file 04)
│  ├─ services/
│  │  ├─ http.ts                # axios instance + interceptors
│  │  ├─ queryClient.ts
│  │  ├─ sse.ts                 # hook/wrapper EventSource có reconnect
│  │  └─ errors.ts              # ApiError, mapErrorToMessage
│  ├─ hooks/                    # useDebounce, useCountdown, useServerTime, useCopy, useOnline, useMediaQuery
│  ├─ lib/                      # format.ts (tiền/ngày), dayjs.ts, env.ts, storage.ts, seatMath.ts, constants.ts
│  ├─ types/                    # api.d.ts, domain.ts (Movie, Showtime, Seat, Booking…)
│  ├─ styles/index.css
│  └─ test/                     # setup, mocks (MSW handlers), factories
├─ e2e/                         # Playwright
├─ .env.example
├─ index.html
├─ vite.config.ts
├─ tailwind.config.ts
└─ tsconfig.json
```
**Quy tắc**: `features/*` không import chéo nội bộ của feature khác (chỉ qua `index.ts` công khai). `components/ui` không biết gì về nghiệp vụ.

Alias đường dẫn: `@/` → `src/`.

---

## 3. BIẾN MÔI TRƯỜNG
```
VITE_API_URL=http://localhost:5000/api/v1
VITE_APP_NAME=PXU Cinema
VITE_ASSET_URL=http://localhost:5000          # tiền tố ảnh upload
VITE_SSE_ENABLED=true
VITE_SENTRY_DSN=
VITE_DEFAULT_CINEMA_ID=1
```
Đọc qua `lib/env.ts` có kiểm tra bằng zod; thiếu biến bắt buộc → lỗi rõ ràng khi khởi động. **Không** đưa secret vào frontend.

---

## 4. HẠ TẦNG HTTP & XÁC THỰC

### 4.1 Axios instance (`services/http.ts`)
- `baseURL = VITE_API_URL`, `timeout = 15000`, header `Accept: application/json`, `X-Request-Id` (uuid mỗi request).
- **Request interceptor**: gắn `Authorization: Bearer <accessToken>` (từ bộ nhớ); gắn `Idempotency-Key` cho các `POST` tạo đơn/thanh toán (truyền qua `config.meta`).
- **Response interceptor**:
  - Chuẩn hóa lỗi → `ApiError { status, code, message, details, requestId }` theo định dạng ở mục 9 file 01.
  - **401 `UNAUTHENTICATED`**: thử `POST /auth/refresh` **một lần** (hàng đợi các request đang chờ, dùng 1 promise duy nhất), thành công → gửi lại; thất bại → `logout()` + chuyển `/dang-nhap?returnTo=…`.
  - `TOO_MANY_REQUESTS (429)`: đọc `Retry-After`, hiển thị toast.
  - Lỗi mạng: ném `ApiError { code: 'NETWORK' }`.
- Header `Date` hoặc trường `server_time` trong phản hồi dùng cập nhật **độ lệch giờ server** (xem 8.3).

### 4.2 Lưu token
- **Khuyến nghị (MUST ưu tiên)**: refresh token ở **cookie httpOnly + SameSite=Lax/Strict + Secure**, access token **chỉ giữ trong bộ nhớ** (biến module/Zustand, không lưu `localStorage`) → giảm rủi ro XSS. Khi tải lại trang, gọi `POST /auth/refresh` để lấy access token mới (`withCredentials: true`).
- Phương án dự phòng (nếu backend chưa hỗ trợ cookie): lưu refresh token trong `localStorage` và ghi chú rủi ro; vẫn xoay vòng token.
- Đa tab: đồng bộ đăng xuất qua `BroadcastChannel('pxu-auth')`.

### 4.3 AuthContext
```ts
type AuthState = { user: User | null; status: 'loading'|'authenticated'|'anonymous' };
login(email, password) · register(payload) · logout() · refreshProfile()
hasRole(...roles) · can(permission: string)
```
- Khởi động: `status='loading'` → thử refresh → `GET /auth/me`. Hiển thị splash cho đến khi xác định.
- `user` gồm: `id, name, email, phone, role, permissions[], cinema_ids[], points, tier, email_verified, status`.
- `ProtectedRoute({ roles?, permissions?, redirectTo? })`: `loading` → splash; `anonymous` → chuyển đăng nhập kèm `returnTo`; sai quyền → trang 403.
- **Lưu ý**: ẩn UI theo quyền chỉ là trải nghiệm; **quyền thật do backend kiểm**.

---

## 5. ĐỊNH TUYẾN

Dùng `createBrowserRouter`, mỗi nhóm trang `lazy()`:
```
/                         PublicLayout
  index                   HomePage
  phim, phim/:slug, lich-chieu, rap, bap-nuoc, khuyen-mai, trang/:slug
/dat-ve                   BookingLayout (ProtectedRoute: customer)
  :showtimeId/ghe         SeatsPage
  :bookingCode/bap-nuoc   CombosPage
  :bookingCode/xac-nhan   ReviewPage
/thanh-toan/:bookingCode  PaymentPage
/ve/:bookingCode          TicketPage
/tai-khoan/*              AccountLayout (profile, ve-cua-toi, diem-thuong, thong-bao)
/dang-nhap /dang-ky /quen-mat-khau /dat-lai-mat-khau/:token /xac-thuc-email/:token
/staff/*                  StaffLayout (ProtectedRoute roles staff|manager|admin)
/admin/*                  AdminLayout (xem file 04)
*                         NotFoundPage
```
- **Guard luồng đặt vé**: `BookingLayout` tải đơn theo `bookingCode` → nếu `status != holding` hoặc hết hạn → chuyển trang phù hợp (vé/ hết hạn). Không cho vào bước sau nếu bước trước chưa hoàn tất.
- `ScrollRestoration`, tiêu đề trang động (`useDocumentTitle`), `<meta>` cơ bản cho SEO/Open Graph ở trang phim (dùng `react-helmet-async`).
- Prefetch dữ liệu khi hover liên kết quan trọng (`queryClient.prefetchQuery`).
- **SEO**: SPA không SSR đủ cho nội dung công khai nhỏ. Nếu cần SEO mạnh: prerender trang phim (vite-plugin-ssr / Next.js ở giai đoạn sau) – ghi chú MAY.

---

## 6. HỢP ĐỒNG API (API CONTRACT)

**Base**: `/api/v1` · JSON · UTF-8 · ngày `YYYY-MM-DD`, giờ `HH:mm:ss`, thời điểm ISO-8601 UTC (`2026-10-23T02:15:00Z`) · tiền là **số nguyên VND**.
**Phản hồi thành công**: `{ "data": …, "meta": { "page": 1, "per_page": 20, "total": 134, "server_time": "…" } }`.
**Lỗi**: `{ "error": { "code", "message", "details" } }` + `X-Request-Id`.
**Phân trang**: `?page=1&per_page=20&sort=-created_at`. **Lọc**: tham số rõ ràng.

### 6.1 Xác thực
| Method & Path | Mục đích | Ghi chú |
| --- | --- | --- |
| `POST /auth/register` | Đăng ký | body: name, email, phone, password, accept_terms |
| `POST /auth/login` | Đăng nhập | trả `access_token`, set cookie refresh; `user` |
| `POST /auth/refresh` | Làm mới token | |
| `POST /auth/logout` | Đăng xuất (thu hồi refresh) | |
| `GET /auth/me` | Hồ sơ + permissions | |
| `PUT /auth/me` | Cập nhật hồ sơ | |
| `POST /auth/change-password` | Đổi mật khẩu | |
| `POST /auth/forgot-password` | Gửi email đặt lại | trả 202 trung tính |
| `POST /auth/reset-password` | Đặt lại | token + mật khẩu mới |
| `POST /auth/verify-email` / `POST /auth/resend-verification` | Xác thực email | |

### 6.2 Danh mục công khai
| Endpoint | Mô tả |
| --- | --- |
| `GET /cinemas`, `GET /cinemas/:id` | Rạp |
| `GET /movies?status=now_showing|coming_soon&genre=&age=&q=&sort=&page=` | Danh sách phim |
| `GET /movies/:slug` | Chi tiết phim |
| `GET /genres` | Thể loại |
| `GET /showtimes?movie_id=&cinema_id=&date=` | Suất chiếu (gom theo định dạng ở client hoặc `group=format`) trả kèm `available_seats`, `min_price`, `sale_status` |
| `GET /showtimes/:id` | Chi tiết suất + phim + phòng |
| `GET /showtimes/:id/seats` | Sơ đồ ghế + trạng thái + giá từng ghế |
| `GET /showtimes/:id/seats/stream` | **SSE** cập nhật trạng thái ghế |
| `GET /combos` | Combo đang bán (kèm `stock_status`) |
| `GET /banners`, `GET /news`, `GET /news/:slug`, `GET /pages/:slug` | Nội dung |
| `GET /vouchers/available?showtime_id=` | Voucher khả dụng (đã đăng nhập) |
| `GET /settings/public` | Cấu hình công khai (hotline, `hold_minutes`, `max_seats`…) |

**Mẫu `GET /showtimes/:id/seats`**
```json
{ "data": {
  "showtime": { "id": 12, "movie": {…}, "room": {"id":3,"name":"Cinema 03","type":"2D"},
                "start_at": "2026-10-23T12:30:00Z", "format": "2D Phụ đề", "sale_status": "selling" },
  "grid": { "rows": 8, "cols": 14 },
  "seats": [
    { "id": 301, "code": "A01", "row": "A", "col": 1, "x": 0, "y": 0,
      "type": "standard", "price": 85000, "status": "available" },
    { "id": 330, "code": "C05", "row": "C", "col": 5, "x": 4, "y": 2,
      "type": "vip", "price": 105000, "status": "held", "mine": false },
    { "id": 351, "code": "F07", "row": "F", "col": 7, "x": 6, "y": 5,
      "type": "couple", "price": 190000, "status": "available", "pair_with": 352 }
  ],
  "max_seats": 8 },
  "meta": { "server_time": "2026-10-23T02:15:00Z" } }
```
`status`: `available | held | sold | disabled`; `mine=true` nếu ghế đang được giữ bởi chính người dùng.

### 6.3 Đặt vé
| Endpoint | Mục đích |
| --- | --- |
| `POST /bookings` | **Tạo đơn & giữ ghế**. Header `Idempotency-Key`. Body `{ showtime_id, seat_ids:[…], accept_age_notice }` → `{ booking_code, status:'holding', hold_expires_at, items…, totals }`. Lỗi: `SEAT_UNAVAILABLE` (kèm `details.seat_ids`), `SEAT_LIMIT_EXCEEDED`, `HOLD_LIMIT_EXCEEDED`, `SHOWTIME_CLOSED` |
| `GET /bookings/:code` | Chi tiết đơn (chủ đơn hoặc staff) gồm `status`, `payment`, `tickets`, `totals`, `hold_expires_at` |
| `PUT /bookings/:code/seats` | Đổi ghế trong lúc `holding` (thay danh sách) |
| `PUT /bookings/:code/combos` | Đặt toàn bộ combo `{ items:[{combo_id, quantity}] }` → totals mới |
| `POST /bookings/:code/voucher` / `DELETE …/voucher` | Áp/gỡ voucher `{ code }` |
| `PUT /bookings/:code/contact` | Thông tin liên hệ |
| `POST /bookings/:code/redeem-points` | Dùng điểm `{ points }` (MAY) |
| `POST /bookings/:code/extend` | Gia hạn giữ ghế (1 lần) |
| `POST /bookings/:code/confirm` | Chốt đơn, chuyển sang bước thanh toán (tạo `payment pending`, đơn vẫn `holding`); đơn 0đ → `confirmed` ngay. Đơn chỉ thành `confirmed` khi thanh toán `paid` (theo mục 4.1 file 01) |
| `DELETE /bookings/:code` | Hủy đơn đang `holding` |
| `GET /me/bookings?status=&page=` | Lịch sử đơn của tôi |
| `GET /me/bookings/:code/tickets` | Danh sách vé của đơn |
| `POST /bookings/:code/resend-email` | Gửi lại vé |

**Cấu trúc `totals`**: `{ tickets_subtotal, combos_subtotal, discount, points_discount, tax, final_amount, currency:'VND' }`. Frontend **chỉ hiển thị**, không tự tính tổng cuối.

### 6.4 Thanh toán
| Endpoint | Mục đích |
| --- | --- |
| `GET /payments/config` | Cấu hình hiển thị: `use_custom_qr`, `qr_image_url`, `bank_name`, `bank_code`, `account_number`, `account_name`, `transfer_prefix` |
| `GET /bookings/:code/payment` | Thông tin thanh toán của đơn: `amount`, `transfer_content`, `qr: { type:'custom'|'vietqr', url }`, `status`, `reject_reason`, `proof_url`, `proof_deadline_at` |
| `POST /bookings/:code/payment/proof` | `multipart/form-data` (`file`, `paid_amount?`) → `status = proof_submitted`. Lỗi: `PAYMENT_PROOF_DUPLICATE`, `HOLD_EXPIRED`, `VALIDATION_ERROR` |
| `GET /bookings/:code/stream` | **SSE** thay đổi trạng thái đơn/thanh toán |

URL VietQR (nếu backend trả sẵn thì dùng; nếu FE tự dựng): `https://img.vietqr.io/image/{bank_code}-{account_number}-compact2.png?amount={amount}&addInfo={encodeURIComponent(transfer_content)}&accountName={encodeURIComponent(account_name)}`. **MUST** `encodeURIComponent`.

### 6.5 Vé & Check-in
| Endpoint | Quyền | Mục đích |
| --- | --- | --- |
| `GET /tickets/:ticketCode` | chủ vé/staff | Chi tiết vé |
| `POST /staff/checkin` | staff+ | Body `{ ticket_code, showtime_id?, override?, id_verified?, pin? }` → `{ result:'OK|ALREADY_USED|TOO_EARLY|…', ticket, booking, checked_in_at, checked_in_by }`. **HTTP 200** cho mọi kết quả nghiệp vụ |
| `POST /staff/checkin/booking` | staff+ | Check-in cả đơn |
| `GET /staff/showtimes/current` | staff+ | Suất trong ±2 giờ của rạp được gán |
| `GET /staff/checkin/stats?showtime_id=` | staff+ | Đã vào/tổng |
| `GET /staff/checkin/recent` | staff+ | Lịch sử quét |
| `POST /staff/combo-pickup` | staff+ | Xác nhận nhận combo |

### 6.6 POS & ca
| Endpoint | Mục đích |
| --- | --- |
| `GET /staff/shifts/current`, `POST /staff/shifts/open`, `POST /staff/shifts/:id/close` | Ca làm việc |
| `POST /staff/pos/bookings` | Tạo đơn POS (giữ ghế 3 phút) |
| `POST /staff/pos/bookings/:code/pay` | `{ method, amount_received?, reference? }` → `confirmed` + vé |
| `GET /staff/pos/today` | Vé/đơn trong ngày |
| `POST /staff/pos/bookings/:code/reprint` | Ghi log in lại |

### 6.7 Tài khoản
`GET/PUT /me/profile` · `GET /me/points` · `GET /me/notifications` · `POST /me/notifications/read` · `GET /me/favorites` · `PUT /me/favorites/:movieId` · `POST /me/reviews` (MAY).

> **Quy tắc đồng bộ**: mọi thay đổi hợp đồng API phải cập nhật `types/api.d.ts` và MSW handlers cùng lúc. Nếu backend chưa có endpoint, FE vẫn dựng trên MSW theo hợp đồng trên.

---

## 7. QUẢN LÝ DỮ LIỆU (TanStack Query)

### 7.1 Cấu hình mặc định
`staleTime: 60s` (danh mục), `gcTime: 5 phút`, `retry: 1` (không retry 4xx), `refetchOnWindowFocus: true` cho dữ liệu động (đơn, ghế).

### 7.2 Query keys (tập trung `lib/queryKeys.ts`)
```ts
qk.movies.list(params) · qk.movies.detail(slug) · qk.showtimes.list({movieId,cinemaId,date})
qk.seats(showtimeId) · qk.combos() · qk.booking(code) · qk.payment(code)
qk.me.bookings(params) · qk.me.profile() · qk.notifications()
```

### 7.3 Mutation & invalidate
- Tạo/sửa đơn → `setQueryData(qk.booking(code), …)` bằng dữ liệu trả về (không refetch thừa).
- Sau thanh toán/duyệt → invalidate `qk.booking`, `qk.me.bookings`.
- `useMutation` luôn xử lý `onError` bằng `mapErrorToMessage(code)`; các lỗi nghiệp vụ có hành động riêng (xem 9).

### 7.4 Chống gửi đúp
Nút submit disable khi `isPending`; `Idempotency-Key` sinh **một lần cho mỗi "ý định"** (lưu trong ref/store, đổi khi dữ liệu thay đổi); double-click không tạo 2 đơn.

---

## 8. TRẠNG THÁI & LOGIC TRỌNG TÂM

### 8.1 Booking store (Zustand, persist `sessionStorage`)
```ts
{ showtimeId, selectedSeatIds: number[], bookingCode?, idempotencyKey?,
  ack_age: boolean, contact: {...}, voucherCode?, usePoints?: number }
actions: toggleSeat(seat) · clearSeats() · setBooking(code) · reset()
```
- Chỉ chứa **lựa chọn của người dùng**; dữ liệu đơn thật lấy từ `useBooking(code)`.
- `reset()` khi: hoàn tất/ hủy/ hết hạn / đổi suất.

### 8.2 SeatMap – yêu cầu kỹ thuật
- Props: `seats`, `selectedIds`, `maxSeats`, `onToggle`, `readOnly`. **Render bằng SVG hoặc lưới CSS** theo `x,y`; mỗi ghế là component `memo` (so sánh `status`, `selected`).
- Ghế đôi: gộp `pair_with` thành một phần tử rộng 2 ô; chọn/bỏ chọn cả cặp.
- `seatMath.ts`: hàm thuần có test:
  - `calcSubtotal(seats, selectedIds)` (chỉ **tạm tính**).
  - `detectOrphanSeat(seats, selectedIds)` → trả ghế lẻ kẹp giữa để cảnh báo.
  - `isSelectable(seat)` (chỉ `available` hoặc `mine`).
- Bàn phím: roving tabindex, mũi tên di chuyển theo lưới, Space/Enter chọn; `aria-label` đầy đủ.
- Zoom/pan trên mobile bằng CSS transform (pointer events), giới hạn tỉ lệ 0.6–2.
- **Realtime**: `useSeatStream(showtimeId)` – mở SSE, nhận sự kiện `{ seat_id, status }` → cập nhật cache `qk.seats`; nếu ghế người dùng đang chọn bị `held/sold` bởi người khác → bỏ chọn + toast. SSE lỗi → fallback polling 5s (`refetchInterval`), tự reconnect có backoff (1s→30s).

### 8.3 Đồng hồ & giờ server
- `useServerTime()`: khi nhận `meta.server_time`, lưu `offset = serverNow - clientNow` (làm trơn bằng trung bình 3 lần gần nhất). `now()` = `Date.now() + offset`.
- `useCountdown(expiresAt)`: dùng `requestAnimationFrame`/`setInterval(250ms)` tính `remaining = expiresAt - now()`; cập nhật lại khi tab `visibilitychange` → visible.
- `HoldCountdown` phát `onExpire` đúng **một lần** → hiển thị modal chặn + gọi `GET /bookings/:code` xác nhận từ server (server là nguồn sự thật; nếu server còn `holding` do gia hạn thì tiếp tục).
- `onWarn` ở 120 giây (toast một lần).

### 8.4 Luồng đặt vé (sơ đồ)
```
SeatsPage ──POST /bookings──▶ (holding) ──▶ CombosPage ──PUT /combos──▶ ReviewPage
   ▲ lỗi SEAT_UNAVAILABLE                         │ voucher / contact / điều khoản
   └── highlight + reload ghế                    ▼
                                     POST /bookings/:code/confirm
                                                  │
                                         PaymentPage (QR + upload)
                                                  │ POST …/payment/proof
                                         proof_submitted ──SSE──▶ paid ──▶ TicketPage
                                                  │                 └──▶ rejected → gửi lại
                                                  └──▶ expired → "Chọn lại ghế"
```
- Quay lại bước trước khi đang `holding`: cho phép, giữ nguyên đơn (dùng `PUT /seats` thay vì tạo đơn mới).
- Đơn đã chốt (gọi `/confirm`, thanh toán `pending`/`proof_submitted`) → không cho đổi ghế/combo (chỉ hủy hoặc thanh toán).
- Đóng tab giữa chừng: khi quay lại, mục **Vé của tôi** hiển thị đơn `holding`/`proof_submitted` kèm nút tiếp tục.

### 8.5 Payment page – logic
1. Tải `GET /bookings/:code` + `GET /bookings/:code/payment`. Nếu `status` ≠ `holding` và không phải chờ thanh toán → điều hướng theo trạng thái.
2. Mở SSE `GET /bookings/:code/stream` (fallback polling 5s khi `proof_submitted`) và phản ứng theo `payment.status` (bảng 4.7b file 02).
3. `ProofUploader`: kiểm tra client (MIME `image/jpeg|png|webp`, ≤ 5MB, xem trước bằng `URL.createObjectURL` và **`revokeObjectURL`** khi dọn); nén ảnh phía client (canvas, cạnh dài ≤ 1600px, chất lượng 0.85) để giảm dung lượng (SHOULD); hiển thị tiến trình (`onUploadProgress`).
4. Nút sao chép: `navigator.clipboard.writeText` có fallback `document.execCommand('copy')`.
5. Nội dung chuyển khoản/ số tiền lấy **từ server**, không tự sinh.
6. Khi `paid`: `canvas-confetti` (dynamic import) + chuyển `/ve/:code`.

### 8.6 Vé điện tử
- QR giá trị = `ticket_code` (không thêm dữ liệu cá nhân). Kích thước ≥ 200px, `level="M"`, nền trắng, quiet zone ≥ 4 module.
- **Lưu ảnh/PDF**: `html-to-image` (`toPng`, `pixelRatio: 2`) → tải; PDF qua `jspdf`. Nhớ đảm bảo ảnh CORS (`crossOrigin="anonymous"` + header CORS ở backend) nếu poster từ domain khác, nếu không canvas bị "tainted".
- **.ics**: sinh chuỗi iCalendar (tên phim, giờ bắt đầu/kết thúc, địa điểm) tải về.
- `Screen Wake Lock API` khi mở vé (nếu hỗ trợ), `release` khi rời trang.
- `@media print` cho in vé.

### 8.7 Staff Scanner
- Import động `html5-qrcode`; cấu hình `fps: 10`, `qrbox` theo khung, ưu tiên camera sau (`facingMode: 'environment'`).
- **Chống quét lặp**: sau khi đọc một mã, tạm dừng nhận 1.5–2s và bỏ qua cùng mã trong khoảng đó; hàng đợi 1 request.
- Gọi `POST /staff/checkin` → hiển thị theo `result` (bảng 5.1 file 02). Âm thanh (`AudioContext`/file ngắn) + `navigator.vibrate` (nếu có).
- Xử lý quyền camera bị từ chối/không có camera → chuyển sang nhập tay. Cảnh báo nếu không phải HTTPS (trừ localhost).
- Dọn dẹp: `stop()` + `clear()` khi unmount; tránh rò rỉ stream.
- (MAY) Chế độ offline: tải `GET /staff/showtimes/:id/tickets-lite` vào IndexedDB, check-in cục bộ, đồng bộ hàng đợi khi có mạng.

### 8.8 POS
- Dùng lại `SeatMap` ở chế độ POS (giữ ghế 3 phút qua `POST /staff/pos/bookings`).
- Máy tính tiền thừa: hàm thuần `calcChange(received, total)`; chặn thanh toán khi `received < total` (tiền mặt).
- In vé: `window.print()` với stylesheet khổ 80mm; (MAY) `WebUSB/ESC-POS` hoặc dịch vụ in cục bộ – ghi rõ ngoài phạm vi P1.
- Chặn bán nếu chưa mở ca (theo `settings`).

---

## 9. XỬ LÝ LỖI & THÔNG BÁO

`mapErrorToMessage(code, details)` ánh xạ **mã lỗi (mục 9 file 01)** → câu tiếng Việt thân thiện + hành động:

| Mã | Thông điệp | Hành động UI |
| --- | --- | --- |
| `SEAT_UNAVAILABLE` | "Ghế {codes} vừa có người khác chọn." | Highlight, tải lại ghế, bỏ chọn ghế mất |
| `HOLD_EXPIRED` | "Đơn đã hết thời gian giữ ghế." | Modal → chọn lại ghế |
| `SHOWTIME_CLOSED` | "Suất chiếu đã đóng bán." | Về lịch chiếu |
| `VOUCHER_*` | Theo từng mã | Hiện lỗi dưới ô voucher |
| `PAYMENT_PROOF_DUPLICATE` | "Ảnh biên lai đã được dùng cho đơn khác." | Cho chọn ảnh khác |
| `TOO_MANY_REQUESTS` | "Bạn thao tác quá nhanh, thử lại sau X giây." | Disable tạm |
| `ACCOUNT_LOCKED` | "Tài khoản đang bị khóa." | Liên hệ hỗ trợ |
| `NETWORK` | "Mất kết nối. Kiểm tra mạng và thử lại." | Nút thử lại |
| `FORBIDDEN` | "Bạn không có quyền thực hiện." | – |
- Lỗi `VALIDATION_ERROR` có `details.fields` → gán vào `react-hook-form` bằng `setError`.
- `ErrorBoundary` bọc từng layout + route; hiển thị `ErrorState` kèm nút thử lại và `requestId`.
- Toast qua một nơi duy nhất (`useToast`), tối đa 3 toast cùng lúc.

---

## 10. FORM & VALIDATION (zod mẫu)

```ts
export const registerSchema = z.object({
  name: z.string().trim().min(2, 'Vui lòng nhập họ tên').max(100),
  email: z.string().trim().toLowerCase().email('Email không hợp lệ'),
  phone: z.string().regex(/^(0|\+84)\d{9}$/, 'Số điện thoại không hợp lệ'),
  password: z.string().min(8, 'Tối thiểu 8 ký tự').regex(/[A-Za-z]/).regex(/\d/),
  confirm: z.string(),
  accept_terms: z.literal(true, { errorMap: () => ({ message: 'Bạn cần đồng ý điều khoản' }) }),
}).refine(v => v.password === v.confirm, { path: ['confirm'], message: 'Mật khẩu nhập lại không khớp' });
```
- Validate `onBlur` + `onSubmit`; phía server là nguồn kiểm tra cuối.
- Sanitize hiển thị: **không** dùng `dangerouslySetInnerHTML` với nội dung người dùng; nội dung CMS (tin tức) phải qua `DOMPurify`.

---

## 11. HIỆU NĂNG
- Code-split: `lazy()` mỗi route; Staff/Admin/scan/confetti/pdf tách chunk riêng. `manualChunks` cho `react`, `vendor`.
- Ảnh: WebP + `srcset`, `width/height` cố định, `loading="lazy"` (trừ ảnh LCP → `fetchpriority="high"`).
- Danh sách dài (admin, lịch sử): phân trang hoặc `@tanstack/react-virtual`.
- Tránh re-render: `memo` cho `SeatCell`, `MovieCard`; selector Zustand hẹp; `useMemo` cho tính toán sơ đồ.
- Debounce tìm kiếm 300ms; hủy request cũ (`AbortController`/`signal` của TanStack Query).
- Prefetch chi tiết phim khi hover; preconnect tới API/ảnh.
- Mục tiêu: LCP < 2.5s, CLS < 0.1, INP < 200ms; bundle khởi tạo (gzip) < 200KB.

---

## 12. BẢO MẬT FRONTEND
- Không lưu access token ở `localStorage`; không log token/PII ra console production.
- Chặn open-redirect: `returnTo` chỉ chấp nhận đường dẫn nội bộ bắt đầu bằng `/` và không phải `//`.
- Kiểm tra file upload phía client (nhưng backend mới là chốt chặn).
- CSP khuyến nghị (cấu hình ở server/CDN): `default-src 'self'; img-src 'self' data: https://img.vietqr.io; frame-src https://www.youtube-nocookie.com; script-src 'self'`.
- Trailer: dùng `youtube-nocookie.com/embed/{id}`, tách `id` từ URL bằng parser an toàn (chỉ chấp nhận `^[\w-]{11}$`).
- Liên kết ngoài `rel="noopener noreferrer"`.
- Phụ thuộc: `npm audit`, khóa phiên bản (lockfile), Dependabot.

---

## 13. TEST & CHẤT LƯỢNG

| Loại | Công cụ | Phạm vi bắt buộc |
| --- | --- | --- |
| Unit | Vitest | `seatMath`, `format`, `calcChange`, `mapErrorToMessage`, parser YouTube, server-time offset |
| Component | Testing Library | `SeatMap` (chọn/bỏ, tối đa ghế, ghế đôi, bàn phím), `HoldCountdown`, `ProofUploader`, form đăng ký/đăng nhập |
| Tích hợp | MSW | Luồng đặt vé với các nhánh `SEAT_UNAVAILABLE`, `HOLD_EXPIRED`, voucher lỗi |
| E2E | Playwright | (1) khách đặt vé → gửi biên lai → (mock duyệt) → xem vé; (2) hai trình duyệt tranh một ghế; (3) staff quét vé hợp lệ/đã dùng; (4) hết giờ giữ ghế |
| a11y | `jest-axe`/`@axe-core/playwright` | Các trang chính |
| Visual (MAY) | Playwright snapshot | Vé, sơ đồ ghế |
Mục tiêu độ phủ: logic thuần ≥ 90%, tổng ≥ 70%. CI chạy: `lint`, `tsc`, `test`, `build`, E2E smoke.

---

## 14. LỘ TRÌNH TRIỂN KHAI CHO AI (THỨ TỰ LÀM)

1. **Nền tảng**: Vite + TS + Tailwind + alias + ESLint/Prettier + `env.ts` + `http.ts` + `queryClient` + `AuthProvider` + router khung + `components/ui` cốt lõi + MSW.
2. **Xác thực**: đăng ký/đăng nhập/quên & đặt lại mật khẩu, `ProtectedRoute`, refresh token.
3. **Danh mục công khai**: Trang chủ, danh sách/chi tiết phim, lịch chiếu, TrailerModal, QuickBooking.
4. **Đặt vé lõi**: SeatMap + SSE, tạo đơn/giữ ghế, `HoldCountdown`, combo, review + voucher.
5. **Thanh toán QR**: trang thanh toán, upload biên lai, trạng thái (chờ/từ chối/hết hạn), SSE.
6. **Vé điện tử**: hiển thị, lưu ảnh/PDF, .ics, in.
7. **Tài khoản**: hồ sơ, vé của tôi, điểm, thông báo.
8. **Staff**: Scanner, POS, ca, vé trong ngày.
9. **Hoàn thiện**: i18n, a11y, hiệu năng, test E2E, tài liệu README FE.
(Admin: xem file 04, có thể làm song song sau bước 2.)

## 15. TIÊU CHÍ HOÀN THÀNH (DEFINITION OF DONE)
- [ ] Mọi route trong mục 5 hoạt động, có 404/403/lỗi.
- [ ] Không có `any` ngầm; `tsc --noEmit` sạch; ESLint sạch.
- [ ] Luồng đặt vé chạy đủ nhánh lỗi (ghế bị mất, hết hạn, voucher, từ chối biên lai).
- [ ] Đồng hồ khớp giờ server (sai lệch < 1 giây trong thử nghiệm với đồng hồ máy lệch 2 phút).
- [ ] Sơ đồ ghế realtime; mất SSE vẫn hoạt động nhờ polling.
- [ ] Không lưu access token ở storage bền vững; `returnTo` an toàn.
- [ ] Lighthouse mobile: Performance ≥ 80, Accessibility ≥ 90.
- [ ] Test unit/tích hợp/E2E theo mục 13 pass trong CI.
- [ ] README FE: cài đặt, biến môi trường, lệnh chạy, cấu trúc, quy ước.
