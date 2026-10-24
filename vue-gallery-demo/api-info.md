# Tổng hợp API quan sát bằng DevTools

> Ghi chú: các giá trị nhạy cảm (ciphertext `_data`, khoá tracking, dữ liệu mã hoá) đã được rút gọn thành dạng `abc...`. Nội dung ảnh thật (id, tên, tên file, URL ảnh) trong response mẫu API 1 được thay bằng dữ liệu mock (`sample`, `cdn.example.com`); cấu trúc trường giữ nguyên.

| Trang / nút | Route | Loại | API |
|---|---|---|---|
| Trang chủ | `#/pages/index/index` | Doc | `GET /?800tk` |
| Trang chủ | `#/pages/index/index` | Fetch/XHR | `GET tuku-be.ptallenvery.com/api/public/pictures/macau` |
| 发现 | `#/pages/forum/find` | Fetch/XHR | `GET tkapi3.352722.com/lhc/bbs/categories`, `GET …/lhc/bbs/views` |
| 发布 | `#/pages/login/login` | — | Không có API (form đăng nhập tĩnh) |
| 好站 | `#/pages/website/index` | Fetch/XHR | `GET api.edgoalb.com/sites` (3 lần gọi) |
| 会员登陆 | `#/pages/login/login` | — | Không có API (form đăng nhập tĩnh) |

---

# 1. Trang chủ (Index)

- **Route:** `https://43.213.218.98:8000/?800tk#/pages/index/index`

## 1.1. Fetch/XHR

### API 1 — Danh sách hình (Macau)

| Mục | Giá trị |
|---|---|
| Method | `GET` |
| URL | `https://tuku-be.ptallenvery.com/api/public/pictures/macau` |
| Query | `sort=site`, `page=1`, `limit=100` |
| Status | `200` |
| Mã hoá | Không (JSON thường) |

Response:

```json
{
    "success": true,
    "data": [
        {
            "id": "1001",
            "lottery": "sample",
            "category": "col",
            "filename": "sample.jpg",
            "name": "Sample picture",
            "keyword": "sample",
            "letter": "S",
            "sortOrder": 10,
            "latest": {
                "year": 2026,
                "issue": 100,
                "url": "https://cdn.example.com/images/sample/2026/col/100/sample.jpg?v=1700000000",
                "thumbnailUrl": "https://cdn.example.com/images/thumbs/sample/2026/col/100/sample.jpg?v=1700000000",
                "originalUrl": "https://cdn.example.com/images/sample/O/2026/col/100/sample.jpg?v=1700000000",
                "width": 1080,
                "height": 1659
            },
            "urlTemplate": "https://cdn.example.com/images/sample/{year}/col/{issue}/sample.jpg",
            "thumbnailUrlTemplate": "https://cdn.example.com/images/thumbs/sample/{year}/col/{issue}/sample.jpg",
            "fetchUrlTemplate": "http://localhost/api/images/sample/{year}/col/{issue}/sample.jpg"
        },
        ...
    ],
    "meta": {
        "page": 1,
        "perPage": 100,
        "total": 2103,
        "totalPages": 22,
        "hasMore": true,
        "site": null
    }
}
```

Giải thích các trường chính:

| Trường | Ý nghĩa |
|---|---|
| `data[].id`, `lottery`, `category` | Mã hình, loại (ví dụ mock `sample`), danh mục (`col`) |
| `data[].name`, `keyword`, `letter` | Tên hiển thị, từ khoá, chữ cái dùng để lọc A–Z |
| `data[].latest` | Kỳ mới nhất: `year`, `issue`, URL ảnh/thumbnail/ảnh gốc, kích thước |
| `data[].*Template` | URL mẫu với `{year}` và `{issue}` để suy ra ảnh các kỳ khác |
| `meta` | Phân trang: `page`, `perPage`, `total` (2103), `totalPages` (22), `hasMore` |

Lưu ý: `fetchUrlTemplate` trỏ `http://localhost/...` (không dùng được từ bên ngoài).

## 1.2. Doc

### Request điều hướng — trang HTML

| Mục | Giá trị |
|---|---|
| Method | `GET` |
| URL | `https://43.213.218.98:8000/?800tk` |
| Query | `800tk` |
| Status | `200` |
| Content-Type | `text/html; charset=UTF-8` |

Response (HTML khung rỗng, chỉ có thẻ `<script>`/`<link>`):

```html
<!doctype html>
<html lang="zh-cn" translate="no">
    <head>
        <link rel="icon" href="data:,">
        <script src="https://api.bdmijg.com/assets/js/runtime.js?v=0.33d" referrerpolicy="no-referrer"></script>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <script>
            window.__cmsTrack = {
                "endpoint": "/e",
                "site": 1,
                "k": "abc..."
            };
            window.__cmsTrackLoaded = 1;
        </script>
        <script src="https://api.bdmijg.com/assets/js/rc.js?v=0.33d" async referrerpolicy="no-referrer"></script>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <meta name="google" content="notranslate">
        <meta name="format-detection" content="telephone=no"/>
        <title>加载中-富强、民主、文明、和谐、自由、平等、公正、法治、爱国、敬业、诚信、友善。</title>
        <script src="https://libs.baidu.com/jquery/1.11.1/jquery.min.js"></script>
        <script src="https://api.bdmijg.com/assets/js/pako.min.js" referrerpolicy="no-referrer"></script>
        <link rel="stylesheet" href="https://api.bdmijg.com/assets/css/style.css?v=0.33d" referrerpolicy="no-referrer">
    </head>
    <body class="bg-light"></body>
    <script src="https://api.bdmijg.com/assets/js/AnimalsHelper.js" referrerpolicy="no-referrer"></script>
    <script src="https://api.bdmijg.com/com.js?v=0.33d" referrerpolicy="no-referrer"></script>
    <script src="https://api.bdmijg.com/js-1-index?v=10243" referrerpolicy="no-referrer"></script>
    <script></script>
</html>
```

Nhận xét: body rỗng (`<body class="bg-light"></body>`); nội dung thật do `js-1-index` giải nén và ghi vào trang.

---

# 2. Trang 发现 (Khám phá)

- **Điều hướng:** Trang chủ → bấm `发现`
- **Route:** `https://43.213.218.98:8000/?800tk#/pages/forum/find`

## 2.1. Fetch/XHR

### API 2 — Danh mục diễn đàn

| Mục | Giá trị |
|---|---|
| Method | `GET` |
| URL | `https://tkapi3.352722.com/lhc/bbs/categories` |
| Query | `_data=abc...` (ciphertext AES, base64, URL-encode) |
| Status | `200` |
| Mã hoá | Có (request và response) |

Request (rút gọn):

```
GET /lhc/bbs/categories?_data=abc...
```

Response:

```json
{
    "_data": "abc...",
    "version": "1.0.0"
}
```

### API 3 — Danh sách bài diễn đàn

| Mục | Giá trị |
|---|---|
| Method | `GET` |
| URL | `https://tkapi3.352722.com/lhc/bbs/views` |
| Query | `per_page=10`, `page=1`, `_data=abc...` |
| Status | `200` |
| Mã hoá | Có (request và response) |

Request (rút gọn):

```
GET /lhc/bbs/views?per_page=10&page=1&_data=abc...
```

Response:

```json
{
    "_data": "abc...",
    "version": "1.0.0"
}
```

---

# 3. Trang 发布 (Đăng bài)

- **Điều hướng:** Trang chủ → bấm `发布`
- **Route:** `https://43.213.218.98:8000/?800tk#/pages/login/login`
- **API:** không có
- **Ghi chú:** chỉ hiển thị form đăng nhập tĩnh để người dùng đăng nhập.

---

# 4. Trang 好站 (Trang tốt)

- **Điều hướng:** Trang chủ → bấm `好站`
- **Route:** `https://43.213.218.98:8000/?800tk#/pages/website/index`

## 4.1. Fetch/XHR

Cả ba API dùng chung host `https://api.edgoalb.com/sites`, `Method: GET`, `Status: 200`, response mã hoá (`encode=true`).

| # | URL | Query | Ý nghĩa tham số |
|---|---|---|---|
| API 4 | `/sites` | `type=hklhc`, `page=1`, `encode=true` | `type=hklhc`: nhóm Hong Kong |
| API 5 | `/sites` | `type=am2lhc`, `page=1`, `encode=true` | `type=am2lhc`: nhóm Macau |
| API 6 | `/sites` | `category_id=h100`, `page=1`, `encode=true` | Lọc theo danh mục |

Response mẫu (API 4):

```json
{
    "code": 1,
    "msg": "",
    "data": "abc...",
    "encoded": true
}
```

Response API 5 và API 6 có cùng cấu trúc, chỉ khác nội dung `data` (`abc...`).

---

# 5. Trang 会员登陆 (Đăng nhập thành viên)

- **Điều hướng:** Trang chủ → bấm `会员登陆`
- **Route:** `https://43.213.218.98:8000/?800tk#/pages/login/login`
- **API:** không có
- **Ghi chú:** chỉ hiển thị form đăng nhập tĩnh.
