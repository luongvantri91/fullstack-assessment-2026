SPA: Kiến trúc ứng dụng web một trang.

uni-app: Framework hỗ trợ phát triển ứng dụng đa nền tảng, bao gồm H5 chạy trên trình duyệt.

Vue 2: Framework JavaScript dùng để xây dựng giao diện và quản lý trạng thái.

webpack: Công cụ đóng gói JavaScript, CSS và các tài nguyên cần thiết cho ứng dụng.

Build H5: Biên dịch ứng dụng thành phiên bản HTML5 để chạy trong trình duyệt web.

## Loại request trong DevTools

Doc (Document): Request điều hướng của trình duyệt để tải một trang HTML (gõ URL, bấm link, F5). Trình duyệt tự gửi, không phải JavaScript gọi. Header có `sec-fetch-dest: document`, `sec-fetch-mode: navigate`.

Fetch/XHR: Request do JavaScript chủ động gửi bằng `fetch()` hoặc `XMLHttpRequest`, thường để lấy dữ liệu (JSON) từ API mà không tải lại trang. Đây là nơi tìm API của website.

WS (WebSocket): Kết nối hai chiều, giữ mở lâu giữa trình duyệt và server, dùng cho dữ liệu realtime.

JS / CSS / Img: Tài nguyên tĩnh của trang (script, style, ảnh), không phải API.

## Kiến trúc & tải trang

Loader: Đoạn mã nhỏ có nhiệm vụ nạp và dựng phần còn lại của trang. Trang chủ ở đây chỉ chứa HTML khung rỗng cộng các thẻ script.

document.write: Hàm ghi trực tiếp HTML vào trang. Bị khuyến cáo tránh vì chặn render và dễ gây lỗi bảo mật.

pako: Thư viện JavaScript nén/giải nén dữ liệu (zlib/deflate) ở trình duyệt.

deflate: Thuật toán nén dữ liệu không mất mát (dùng trong zip, gzip).

gzip: Định dạng nén dùng để giảm dung lượng response HTTP (`content-encoding: gzip`).

Base64: Cách mã hoá dữ liệu nhị phân thành chuỗi ký tự ASCII để nhúng vào văn bản. Đây là mã hoá biểu diễn, không phải bảo mật.

Transfer-Encoding: chunked: Server gửi response theo từng khối nhỏ, không cần biết trước tổng dung lượng.

Hash router: Cách định tuyến SPA dùng phần sau dấu `#` trong URL (ví dụ `#/pages/index/index`). Phần này không gửi lên server.

Lazy-load / chunk: Chia mã JavaScript thành nhiều file nhỏ (chunk), chỉ tải khi người dùng vào trang cần dùng.

CDN (Content Delivery Network): Mạng máy chủ phân tán lưu bản sao tài nguyên tĩnh, giúp tải nhanh và giảm tải cho server gốc.

Origin server: Máy chủ gốc chứa nội dung thật, nằm sau CDN hoặc cache.

nginx: Phần mềm web server / reverse proxy / load balancer hiệu năng cao.

Reverse proxy: Máy chủ đứng trước server thật, nhận request từ client rồi chuyển tiếp và trả kết quả.

## Cache

Cache-Control: Header quy định cách cache response. `max-age=60` nghĩa là cache trình duyệt/proxy dùng được trong 60 giây; `s-maxage=600` áp dụng cho cache dùng chung (CDN/proxy) là 600 giây; `public` cho phép mọi cache lưu.

x-cache-status: Header tuỳ chỉnh của nginx cho biết trạng thái cache: `MISS` (không có trong cache, lấy từ origin), `HIT` (lấy từ cache), `STALE` (phục vụ bản cũ đã hết hạn).

ETag: Mã định danh phiên bản của response, giúp client hỏi "có thay đổi không?" và nhận `304 Not Modified` nếu không đổi.

Vary: Header cho biết cache phải tách bản lưu theo header nào (ví dụ `Accept-Encoding`).

## API & giao thức

API (Application Programming Interface): Giao diện để chương trình này gọi chức năng/dữ liệu của chương trình khác.

REST / RESTful: Phong cách thiết kế API dùng HTTP method (GET, POST...) trên tài nguyên được đặt tên bằng URL (ví dụ `bbs/{id}/like`).

Endpoint: Một đường dẫn URL cụ thể của API.

HTTP method: GET (lấy dữ liệu), POST (tạo/gửi dữ liệu), PUT/PATCH (cập nhật), DELETE (xoá).

HTTP status code: Mã kết quả: 200 thành công, 400 request sai, 401 chưa xác thực, 403 bị từ chối, 422 dữ liệu không hợp lệ.

JSON: Định dạng dữ liệu văn bản dạng key-value, phổ biến cho API.

Payload: Phần dữ liệu thực sự được gửi/nhận trong request hoặc response.

Pagination: Chia danh sách lớn thành nhiều trang. `per_page`/`page` là kiểu offset; cursor pagination dùng con trỏ vị trí, ổn định hơn với dữ liệu lớn.

Versioning (`/api/v1/`): Đánh số phiên bản API để nâng cấp mà không phá client cũ.

OpenAPI: Chuẩn mô tả API bằng file (swagger), dùng để sinh tài liệu và test.

Laravel: Framework PHP phổ biến; API kiểu `per_page`, mã lỗi 422 validation là đặc trưng của nó.

Rate limiting: Giới hạn số request trong một khoảng thời gian để chống lạm dụng.

Load balancing: Chia request giữa nhiều server. Ở đây client tự chọn ngẫu nhiên một host (client-side), khác với dùng bộ cân bằng tải ở server.

Failover / health-check: Tự chuyển sang server dự phòng khi server chính lỗi; health-check là kiểm tra định kỳ server còn sống không.

WebSocket: Giao thức kết nối hai chiều lâu dài giữa client và server, dùng cho dữ liệu realtime.

socket.io: Thư viện WebSocket có thêm tính năng tự kết nối lại và "event" (sự kiện có tên).

Polling: Client gọi API định kỳ để kiểm tra dữ liệu mới (đối lập với server đẩy qua WebSocket).

## Xác thực & mã hoá

Authentication / Authorization: Xác thực là "bạn là ai"; phân quyền là "bạn được làm gì".

Token: Chuỗi do server cấp sau khi đăng nhập, client gửi kèm (header `authorization`) để chứng minh danh tính.

JWT (JSON Web Token): Token có cấu trúc, tự chứa thông tin và chữ ký, thường có hạn ngắn.

localStorage: Kho lưu trữ khoá-giá trị trong trình duyệt. JavaScript đọc được nên token lưu ở đây dễ bị lấy khi có XSS.

Cookie HttpOnly: Cookie mà JavaScript không đọc được, an toàn hơn localStorage cho token.

MD5: Hàm băm cho ra chuỗi cố định từ dữ liệu vào. Không có khoá bí mật thì không dùng làm chữ ký chống giả mạo được; MD5 cũng đã lỗi thời.

HMAC: Chữ ký dựa trên hàm băm kèm khoá bí mật, chỉ bên có khoá mới tạo/kiểm tra được.

AES-128-CBC: Thuật toán mã hoá đối xứng (cùng một khoá để mã hoá và giải mã), khoá 128 bit, chế độ CBC.

Key / IV (Initialization Vector): Khoá bí mật và giá trị khởi tạo ngẫu nhiên của AES. Nếu hard-code trong JS client thì ai cũng đọc được nên mã hoá mất ý nghĩa.

Hard-code: Ghi cứng giá trị (khoá, mật khẩu) trực tiếp trong mã nguồn.

Obfuscation (che giấu): Làm mã/dữ liệu khó đọc nhưng không thực sự bảo mật.

Replay attack: Kẻ tấn công gửi lại một request hợp lệ đã bắt được. Chống bằng timestamp/nonce kiểm tra phía server.

timestamp / sign: Thời gian gửi request và chữ ký của các tham số, dùng để xác thực và chống replay.

noCrypt: Cờ trong code client đánh dấu endpoint dùng JSON thường, không ký/mã hoá.

## Bảo mật & quyền riêng tư

XSS (Cross-Site Scripting): Kẻ tấn công chèn mã JavaScript vào trang để chạy trên máy nạn nhân (đánh cắp token, v.v.).

CSP (Content Security Policy): Header giới hạn nguồn script/tài nguyên mà trang được tải, giảm rủi ro XSS.

SRI (Subresource Integrity): Thuộc tính `integrity` giúp trình duyệt kiểm tra file bên thứ ba có bị sửa không.

Supply chain risk (rủi ro chuỗi cung ứng): Rủi ro khi trang phụ thuộc script của bên thứ ba; nếu họ bị chiếm quyền thì trang của bạn cũng bị ảnh hưởng.

Bot detection: Mã kiểm tra trình duyệt có phải công cụ tự động không (`navigator.webdriver`, headless, Playwright, Cypress).

Headless browser: Trình duyệt chạy không giao diện, dùng cho tự động hoá/crawl.

Cloaking: Hiển thị nội dung khác nhau cho người dùng và cho bot/kiểm duyệt để che giấu nội dung thật.

Tracking / analytics: Thu thập hành vi người dùng. Baidu Hm (`hm.baidu.com`) là dịch vụ thống kê của Baidu.

Consent (đồng ý): Cơ chế xin phép người dùng trước khi theo dõi, bắt buộc theo nhiều luật bảo mật dữ liệu.

APK: File cài đặt ứng dụng Android. Cài ngoài Google Play tiềm ẩn rủi ro mã độc.

.mobileconfig: File cấu hình (profile) iOS, có thể cài chứng chỉ, proxy, VPN. Phân phối ngoài App Store là vector tấn công quen thuộc.

Fallback domain (tên miền dự phòng): Tên miền thay thế dùng khi tên miền chính bị chặn.

## Khác

Mock API: API giả trả dữ liệu mẫu để phát triển/test mà không cần server thật.

Passive analysis (phân tích thụ động): Chỉ quan sát và đọc thông tin công khai, không gửi request tấn công hay can thiệp hệ thống.

Static analysis (phân tích tĩnh): Đọc mã nguồn mà không chạy chương trình.

