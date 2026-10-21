# SESSION HANDOFF — làm tiếp bài quiz tuyển dụng

> File riêng tư, KHÔNG commit, KHÔNG gửi cho NTD. Dùng để mở lại phiên làm việc với AI sau khi máy bị reset.
> Cập nhật lần cuối: 2026-10-20 15:16 (cập nhật lại cuối mỗi phiên).

## 1. Vai trò của AI (bắt buộc giữ)

- AI là **người cố vấn kiêm người phỏng vấn**. Chấm câu trả lời theo đúng "Tín hiệu tốt / Tín hiệu yếu" trong đề, chỉ ra điểm yếu, gợi ý cách sửa.
- **Không viết hộ nội dung cá nhân** (ký ức, trải nghiệm). Chỉ sửa chính tả, ngữ pháp, cấu trúc khi tôi yêu cầu, và không thêm chi tiết tôi không nói.
- Trả lời bằng **tiếng Việt**, ngắn gọn nhưng kỹ. Thẳng thắn, khách quan.
- Hỏi nối tiếp như NTD sẽ hỏi.
- Tôi dùng tiếng Việt cho các câu trả lời. README tiếng Anh đơn giản.

## 2. Bối cảnh

- **Vị trí:** Full-Stack Software Engineer (Papaya level: Staff). Yêu cầu: 6+ năm, TypeScript cả FE+BE, dùng LLM hỗ trợ code, giao tiếp/viết tốt. Ưu tiên: kiến trúc bài toán mơ hồ, technical leadership, dùng Claude Code/Codex/Cursor.
- **Tôi:** ~9 năm kinh nghiệm. Go, PHP, Vue, DDD, microservices. Project Owner ở Teecom, dùng Copilot/Claude khoảng 90% công việc.
- **Đánh giá độ khớp (AI đã nhận xét):** phù hợp mức Senior, Staff còn là bước nhảy.
  - Điểm mạnh: số năm, DDD/kiến trúc, quy trình kiểm soát output AI (spec → review → validate), ownership.
  - Điểm yếu cần bù: **TypeScript** (CV gần như không có), **tiếng Anh viết/nói** (CV ghi Basic), chưa có sản phẩm tự xây bằng LLM, chưa có AWS serverless/GraphQL, mốc thời gian Teecom chồng với GMO.
  - Kết luận: **nên thử**, vì challenge chấm đúng điểm mạnh (design, judgment, kiểm soát AI) và tạo portfolio TypeScript.

## 3. Cấu trúc bài test

1. **Câu hỏi tư duy (logical):** bắt buộc **Q1, Q5**. Tôi chọn thêm **Q6, Q7, Q8, Q9, Q10** (đã bỏ Q4). Luôn giải thích "tại sao". Đừng "diễn".
2. **AI Engineering Challenge:** tôi chọn **#15 Multi-Tenant Configuration Platform** (advanced = 3 điểm, đủ yêu cầu). Cần nộp:
   - GitHub repo, writeup (cách tiếp cận + quyết định), **log hội thoại nguyên văn với coding agent**.
   - **Thiếu log = tự động fail. One-shot = fail.** Họ sẽ hỏi nối tiếp từng quyết định.
   - **Chưa có file đề `AI_Challenge_15.md`.** Lần sau phải gửi nội dung đề cho AI.
3. **Ngày phỏng vấn:** build trực tiếp, chọn 1 trong 4 đề (phát hiện tài liệu do AI tạo / phát hiện scan bị sửa / giảm chi phí extraction agent / xây extractor giấy ra viện từ đầu).

## 4. Quy tắc an toàn (rất quan trọng)

- **KHÔNG dùng tài liệu nội bộ Teecom** (RBAC, sprint report, Module list, tên BU, schema, roadmap) trong bài nộp, trong repo, hay dán vào công cụ AI. Chỉ dùng khái niệm chung trong đầu.
- Bài Papaya là nhiều đối tác/quốc gia/sản phẩm bảo hiểm, khác với nhiều BU trong một công ty. Cần: cô lập tenant, kế thừa cấu hình (global → country → tenant → product), versioning, ngày hiệu lực, nháp/xuất bản/rollback, audit, validation.
- **Log AI của challenge phải là phiên riêng**, không phải cuộc trò chuyện cố vấn này.
- Tên repo trung tính, **không có tên công ty**. Repo nên **private** + mời reviewer.
- Nếu bị hỏi, nói thật là đã dùng AI để review/rà chính tả cho phần câu hỏi tư duy; nội dung là của tôi.

## 5. Trạng thái các câu trả lời (thư mục `logical-answers/`)

| Câu | Trạng thái | Việc còn lại |
|---|---|---|
| Q1 | Đã viết, định dạng xong | Sửa "dôi mắt" → "đôi mắt"; "gầm gừ" bị dùng 2 lần (sấm và tiếng mưa). Tùy chọn: thêm 1 câu phân biệt phần nhớ thật / nghe kể / ước chừng; chuẩn bị trả lời miệng "có ký ức nào sớm hơn lúc 10 tuổi không?", "bài hát có thêm chi tiết vào ký ức không?". Các chi tiết mới (thau nồi, khóc vì sợ, quái vật) cần tự kiểm tra là nhớ thật hay tưởng tượng thêm. |
| Q5 | Đã viết, định dạng xong | (1) Thêm **kết quả thật** của hôm đó: có đón vợ đúng 17:30 không, có lỡ việc quan trọng không. (2) Phần checklist còn chung chung và đang ghi "tự nhớ một checklist" (yếu, vì chính việc nhớ đã thất bại): ghi cụ thể gồm gì, kiểm tra lúc nào, để ở đâu, hiệu quả ra sao, chỉ viết điều có thật. (3) Xác nhận vợ có ngồi sau xe lúc nhận ra không. (4) 15/08/2026 là thứ Bảy, kiểm tra lại ngày. (5) Xác nhận đây là lần quên gần nhất. |
| Q6 | Đã viết | Đề hỏi vị trí trong **hàng đợi vào rạp**, không phải chọn ghế. Nêu mục tiêu trước, chọn vị trí cụ thể, để ý điểm nghẽn thật (máy quét vé) và người đứng trước/sau. |
| Q8 | Đã viết | Quy trình đổ xăng nhanh nhất: giả định rõ xe/cây xăng, điểm nghẽn, song song hóa, trường hợp đặc biệt, đánh đổi. |
| Q9 | Chưa viết | Ràng buộc → các phương án → khuyến nghị. Gồm vệ sinh, riêng tư, luồng đi, mùi, khả năng tiếp cận, chi phí, bảo trì. |
| Q10 | Chưa viết | Một dấu hiệu nén duy nhất đọc từ quầy/người bán (lâu năm + nhiều loại khách + sòng phẳng), không phải từ trái cây, không dùng "quầy đông khách". |

Khi nào nghĩ ra, ghi vào đây ý chính từng câu để khỏi quên:
- Q8:
- Q9:
- Q10:

## 6. Cấu trúc repo (thư mục `papaya_insurance_quiz_test/`)

```
├── README.md                              # tổng quan (tiếng Anh, không nêu tên công ty)
├── .gitignore                             # ignore /*.md ở gốc (trừ README.md), /*.pdf, /_private/
├── SUBMISSION_CHECKLIST.md                # checklist nội bộ (ignored)
├── _private/SESSION_HANDOFF.md            # file này (ignored)
├── logical-answers/                       # q01, q05 (và q06–q10 sau này)
└── ai-challenge-15-multi-tenant-config/
    ├── README.md  (còn TODO)
    ├── docs/assumptions.md, docs/decisions.md
    ├── ai-conversation/README.md, notes.md   # log nguyên văn + ghi chú AI sai ở đâu
    ├── src/, tests/
```

Các file gốc bị ignore (cần backup riêng): JD, CV, `logical_questions.md`, `AI_engineering_challenges.md`, `papay-info.md`, `note.md`, các bản nháp `logical_q*.md`.

## 7. Việc làm tiếp theo (theo thứ tự)

1. Gửi nháp **Q6** vào `logical-answers/q06-cinema-queue.md` để được chấm.
2. Làm Q8, Q9, Q10.
3. Hoàn tất hai điểm còn thiếu ở Q5 và Q1.
4. Lấy nội dung `AI_Challenge_15.md`, rồi phân tích đề: điểm mơ hồ, giả định, kế hoạch (TypeScript, Zod, Postgres, test cô lập tenant).
5. Làm challenge trong **phiên AI riêng**, lưu log nguyên văn.
6. `git init` bên trong thư mục này, tạo repo GitHub private với tên trung tính (gợi ý `fullstack-assessment-2026`), mời reviewer.

## 8. Câu mở đầu để dán vào phiên mới

```
Bạn hãy làm người cố vấn kiêm người phỏng vấn cho tôi. Hãy đọc file
papaya_insurance_quiz_test/_private/SESSION_HANDOFF.md để nắm bối cảnh và
tiến độ, sau đó đọc luôn các file: Full_Stack_Software_Engineer_JD.md,
luong-van-tri-cv.md, logical_questions.md, AI_engineering_challenges.md và
các file trong logical-answers/. Tóm tắt lại trạng thái hiện tại trong
5 dòng rồi chúng ta làm tiếp ở mục "Việc làm tiếp theo".
```
