Quiz:

1. Analys this website API

https://43.213.218.98:8000/?800tk#/pages/index/index

# Trả lời

## Nhiệm vụ 1. Phân tích API của website

### 1. Tóm tắt
- Website "800图库" (800tk) là một SPA xây bằng **uni-app (Vue 2 + webpack, build H5)**, nội dung là tra cứu hình/video/kết quả xổ số Lục Hợp Thái (Hong Kong, Macau, KL8) kèm diễn đàn, đăng ký/đăng nhập, điểm thưởng.

- Trang gốc `43.213.218.98:8000` (nginx) chỉ trả **HTML rỗng** (chỉ có thẻ `<script>`). Trình duyệt tải `js-1-index` từ `api.bdmijg.com`, giải base64 và deflate ra HTML thật, rồi nạp `index.js` và `chunk-vendors.js` từ CDN `cdn.edgovn.com` để Vue (uni-app) dựng giao diện. Sơ đồ chi tiết ở mục 3.

- Dữ liệu qua **REST API (đường dẫn `/lhc/...`, có thể dựng bằng Laravel — suy luận từ `per_page` và mã lỗi 422, chưa có bằng chứng trực tiếp)** có cân bằng tải phía client giữa 3 host `tkapi1/2/3.352722.com` (theo cấu hình và code). Cấu hình có khai báo **WebSocket/socket.io** cho dữ liệu realtime (chưa quan sát kết nối thật). Tham số request/response nhóm `/lhc` được **ký (MD5) + mã hoá (AES-CBC)** ở client (đã xác minh bằng cách giải mã dữ liệu thật).

### 2. Phương pháp & phạm vi
- Công cụ: `curl`, đọc tĩnh các file JS công khai. Không chạy script của site, không đăng nhập, không gửi request ghi dữ liệu.
- Phạm vi: chỉ phân tích **thụ động**; không tấn công, không brute-force, không thử xâm nhập.
- Mức xác minh: **đã xác minh thực tế bằng DevTools** cho các trang index, 发现 (forum/find), 好站 (website), 发布/会员登陆 (login); phần còn lại được **suy ra từ mã nguồn client (static)** và đánh dấu rõ. Các nút/trang con khác chưa khảo sát.

### 3. Kiến trúc & luồng tải trang

**Sơ đồ luồng tải trang (từ lúc gõ URL đến lúc có giao diện):**

```
Trình duyệt
  │ ① GET /?800tk
  ▼
nginx 43.213.218.98:8000 ──► HTML rỗng (chỉ có thẻ <script>)          [Doc]
  │
  │ ② đọc HTML → tải các script
  ▼
api.bdmijg.com
  ├─ runtime.js    chống bot (nghi bot → xoá trắng trang)
  ├─ rc.js         thống kê/tracking (/e)
  ├─ pako.min.js   thư viện giải nén
  ├─ com.js        hàm base64 → byte
  └─ js-1-index    chứa "HTML đã nén" (base64 + deflate)
  │
  │ ③ giải base64 → giải nén deflate → document.write(HTML thật)
  ▼
HTML thật (tiêu đề "800图库", <div id="app">) nạp tiếp:
  ├─ api.bdmijg.com/js-b-tkconfig        cấu hình: host API, banner, link app
  ├─ cdn.edgovn.com/.../chunk-vendors.js thư viện (Vue, uni-app)
  └─ cdn.edgovn.com/.../index.js         mã ứng dụng (router, màn hình)
  │
  │ ④ Vue chạy → dựng giao diện vào <div id="app">
  ▼
Trang chủ hiển thị (banner, menu, nút)
  │
  │ ⑤ người dùng bấm nút (vd. 发现) → tải chunk của route → gọi API [Fetch/XHR]
  ▼
Các API (xem sơ đồ ở mục 4)
```

Chi tiết từng tài nguyên:

| Bước | Tài nguyên | Vai trò |
|---|---|---|
| 1 | `GET /?800tk` trên nginx `43.213.218.98:8000` | Trả HTML tối giản (cache `max-age=60`, `s-maxage=600`) |
| 2 | `api.bdmijg.com/assets/js/runtime.js` | Phát hiện bot/headless (webdriver, Playwright, Cypress…) → nếu nghi bot thì xoá trắng trang |
| 3 | `api.bdmijg.com/assets/js/rc.js` + `window.__cmsTrack` | Theo dõi/thống kê gửi về `/e` kèm site id và khoá `k` |
| 4 | `api.bdmijg.com/js-1-index?v=…` | Chứa HTML đã nén (deflate+base64); `pako.inflateRaw` giải nén rồi `document.write` |
| 5 | `api.bdmijg.com/js-b-tkconfig` | Cấu hình động: danh sách host API, socket, CDN, banner, link tải app, thông báo tên miền dự phòng |
| 6 | `cdn.edgovn.com/800/static/js/index.*.js`, `chunk-vendors.*.js` | Bundle uni-app, tải lazy ~40 chunk theo route |
| 7 | `config.listAPI` (random 1 trong 3 host) | Gọi REST API `/lhc/...` |
| 8 | `wss://ws.edgoapp.com` / `ws1–5.servers01.com` | Realtime (socket.io 4.7.5) |

**Entry point (Document, không phải API)** — đã xác minh bằng DevTools:
- `GET https://43.213.218.98:8000/?800tk` → `200 OK`, `text/html; charset=UTF-8`, `content-encoding: gzip`, `transfer-encoding: chunked`. Loại request là **Doc (Document)**, không phải Fetch/XHR: đây là request điều hướng của trình duyệt (`sec-fetch-dest: document`, `sec-fetch-mode: navigate`) để tải trang HTML.
- Response header: `server: nginx/1.30.5`, `cache-control: max-age=60, public, s-maxage=600`, `vary: Accept-Encoding`, **`x-cache-status: STALE`** (nginx proxy cache phục vụ bản cũ; lần gọi đầu của tôi là `MISS`).
- Request cookie chỉ có cookie thống kê Baidu (`Hm_lvt_*`, `HMACCOUNT`, `Hm_lpvt_*`), **không có cookie phiên** → trang vào công khai.
- Body là **HTML khung rỗng** (`<body class="bg-light"></body>`) chứa các thẻ `<script>`/`<link>` tới `api.bdmijg.com` và `window.__cmsTrack`; **không có dữ liệu nghiệp vụ**.
- Nội dung giao diện đến từ (1) HTML nén trong `js-1-index`, (2) cấu hình trong `js-b-tkconfig`, (3) bundle uni-app. API dữ liệu chỉ được gọi sau khi bundle chạy (xem mục 4.1).

Các route (hash router `#/pages/...`): `index`, `article`, `detail`, `forum`, `live`, `open` (kết quả mở thưởng), `login`, `register`, `website`, `mine/*` (tài khoản, điểm, điểm danh, nâng cấp, yêu thích…).

Domain / CDN liên quan:

| Domain | Vai trò |
|---|---|
| `43.213.218.98:8000` | Origin nginx (trang vào) |
| `api.bdmijg.com` | Loader, chống bot, tracking, cấu hình động |
| `cdn.edgovn.com`, `otc.edgovn.com` | Bundle front-end, ảnh/banner, thư viện (`socket.io`, `TcPlayer`, `jzt.js`) |
| `tkapi1/2/3.352722.com` | REST API chính (`baseLHC = {domain}/lhc/`, `baseJson = {domain}/json/`) |
| `tuku-be.ptallenvery.com` | API công khai cho hình ảnh/video (`TUKU_BASE`) |
| `352722.com/api/` , `api.edgoalb.com` | `libApi`; API danh sách site (`/sites?…&encode=true`) |
| `ws.edgoapp.com`, `ws1–5.servers01.com` | WebSocket realtime |
| `xgtk{ver}.664460.com`, `au{ver}.ptallenvery.com` | Host ảnh HK / Macau; `{ver}` = ngày+giờ+phút → đổi liên tục (chống chặn/cache) |
| `hm.baidu.com`, `libs.baidu.com` | Thống kê Baidu, jQuery |

### 4. Danh mục API / endpoint

#### 4.0. Kiến trúc API

**Sơ đồ tổng thể:**

```
┌────────────────────────────────────────────────────────────┐
│ Trình duyệt: SPA uni-app (Vue 2)                           │
│  Trang/nút ─► lớp gọi API chung `apiLHCResquest` (nhóm /lhc)│
│                lấy cấu hình host từ `js-b-tkconfig`        │
└───┬──────────────┬───────────────┬───────────────┬─────────┘
    │ A            │ B             │ C             │ D (chưa xác minh)
    ▼              ▼               ▼               ▼
┌─────────────┐ ┌────────────┐ ┌────────────────┐ ┌───────────────┐
│ API nghiệp  │ │ API hình/  │ │ API danh sách  │ │ WebSocket     │
│ vụ /lhc     │ │ video      │ │ site           │ │ socket.io     │
│ tkapi1/2/3  │ │ tuku-be.   │ │ api.edgoalb.com│ │ ws.edgoapp.com│
│ .352722.com │ │ ptallenvery│ │ /sites         │ │ (kết quả      │
│             │ │ .com       │ │                │ │  realtime)    │
│ diễn đàn,   │ │ /api/public│ │ ?type|category │ │               │
│ thành viên, │ │            │ │ &encode=true   │ │               │
│ bình chọn…  │ │ JSON thường│ │ data mã hoá    │ │               │
│ ký MD5 +    │ │ công khai  │ │ công khai      │ │               │
│ AES (_data) │ │ meta phân  │ │ {code,msg,data,│ │               │
│ token (ghi) │ │ trang      │ │  encoded}      │ │               │
└─────────────┘ └─────┬──────┘ └────────────────┘ └───────────────┘
                      │ URL ảnh
                      ▼
              CDN ảnh: tuku.ptallenvery.com (ảnh, thumbnail, ảnh gốc)

Ngoài luồng dữ liệu: api.bdmijg.com (cấu hình, tracking /e), Baidu Hm (thống kê)
```

**Bảng so sánh 3 nhóm API đã quan sát:**

| | A. Nghiệp vụ `/lhc` | B. Hình/video `tuku-be` | C. Danh sách site `edgoalb` |
|---|---|---|---|
| Host | `tkapi1/2/3.352722.com` (random phía client) | `tuku-be.ptallenvery.com` | `api.edgoalb.com` |
| Chức năng | Diễn đàn, thành viên, bình chọn, kết quả | Ảnh, video theo loại xổ số | Danh sách site "好站" |
| Truy cập | Đọc: công khai; ghi: cần token | Công khai | Công khai |
| Ký/mã hoá | MD5 + AES-CBC cả request và response | Không (`noCrypt`) | Response mã hoá (`encode=true`) |
| Envelope | `{_data, version}` → `{status, message, data}` | `{success, data, meta}` | `{code, msg, data, encoded}` |
| Phân trang | `per_page`, `page` | `limit`, `page` + `meta` | `page` |

**Pipeline một request nhóm A (từ code, đã đối chiếu bằng giải mã dữ liệu thật):**

```
tham số → thêm timestamp → sắp xếp "k=v" nối "&" → sign = MD5(chuỗi)
   → gói {tham số, timestamp, sign} → AES-128-CBC → { _data } trên query/body
   → chọn ngẫu nhiên 1 host trong listAPI → gửi (kèm authorization nếu cần)
   ◄── response { _data, version } → AES giải mã → JSON { status, message, data }
   → xử lý status (400/401/403/422) → hiển thị
```

**Các lớp kiến trúc:**
1. **Cấu hình** (`js-b-tkconfig`): host, CDN, banner, link app → đổi host không cần build lại app.
2. **Lớp gọi API duy nhất** (`apiLHCResquest`): tập trung xác thực, ký, mã hoá, xử lý lỗi.
3. **Dịch vụ dữ liệu tách theo miền**: nghiệp vụ, hình/video, danh sách site.
4. **CDN** cho ảnh và bundle tĩnh.
5. **Realtime** qua WebSocket (chỉ thấy trong cấu hình).

#### 4.1. API đã quan sát bằng DevTools (xác minh thực tế)

Tổng hợp theo trang:

| Trang / nút | Route | API (Fetch/XHR) | Ghi chú |
|---|---|---|---|
| Trang chủ | `#/pages/index/index` | `GET tuku-be.ptallenvery.com/api/public/pictures/macau?sort=site&page=1&limit=100` | Danh sách hình, JSON thường |
| 发现 (Khám phá) | `#/pages/forum/find` | `GET tkapi3.352722.com/lhc/bbs/categories`, `GET …/lhc/bbs/views?per_page=10&page=1` | Diễn đàn; request/response mã hoá |
| 好站 (Trang tốt) | `#/pages/website/index` | `GET api.edgoalb.com/sites?type=hklhc\|am2lhc&page=1&encode=true`, `…?category_id=h100&page=1&encode=true` | Danh sách site; response mã hoá |
| 发布 (Đăng bài), 会员登陆 (Đăng nhập) | `#/pages/login/login` | Không có | Chỉ render form đăng nhập tĩnh |

**API #1 — Danh sách hình (Macau)**
- `GET https://tuku-be.ptallenvery.com/api/public/pictures/macau?sort=site&page=1&limit=100` → `200`.
- Query: `sort=site`, `page=1`, `limit=100`. Không mã hoá, không `authorization` (API công khai).
- Response (JSON thường):
  - Envelope: `{ "success": true, "data": [...], "meta": {...} }`.
  - Mỗi phần tử `data`: `id` (chuỗi), `lottery` (ví dụ mock `sample`), `category` (`col`), `filename`, `name` (ví dụ mock "Sample picture"), `keyword`, `letter` (A–Z, dùng cho lọc theo chữ cái), `sortOrder`, `latest` (`year`, `issue`, `url`, `thumbnailUrl`, `originalUrl`, `width`, `height`), `urlTemplate`, `thumbnailUrlTemplate`, `fetchUrlTemplate`.
  - `meta`: `page`, `perPage`, `total` (2103), `totalPages` (22), `hasMore`, `site` (`null`).
- Nhận xét: phân trang kiểu offset rõ ràng, có `hasMore` giúp client không cần tính; có URL mẫu (`{year}/{issue}`) để suy ra URL ảnh các kỳ trước; ảnh tách sang CDN `tuku.ptallenvery.com` (có thumbnail và ảnh gốc `O/`); `?v=` làm cache-busting. **Lưu ý:** `fetchUrlTemplate` trỏ tới `http://localhost/api/images/...` (không dùng được từ bên ngoài, nên bỏ hoặc đổi sang domain thật).

**API #2 — Danh mục diễn đàn**
- `GET https://tkapi3.352722.com/lhc/bbs/categories?_data=<ciphertext>` → `200`.
- Request: toàn bộ tham số đi trong query `_data` (AES, base64, URL-encode). Giải mã ra: `{ "timestamp": "<10 số>", "sign": "<md5>" }`.
- Response: `{ "_data": "<ciphertext>", "version": "1.0.0" }`. Giải mã ra: `{ "status": "success", "message": "show success", "data": [ { id, name, logo, sort, status, remark, created_at, updated_at } ] }`. Ví dụ danh mục: `1 = 澳彩 (Macau)`, `2 = 港彩 (Hong Kong)`.

**API #3 — Danh sách bài diễn đàn**
- `GET https://tkapi3.352722.com/lhc/bbs/views?per_page=10&page=1&_data=<ciphertext>` → `200`.
- Query: `per_page=10`, `page=1` lộ ở dạng thường **và** nằm trong `_data`. Giải mã `_data` ra: `{ "page": 1, "category_id": "", "isLogo": 1, "sort": [], "title": "", "timestamp": "<10 số>", "sign": "<md5>" }` → lộ ra bộ lọc `category_id`, `title` (tìm kiếm), `sort`.
- Response: `{ "_data": "...", "version": "1.0.0" }` (mã hoá, cùng dạng với #2).

**API #4–#6 — Danh sách site ("好站")**
- `GET https://api.edgoalb.com/sites?type=hklhc&page=1&encode=true`
- `GET https://api.edgoalb.com/sites?type=am2lhc&page=1&encode=true`
- `GET https://api.edgoalb.com/sites?category_id=h100&page=1&encode=true`
- Query: `type` (`hklhc` = Hong Kong, `am2lhc` = Macau) hoặc `category_id`, `page`, `encode=true`.
- Response: `{ "code": 1, "msg": "", "data": "<chuỗi mã hoá>", "encoded": true }`. Client tự yêu cầu mã hoá qua `encode=true`, và cờ `encoded` cho biết `data` là chuỗi mã hoá. _Thuật toán giải mã của endpoint này chưa được xác minh._

**Cơ chế gọi chung đã xác minh bằng quan sát thực tế (nhóm `tkapi*/lhc`)**
- Quan sát thực tế: các request **GET** bọc tham số trong `_data` ngay trong **query string** (ciphertext nằm trên URL). Request POST chưa quan sát; theo mã client thì dùng cùng cơ chế.
- Plaintext request luôn chứa `timestamp` (Unix giây) và `sign` (MD5 của các tham số đã sắp xếp); response bọc trong `_data` + `version`.

**Ba định dạng response khác nhau trong cùng một hệ thống:**

| Nhóm API | Envelope |
|---|---|
| `tuku-be.ptallenvery.com` | `{ success, data, meta }` |
| `tkapi*.352722.com/lhc` | `{ _data, version }` → giải mã: `{ status, message, data }` |
| `api.edgoalb.com` | `{ code, msg, data, encoded }` |

#### 4.2. Phân tích chi tiết một API tiêu biểu: `GET /lhc/bbs/views`
| Thành phần | Chi tiết |
|---|---|
| Host | `tkapi1/2/3.352722.com` (client chọn ngẫu nhiên) |
| Method | `GET` |
| Header đáng chú ý | Theo mã client: `Content-Type: application/json`, không gửi `authorization` khi endpoint công khai _(header thực tế chưa ghi lại)_ |
| Tham số | `per_page`, `page` (thường) + `_data` (AES) |
| Plaintext của `_data` | `page`, `category_id`, `isLogo`, `sort`, `title`, `timestamp`, `sign` |
| Response | `{ _data, version }` → `{ status, message, data }` |
| Thành công | HTTP `200`, `status: "success"` |
| Lỗi (suy ra từ mã client) | `400/401/403` xử lý chung; `422` trả lỗi validation theo field |
| Auth | Không yêu cầu token để đọc |

#### 4.3. Endpoint suy ra từ mã nguồn (chưa quan sát request thật)

**Cơ chế gọi API chung** (hàm `apiLHCResquest`):
- URL = `{random(listAPI)}/lhc/` + đường dẫn. Client tự chọn ngẫu nhiên 1 trong 3 host (load-balancing/failover phía client).
- Header: `Content-Type: application/json`, `Cache-Control: max-age`, và `authorization: <token>` (token lưu trong `localStorage`) nếu endpoint cần đăng nhập.
- Body/query: `getSignParams` → thêm `timestamp` (10 số), sắp xếp `k=v`, nối `&`, tính **MD5** làm `sign`, rồi **AES-128-CBC mã hoá toàn bộ** và gửi dưới dạng `{ "_data": "<ciphertext>" }`.
- Response: `{ "_data": "<ciphertext>" }` → client giải mã AES rồi `JSON.parse`. Riêng các endpoint đánh dấu `noCrypt` (host `tuku-be`) trả JSON thường.
- Phân trang: `?per_page=30` (hoặc `limit`/`page`).

**Endpoint REST (`{domain}/lhc/`)** — xác minh bằng đọc mã:

| # | Endpoint | Method | Mục đích | Auth |
|---|---|---|---|---|
| 1 | `pictures?per_page=30` | GET | Danh sách hình (tranh đồ thị) | Công khai |
| 2 | `pictures/{lottery_id}/search` | GET | Tìm hình theo loại xổ số | Cần token |
| 3 | `picture-replies?per_page=30` | GET | Bình luận của hình | Công khai |
| 4 | `picture-replies`, `/like`, `/report` | POST | Đăng, thích, báo cáo bình luận hình | Cần token |
| 5 | `draws?per_page=N`, `draws/{id}`, `draw/lastdraw` | GET | Kết quả mở thưởng / kỳ gần nhất | Công khai |
| 6 | `setting-year/{year}` | GET | Cấu hình theo năm | Công khai |
| 7 | `bbs/views?per_page=N`, `bbs/categories` | GET | Bài viết diễn đàn, danh mục _(đã quan sát, xem 4.1)_ | Công khai |
| 8 | `bbs`, `bbs/{id}/like` | POST | Đăng bài, thích bài | Cần token |
| 9 | `bbs-replies?per_page=30` | GET | Bình luận bài viết | Công khai |
| 10 | `bbs-replies`, `/like`, `/report` | POST | Đăng, thích, báo cáo bình luận | Cần token |
| 11 | `vote/{id}/type/{t}` (GET), `vote` (POST) | GET/POST | Bình chọn | GET công khai, POST cần token |
| 12 | `member` | POST | Đăng ký | Công khai |
| 13 | `member/login` | POST | Đăng nhập (trả token) | Công khai |
| 14 | `member` | GET | Thông tin tài khoản | Cần token |
| 15 | `member/logout` | GET | Đăng xuất | Cần token |
| 16 | `bbsdoing/add` | — | Hành động diễn đàn (không hiện loading) | Cần token (suy luận) |

**Endpoint không mã hoá (`TUKU_BASE = tuku-be.ptallenvery.com/api/public/`):**
`pictures/{type}?sort=site&page=&limit=100`, `pictures/{type}/letters…`, `pictures/detail/{id}`, `videos/{type}?limit=&page=` (type: `hongkong`=1, `macau`=2, `kl8`=4).

**Khác:**
- `GET api.edgoalb.com/sites?…&page=&encode=true` — danh sách site (đã quan sát, xem 4.1).
- `GET year/{year}.json?…` trên `baseJson` — dữ liệu tĩnh theo năm.
- `POST /e` trên `api.bdmijg.com` — tracking.

Realtime (WebSocket / socket.io): client dùng `socket.io 4.7.5` kết nối `wss://ws.edgoapp.com` (và danh sách `kjHosts = ws1–5.servers01.com`) để nhận kết quả mở thưởng trực tiếp. _Tên event và payload: chưa xác minh — cần DevTools → Network → WS._

### 5. Đánh giá thiết kế API
- **Đặt tên & versioning:** RESTful, danh từ số nhiều, tài nguyên lồng nhau hợp lý (`bbs/{id}/like`). **Không có version** trong URL (chỉ có tiền tố `/lhc/`) → khó nâng cấp tương thích ngược. Có một số điểm không nhất quán: `draw/lastdraw` (số ít) và `draws` (số nhiều); `bbsdoing/add` dùng động từ trong URL.
- **Định dạng & lỗi:** hệ thống có **3 envelope response khác nhau** (`{success,data,meta}` ở `tuku-be`; `{_data,version}` → `{status,message,data}` ở `tkapi*/lhc`; `{code,msg,data,encoded}` ở `api.edgoalb.com`) → client phải xử lý từng kiểu; có thể cân nhắc thống nhất một chuẩn. Body bọc trong `_data` (mã hoá) nên không dùng được cache HTTP/CDN theo nội dung, khó debug. Lỗi dựa vào HTTP status (400 sai đăng nhập, 401 hết phiên, 403, 422 validation theo từng field) — hợp lý, nhưng thông điệp lỗi trả về trong body mã hoá.
- **Thiết kế GET:** tham số GET được nhét vào ciphertext `_data` trên query string, đồng thời `per_page`/`page` vẫn lộ dạng thường (trùng lặp). URL dài, ciphertext vào log/proxy, thay đổi theo mỗi lần gọi do `timestamp` nên **không cache được**. Chuẩn hơn: tham số rõ ràng trên query, bảo mật bằng HTTPS và xác thực.
- **Cache:** header `Cache-Control: max-age` gửi ở **request** (không có tác dụng); phía trang vào có `max-age=60, s-maxage=600` (tốt cho trang tĩnh).
- **Phân trang:** hai kiểu: `per_page`/`page` ở `tkapi*` và `limit`/`page` kèm `meta` (`total`, `totalPages`, `hasMore`) ở `tuku-be` — kiểu sau rõ ràng hơn. Cả hai đều là offset; dữ liệu lớn (2.103 ảnh/22 trang) nên cân nhắc cursor.
- **Ký và mã hoá phía client:** nhóm `/lhc` ký (MD5) và mã hoá (AES-CBC) bằng khoá nằm trong mã client; lớp này làm tăng độ phức tạp của request nhưng không thay thế được xác thực phía server (ví dụ chữ ký HMAC với bí mật phía server).
- **Phân quyền:** token trong header `authorization` (lưu `localStorage`); 401 thì tự chuyển về trang đăng nhập.
- **Độ sẵn sàng:** chọn host ngẫu nhiên phía client, không health-check → một host chết là một phần request lỗi.

### 6. Chiến lược kiểm thử API

> Chỉ thực hiện trong phạm vi được phép (API công khai, dữ liệu đọc). Đây là kế hoạch kiểm thử đề xuất dựa trên các API đã quan sát ở mục 4.1.

**6.1. Ưu tiên theo rủi ro**

| Ưu tiên | API | Lý do |
|---|---|---|
| Cao | `tuku-be …/pictures/{type}` | Dữ liệu cốt lõi, lượng lớn (2.103 mục/22 trang), có phân trang |
| Cao | `tkapi*/lhc/bbs/views`, `bbs/categories` | Có ký + mã hoá, nhiều tham số lọc, dễ lỗi khi sai chữ ký/timestamp |
| Trung bình | `api.edgoalb.com/sites` | Định dạng response khác, dữ liệu mã hoá |
| Thấp | Các API ghi (`bbs`, `vote`, `member`) | Cần tài khoản/token, chỉ kiểm thử trên môi trường được cấp |

**6.2. Kiểm thử chức năng (Functional)**
- Mỗi API trả `200` và đúng envelope (`{success,data,meta}`, `{_data,version}` → `{status,message,data}`, `{code,msg,data,encoded}`).
- Kiểm tra schema và kiểu dữ liệu: `id` kiểu chuỗi hay số, `latest.year/issue` kiểu số, URL ảnh hợp lệ và truy cập được.
- Phân trang: `page=1..totalPages`, `limit` thay đổi, `meta.total`/`totalPages`/`hasMore` nhất quán (ví dụ 2.103 mục, `limit=100` → 22 trang, trang cuối `hasMore=false`).
- Lọc và sắp xếp: `sort=site`, `category_id`, `title`, `type=hklhc|am2lhc`.
- Giải mã `_data` ra đúng cấu trúc mong đợi; dữ liệu danh mục diễn đàn khớp với giao diện.

**6.3. Kiểm thử tiêu cực (Negative)**
- Thiếu hoặc sai tham số: bỏ `page`, `page=0`, `page=-1`, `page=abc`, `limit=0`, `limit` rất lớn.
- Giá trị ngoài phạm vi: `page` vượt `totalPages`, `type` không tồn tại, `lottery` không hợp lệ.
- `_data` sai: thiếu, sai base64, sai chữ ký (`sign`), `timestamp` quá cũ hoặc ở tương lai → kỳ vọng lỗi rõ ràng (`400/422`), không lỗi `500`.
- HTTP method không hỗ trợ (POST vào endpoint GET) → `405`.
- Xác minh thông báo lỗi nhất quán, không lộ thông tin nội bộ.

**6.4. Kiểm thử bảo mật (trong phạm vi được phép)**
- Xác thực/phân quyền: API đọc công khai hoạt động không cần token; API ghi bị từ chối khi thiếu token (`401`).
- Response không chứa thông tin nhạy cảm hoặc thông tin môi trường nội bộ (ví dụ trường `fetchUrlTemplate` trỏ `localhost`).
- Chống replay: gửi lại cùng `_data` sau khi quá hạn `timestamp` → kỳ vọng bị từ chối.
- Kiểm tra giới hạn tần suất (rate limiting) ở mức nhẹ, không gây tải.

**6.5. Kiểm thử hiệu năng**
- Đo thời gian phản hồi (p50/p95) cho từng host `tkapi1/2/3` để kiểm tra chênh lệch.
- Kích thước response theo `limit` (ví dụ 30 so với 100) và hiệu quả nén `gzip`.
- Hành vi khi một host không phản hồi (timeout, lỗi một phần).
- Cache: kiểm tra `Cache-Control`/`ETag` của response API và CDN ảnh (`?v=` cache-busting).
- Mọi kiểm thử tải chỉ làm khi được cho phép.

**6.6. Công cụ đề xuất**
DevTools Network (quan sát), Postman/Newman (kịch bản và kiểm tra schema), k6 hoặc JMeter (hiệu năng, khi được phép).

### 7. Kết luận

- **Các API đã xác định:** 3 nhóm đã quan sát bằng DevTools — API hình/video (`tuku-be`, JSON thường), API nghiệp vụ diễn đàn (`tkapi*/lhc`, ký MD5 + mã hoá AES) và API danh sách site (`api.edgoalb.com`, response mã hoá). Trang chủ chỉ là điểm vào (Document); API chỉ xuất hiện khi người dùng thao tác ở các trang con.
- **Luồng chính:** người dùng bấm nút → tải chunk của route → ứng dụng gọi API tương ứng (qua một lớp gọi API chung) → giải mã và hiển thị.
- **Ưu tiên kiểm thử:** phân trang và tính nhất quán dữ liệu của API hình; tính đúng của chữ ký/timestamp ở nhóm `/lhc`; xử lý lỗi nhất quán giữa 3 định dạng response.
- **Giới hạn của phân tích:**
  - Chỉ dựa trên quan sát DevTools và đọc mã client công khai; không truy cập hệ thống phía server.
  - Một số trang/nút chưa khảo sát; các endpoint chưa gặp được chỉ suy ra từ mã nguồn.
  - Kết nối WebSocket và thuật toán giải mã của `api.edgoalb.com` chưa xác minh.
  - Chưa có số liệu hiệu năng thực đo.
- **Cần thêm để phân tích đầy đủ:** tài liệu API (OpenAPI/Swagger), tài khoản kiểm thử, môi trường test, và xác nhận phạm vi được phép cho kiểm thử bảo mật/hiệu năng.

