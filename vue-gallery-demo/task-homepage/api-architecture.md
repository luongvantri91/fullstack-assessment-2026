# So sánh kiến trúc API: trang mẫu và bản Vue 3

Tài liệu này so sánh kiến trúc API của trang mẫu (đã phân tích ở Nhiệm vụ 1) với bản dựng lại trong thư mục này.

## 1. Tóm tắt

Bản Vue 3 chỉ áp dụng **một phần nhỏ** kiến trúc của trang mẫu, cụ thể là kiểu API ảnh nhóm B (`tuku-be`). Các phần còn lại được bỏ có chủ ý vì bài chỉ yêu cầu dựng giao diện.

- **Đã áp dụng:** lớp gọi API tách khỏi giao diện, envelope `{success, data, meta}`, phân trang theo `page` + `meta.hasMore`, hash router, xử lý lỗi tập trung.
- **Không áp dụng:** ký/mã hoá request, token, nhiều host, CDN ảnh, cấu hình động, WebSocket, chống bot và tracking.

## 2. Kiến trúc API của trang demo

### 2.1. Sơ đồ kiến trúc

```
┌──────────────────────────────────────────────────────────────┐
│ Trình duyệt: SPA Vue 3 (nạp từ CDN, không cần build)         │
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌─────────────────┐   │
│  │ Giao diện    │   │ Composable   │   │ Lớp API         │   │
│  │ (component)  │──►│ useGallery   │──►│ GalleryApi      │   │
│  │ Header, Grid,│   │ trạng thái,  │   │ fetchItems()    │   │
│  │ Card, Nav…   │◄──│ phân trang   │◄──│ cache theo      │   │
│  └──────────────┘   └──────────────┘   │ danh mục        │   │
│                                        └───────┬─────────┘   │
└────────────────────────────────────────────────┼─────────────┘
                                                 │ ① danh sách (JSON)
                             ┌───────────────────┴──────────────┐
                             ▼                                  ▼
                  ┌─────────────────────┐           ┌────────────────────┐
                  │ picsum.photos       │           │ Dữ liệu giả        │
                  │ GET /v2/list        │           │ (dự phòng khi lỗi  │
                  │ nguồn chính         │           │ mạng/timeout)      │
                  └─────────────────────┘           └────────────────────┘

Trình duyệt ──② tải từng ảnh (lazy)──► picsum.photos/id/{id}/{w}/{h}
                                        │ lỗi → thẻ <img> chuyển sang ảnh SVG sinh tại chỗ
```

### 2.2. Sơ đồ luồng dữ liệu

```
Người dùng mở trang / đổi danh mục / gõ tìm kiếm / cuộn tới cuối
   │
   ▼
useGallery.loadNext()  ── chặn nếu đang tải hoặc hasMore = false
   │  gọi fetchItems({ category, page, query })
   ▼
GalleryApi.getCatalog(category)
   ├─ đã có cache ─────────────► dùng lại
   └─ chưa có ─► GET /v2/list?page={cửa sổ}&limit=50 (timeout 8 giây)
                   ├─ thành công ─► map thành { id, title, ratio, hue, imageUrl }
                   └─ lỗi/timeout ─► dùng dữ liệu giả
   │
   ▼
Lọc theo query → cắt theo trang (10 mục)
   │
   ▼
Trả về  { success, data[10], meta{ page, perPage, total, totalPages, hasMore } }
   │
   ▼
useGallery: thêm vào cột ngắn hơn (masonry), cập nhật hasMore
   │
   ▼
Vue render thẻ ảnh ──► trình duyệt tải ảnh ──► lỗi? ──► ảnh SVG dự phòng
   │
   ▼
IntersectionObserver theo dõi phần tử cuối danh sách ──► tải trang kế tiếp
```

### 2.3. Vai trò từng lớp

| Lớp | Thành phần | Trách nhiệm |
|---|---|---|
| Giao diện | `AppHeader`, `QuickMenu`, `MasonryGrid`, `GalleryCard`, `CategoryPills`, `BottomNav` | Hiển thị, phát sự kiện (chọn danh mục, tìm kiếm); không gọi API trực tiếp |
| Trạng thái | `useGallery` | Giữ danh mục, từ khoá, các cột, `page`, `hasMore`, `loading`; chống kết quả cũ khi đổi danh mục (`requestId`) |
| Truy cập dữ liệu | `GalleryApi.fetchItems` | Điểm vào duy nhất cho dữ liệu; cache, lọc, phân trang, đóng gói envelope |
| Nguồn dữ liệu | `picsum.photos` / dữ liệu giả | Cung cấp danh sách ảnh và ảnh |
| Điều hướng | `useHashRoute` | Đọc `#/home`, `#/discover`… từ URL |

Nhờ có lớp truy cập dữ liệu riêng, việc đổi nguồn (Picsum → API thật) chỉ ảnh hưởng một hàm.

### 2.4. Mô tả nghiệp vụ các hàm chính

Mỗi hàm đều có chú thích nghiệp vụ ngay trên mã nguồn ([api.js](./api.js), [app.js](./app.js)). Bảng tóm tắt:

**`api.js` – lớp dữ liệu**

| Hàm | Nghiệp vụ |
|---|---|
| `fetchItems({category, page, query})` | Điểm vào duy nhất của UI: lấy danh sách danh mục → lọc theo từ khoá → cắt 10 mục/trang → trả `{success, data, meta}` |
| `getCatalog(category)` | Trả danh sách của một danh mục, chỉ tải một lần (cache); lỗi thì dùng dữ liệu dự phòng |
| `loadRemoteCatalog(category)` | Gọi Picsum lấy danh sách ảnh của danh mục, timeout 8 giây |
| `buildFallbackCatalog(categoryId)` | Tạo 50 mục cục bộ khi không có mạng; thẻ sẽ vẽ ảnh SVG thay cho ảnh thật |
| `placeholderImage(item)` | Sinh ảnh SVG nền gradient để thẻ không bao giờ trống |
| `ratioFor`, `hueFor`, `seededRandom` | Sinh tỉ lệ khung, màu và số ngẫu nhiên cố định từ id, để bố cục ổn định giữa các lần tải |

**`app.js` – giao diện và trạng thái**

| Thành phần | Nghiệp vụ |
|---|---|
| `useGallery` | Logic cốt lõi "hiển thị ảnh theo trang": `loadNext()` nối trang kế tiếp vào cột ngắn hơn, `reset()` tải lại từ trang 1, bỏ qua phản hồi cũ |
| `useHashRoute` | Điều hướng theo `#/…` không cần thư viện |
| `useToast` | Thông báo ngắn tự tắt sau 1,8 giây |
| `AppHeader` | Tab danh mục, ô tìm kiếm, nút tải/chia sẻ |
| `GalleryCard` | Một ảnh và tiêu đề; ảnh lỗi thì chuyển sang SVG |
| `MasonryGrid` | Hai cột thẻ, có khối chờ (skeleton) khi đang tải |
| `QuickMenu`, `CategoryPills`, `BottomNav` | Lối tắt, chuyển danh mục nhanh, thanh điều hướng dưới |
| `App` | Ghép các phần: cuộn vô hạn bằng `IntersectionObserver`, đổi danh mục, tìm kiếm có trì hoãn 300 ms, chia sẻ liên kết, thử lại khi lỗi |

## 3. Bảng so sánh với trang mẫu

| Khía cạnh | Trang mẫu (đã phân tích) | Bản Vue 3 | Kế thừa? |
|---|---|---|---|
| Mô hình tổng thể | SPA gọi nhiều backend qua REST | SPA gọi một lớp API duy nhất (`GalleryApi`) | Một phần: giữ ý tưởng tách lớp gọi API khỏi giao diện |
| Lớp gọi API chung | `apiLHCResquest` (một hàm gọi chung cho nhóm `/lhc`) | `fetchItems()` (một hàm duy nhất) | Có: một điểm vào cho dữ liệu |
| Số nhóm API | 3 nhóm (`/lhc`, `tuku-be`, `edgoalb`) | 1 nhóm | Không: không cần cho bài |
| Envelope response | 3 kiểu khác nhau | Một kiểu `{success, data, meta}` | Có: lấy kiểu của nhóm B vì rõ ràng nhất |
| Phân trang | `limit`, `page` + `meta` (`total`, `totalPages`, `hasMore`) | `page`, `PER_PAGE = 10` + `meta` đầy đủ các trường đó | Có: giữ nguyên cấu trúc `meta` |
| Cờ `hasMore` | Có | Có, dùng để dừng tải khi cuộn | Có |
| Lọc / tìm kiếm | `category_id`, `title`, `type` | `category`, `query` | Có (đơn giản hoá) |
| URL ảnh | CDN riêng, có `urlTemplate`, `?v=` cache-busting | `picsum.photos/id/{id}/{w}/{h}` (Lorem Picsum), SVG dự phòng khi ảnh lỗi | Một phần: cùng ý tưởng URL ảnh theo mẫu, khác nguồn |
| Host API | Chọn ngẫu nhiên giữa 3 host phía client | Một dịch vụ công khai `picsum.photos` (`/v2/list`), có dữ liệu giả dự phòng khi mất mạng | Không |
| Ký MD5 + mã hoá AES (`_data`, `sign`, `timestamp`) | Có (nhóm `/lhc`) | Không | Không: đặc thù, khoá nằm trong client |
| Token / đăng nhập | `authorization` từ `localStorage` | Không có đăng nhập | Không |
| Xử lý lỗi | Theo HTTP status (400/401/403/422), suy ra từ mã client | Lỗi tải danh sách tự chuyển sang dữ liệu dự phòng; lỗi bất ngờ khác hiện "Tap to retry" | Một phần: xử lý lỗi tập trung |
| Cấu hình động | `js-b-tkconfig` (host, CDN, banner) | Hằng số trong `api.js` | Không |
| WebSocket realtime | Có trong cấu hình (`socket.io`, chưa xác minh) | Không | Không |
| Chống bot, tracking | `runtime.js`, `rc.js` | Không | Không |
| Router | Hash router `#/pages/...` | Hash router `#/home`... | Có |
| Cách tải trang | Loader nén + bundle + chunk theo route | Một file JS, không có chunk | Không |

## 4. Nguồn dữ liệu và cách chuyển sang API thật

Ảnh lấy từ dịch vụ công khai [Lorem Picsum](https://picsum.photos), miễn phí, không cần API key và cho phép gọi từ trình duyệt (CORS `*`).

| Thành phần | Chi tiết |
|---|---|
| Danh sách | `GET https://picsum.photos/v2/list?page={n}&limit=50` trả `id`, `author`, `width`, `height`, `url`, `download_url` |
| Ảnh hiển thị | `https://picsum.photos/id/{id}/400/{height}` (đã resize, cắt theo tỉ lệ thẻ) |
| Danh mục | Picsum không có danh mục. Mỗi danh mục (Nature, City, Travel) lấy một cửa sổ khác nhau của danh sách (`page` 1, 2, 3) |
| Phân trang và tìm kiếm | Lấy 50 ảnh/danh mục một lần (có cache), rồi phân trang 10 ảnh/trang và lọc theo tiêu đề (dạng "tên tác giả #id") ở phía client |
| Envelope | `fetchItems` đóng gói kết quả thành `{success, data, meta}` như API ảnh nhóm B |
| Khi lỗi mạng | Dùng dữ liệu giả và ảnh SVG sinh tại chỗ; một ảnh lỗi riêng lẻ cũng chuyển sang SVG |

Khi có API thật, chỉ cần thay thân `fetchItems` trong [api.js](./api.js) bằng một lời gọi `fetch()`. Phần giao diện trong [app.js](./app.js) không phải đổi.

```
GET /pictures/{category}?page=1&limit=10&q=...
→ { success, data: [...], meta: { page, perPage, total, totalPages, hasMore } }
```

Lưu ý: danh sách chỉ là ảnh minh hoạ, không đúng nghĩa "Nature/City/Travel"; tên danh mục chỉ để minh hoạ giao diện.

