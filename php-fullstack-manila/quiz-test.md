Quiz:

1. Analys this website API

https://43.213.218.98:8000/?800tk#/pages/index/index

# Trả lời

## Nhiệm vụ 1. Phân tích API của website

### 1. Tóm tắt

- **Kiến trúc:** website là SPA (uni-app / Vue 2). Trang gốc chỉ trả HTML khung; giao diện và dữ liệu được nạp sau bằng JavaScript.
- **API quan sát bằng DevTools:** 3 nhóm, 6 request Fetch/XHR trên 3 trang (Trang chủ, 发现, 好站).
- **3 điểm đáng chú ý:**
  1. Ba nhóm API dùng **ba định dạng response khác nhau**.
  2. Nhóm `/lhc` **ký (MD5) và mã hoá (AES-CBC)** toàn bộ tham số và response trong trường `_data`.
  3. API hình ảnh có **phân trang rõ ràng** (`meta.total`, `totalPages`, `hasMore`).

### 2. Phương pháp và phạm vi

- **Công cụ:** Chrome DevTools (tab Network), `curl`, đọc mã JavaScript công khai của client.
- **Phạm vi:** phân tích thụ động; không đăng nhập, không gửi request ghi dữ liệu.
- **Mức xác minh:**
  - **Đã quan sát** bằng DevTools: Trang chủ, 发现, 好站.
  - **Suy ra từ mã client:** các endpoint khác; được đánh dấu riêng ở mục 4.4.
  - **Chưa khảo sát:** các nút/trang con còn lại.

### 3. Luồng tải trang

```
Trình duyệt
  │ ① GET /?800tk
  ▼
nginx 43.213.218.98:8000 ──► HTML khung, chỉ có thẻ <script>           [Doc]
  │ ② tải script từ api.bdmijg.com (loader, cấu hình, tracking)
  ▼
api.bdmijg.com/js-1-index ──► HTML nén (base64 + deflate)
  │ ③ giải nén → document.write(HTML thật)
  ▼
cdn.edgovn.com ──► bundle uni-app (index.js, chunk-vendors.js)
  │ ④ Vue dựng giao diện trang chủ
  ▼
Người dùng bấm nút ──► ⑤ tải chunk của route ──► gọi API              [Fetch/XHR]
```

**Điểm vào là request loại Document, không phải API.** `GET /?800tk` trả `200`, `text/html`, `gzip`, `cache-control: max-age=60, public, s-maxage=600`, `x-cache-status: STALE` (nginx proxy cache). Body không chứa dữ liệu nghiệp vụ. API dữ liệu chỉ xuất hiện sau khi bundle chạy.

### 4. Danh mục API

#### 4.1. Kiến trúc API

```
                    ┌──────────────────────────┐
                    │  SPA uni-app (trình duyệt)│
                    └────┬─────────┬─────────┬─┘
                  A      │       B │       C │
        ┌────────────────▼─┐ ┌─────▼────────┐ ┌▼─────────────────┐
        │ Nghiệp vụ /lhc   │ │ Hình / video │ │ Danh sách site   │
        │ tkapi1/2/3       │ │ tuku-be      │ │ api.edgoalb.com  │
        │ .352722.com      │ │ .ptallenvery │ │ /sites           │
        │ MD5 + AES (_data)│ │ JSON thường  │ │ data mã hoá      │
        └──────────────────┘ └──────┬───────┘ └──────────────────┘
                                    │ URL ảnh
                                    ▼
                         CDN ảnh tuku.ptallenvery.com
```

| | A. Nghiệp vụ `/lhc` | B. Hình / video | C. Danh sách site |
|---|---|---|---|
| Host | `tkapi1/2/3.352722.com` (client chọn ngẫu nhiên) | `tuku-be.ptallenvery.com` | `api.edgoalb.com` |
| Chức năng | Diễn đàn, thành viên, bình chọn | Ảnh theo loại xổ số | Danh sách site "好站" |
| Truy cập (đọc) | Công khai | Công khai | Công khai |
| Mã hoá | Request và response (`_data`) | Không | Response (`encode=true`) |
| Envelope | `{_data, version}` → `{status, message, data}` | `{success, data, meta}` | `{code, msg, data, encoded}` |
| Phân trang | `per_page`, `page` | `limit`, `page` + `meta` | `page` |

#### 4.2. API đã quan sát bằng DevTools

| # | Trang | Method + URL | Nhóm | Status |
|---|---|---|---|---|
| 1 | Trang chủ | `GET tuku-be.ptallenvery.com/api/public/pictures/macau?sort=site&page=1&limit=100` | B | 200 |
| 2 | 发现 | `GET tkapi3.352722.com/lhc/bbs/categories?_data=abc...` | A | 200 |
| 3 | 发现 | `GET tkapi3.352722.com/lhc/bbs/views?per_page=10&page=1&_data=abc...` | A | 200 |
| 4 | 好站 | `GET api.edgoalb.com/sites?type=hklhc&page=1&encode=true` | C | 200 |
| 5 | 好站 | `GET api.edgoalb.com/sites?type=am2lhc&page=1&encode=true` | C | 200 |
| 6 | 好站 | `GET api.edgoalb.com/sites?category_id=h100&page=1&encode=true` | C | 200 |

Trang 发布 và 会员登陆 là form đăng nhập tĩnh, không phát sinh API. Chi tiết request/response nằm ở [api-info.md](./api-info.md).

#### 4.3. Phân tích chi tiết hai API tiêu biểu

**API #1: `GET /api/public/pictures/macau`** (JSON thường)

| Thành phần | Chi tiết |
|---|---|
| Query | `sort=site`, `page=1`, `limit=100` |
| Auth | Không cần |
| Response | `{ success, data: [...], meta }` |
| `data[]` | `id`, `lottery`, `category`, `filename`, `name`, `keyword`, `letter`, `sortOrder`, `latest {year, issue, url, thumbnailUrl, originalUrl, width, height}`, `urlTemplate`, `thumbnailUrlTemplate`, `fetchUrlTemplate` |
| `meta` | `page`, `perPage`, `total` = 2103, `totalPages` = 22, `hasMore` |

Nhận xét:
- Phân trang kiểu offset, `hasMore` giúp client không phải tự tính trang cuối.
- `urlTemplate` với `{year}/{issue}` cho phép dựng URL các kỳ trước mà không cần gọi thêm API.
- `?v=` trên URL ảnh dùng làm cache-busting.
- `fetchUrlTemplate` trỏ tới `http://localhost/...`, có vẻ là giá trị cấu hình môi trường phát triển.

**API #3: `GET /lhc/bbs/views`** (ký và mã hoá)

| Thành phần | Chi tiết |
|---|---|
| Query | `per_page=10`, `page=1` (dạng thường) và `_data` (AES, base64) |
| Plaintext `_data` | `page`, `category_id`, `isLogo`, `sort`, `title`, `timestamp`, `sign` |
| Response | `{ _data, version: "1.0.0" }` → giải mã: `{ status, message, data }` |
| Auth | Không cần token để đọc |

Nhận xét:
- Plaintext cho thấy các bộ lọc hỗ trợ: `category_id`, `title` (tìm kiếm), `sort`.
- `timestamp` và `sign` (MD5 của các tham số đã sắp xếp) cho phép phía server kiểm tra tính toàn vẹn và hạn dùng của request.
- Plaintext của API #2 đã giải mã thành `{ timestamp, sign }`; response là danh mục `{ id, name, logo, sort, status, remark, created_at, updated_at }`.

**Pipeline request nhóm A** (đã đối chiếu bằng cách giải mã dữ liệu thật):

```
tham số + timestamp → sắp xếp "k=v" nối "&" → sign = MD5
  → AES-128-CBC → { _data } trên query
  → response { _data, version } → AES giải mã → { status, message, data }
```

#### 4.4. Endpoint suy ra từ mã client (chưa quan sát request thật)

| Nhóm | Endpoint | Mục đích |
|---|---|---|
| A (`/lhc`, đọc) | `pictures`, `picture-replies`, `draws`, `draws/{id}`, `draw/lastdraw`, `bbs-replies`, `setting-year/{year}`, `vote/{id}/type/{t}` | Hình, bình luận, kết quả mở thưởng, bình chọn |
| A (`/lhc`, ghi, cần token) | `bbs`, `bbs/{id}/like`, `bbs-replies`, `picture-replies`, `vote`, `member`, `member/login`, `member/logout` | Đăng bài, thích, bình chọn, đăng ký, đăng nhập |
| B | `pictures/{type}`, `pictures/detail/{id}`, `videos/{type}` | Hình chi tiết, video |
| Realtime | `wss://ws.edgoapp.com` (socket.io) | Kết quả mở thưởng trực tiếp (kết nối thật chưa xác minh) |

### 5. Quan sát về thiết kế API

- **Ưu điểm:**
  - Nhóm B có phân trang và `meta` rõ ràng.
  - Hạ tầng phân tách tốt: loader, cấu hình, API và CDN ảnh ở các domain riêng; đổi host API không cần build lại app.
  - Nhóm A có `timestamp` và `sign` giúp hạn chế request giả mạo hoặc bị phát lại.
- **Điểm cần lưu ý khi kiểm thử:**
  - Ba envelope response khác nhau nên client và test phải xử lý riêng từng nhóm.
  - Nhóm A chỉ gọi được qua `_data` đã ký; response mã hoá khiến debug và kiểm thử bằng công cụ chung khó hơn.
  - Tham số `timestamp` thay đổi mỗi lần gọi, nên request nhóm A không cache được theo URL.
  - URL không có version (chỉ có tiền tố `/lhc/`).

### 6. Chiến lược kiểm thử

> Chỉ thực hiện trong phạm vi được phép; ưu tiên API đọc công khai.

| Loại | Nội dung kiểm tra | Áp dụng cho |
|---|---|---|
| Chức năng | Status `200` và đúng envelope; schema và kiểu dữ liệu; `meta.total`, `totalPages`, `hasMore` nhất quán (2103 mục, `limit=100` → 22 trang, trang cuối `hasMore=false`); lọc `category_id`, `title`, `type` | #1, #3, #4–6 |
| Tiêu cực | `page=0`, `-1`, `abc`; `page` vượt `totalPages`; `limit=0` hoặc rất lớn; `type` không tồn tại; `_data` thiếu, sai base64, sai `sign`, `timestamp` quá cũ → kỳ vọng `400/422`, không `500`; POST vào endpoint GET → `405` | #1, #3 |
| Bảo mật | API đọc chạy không cần token, API ghi trả `401` khi thiếu token; response không chứa thông tin nội bộ (ví dụ `localhost`); request phát lại sau khi hết hạn `timestamp` bị từ chối; rate limit ở mức nhẹ | A, B |
| Hiệu năng | Thời gian phản hồi p50/p95 của từng host `tkapi1/2/3`; kích thước response theo `limit` và hiệu quả `gzip`; `Cache-Control` / `ETag` của API và CDN ảnh; hành vi khi một host không phản hồi | A, B |

**Công cụ:** DevTools (quan sát), Postman/Newman (kịch bản và kiểm tra schema), k6 hoặc JMeter (hiệu năng, khi được phép).

### 7. Kết luận

- **Đã xác định:** 3 nhóm API (nghiệp vụ `/lhc` có ký và mã hoá, hình/video JSON thường, danh sách site mã hoá). Trang chủ chỉ là điểm vào; API xuất hiện khi người dùng thao tác ở các trang con.
- **Luồng chính:** bấm nút → tải chunk của route → gọi API qua lớp gọi chung → giải mã → hiển thị.
- **Ưu tiên kiểm thử:** phân trang và tính nhất quán dữ liệu API hình; tính đúng của `sign`/`timestamp` ở nhóm `/lhc`; xử lý lỗi giữa 3 định dạng response.
- **Giới hạn:** chỉ dựa trên DevTools và mã client công khai; một số trang/nút chưa khảo sát; WebSocket và cách giải mã của `api.edgoalb.com` chưa xác minh; chưa có số liệu hiệu năng thực đo.
- **Cần thêm:** tài liệu API (OpenAPI/Swagger), tài khoản và môi trường test, xác nhận phạm vi kiểm thử bảo mật/hiệu năng.
