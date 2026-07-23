# K3 — Ngày 1: Bài Tập & Phản Ánh
## Khám Phá LLM API | Phiếu Thực Hành

**Thời lượng:** 9h00–13h00
**Cách làm:** Trả lời từng câu ngay sau khi hoàn thành block tương ứng —
đừng để dồn hết về cuối buổi. Thay dòng `*Câu trả lời của bạn*` bằng câu
trả lời thật (chấm tự động sẽ đếm số câu đã trả lời).

---

## Block 1 — API Cơ Bản (trả lời sau Checkpoint 1)

### Câu 1.1 — Độ nhạy của temperature
Gọi `call_openai` với temperature 0.0, 0.5, 1.0 và 1.5 dùng prompt
**"Hãy kể cho tôi một sự thật thú vị về Việt Nam."**

**Bạn nhận thấy quy luật gì qua bốn phản hồi?** (2–3 câu)
> *Ở temperature 0.0, model gần như trả lời giống hệt nhau mỗi lần gọi lại — chọn sự thật "an toàn", phổ biến (ví dụ về diện tích, dân số). Khi tăng lên 0.5–1.0, câu trả lời bắt đầu đa dạng hơn về cách diễn đạt và đôi khi chọn sự thật khác lạ hơn. Ở 1.5, phản hồi có thể trở nên lan man, thậm chí thiếu chính xác hoặc lặp từ bất thường — model "sáng tạo" nhưng đánh đổi bằng độ tin cậy*

### Câu 1.2 — Chọn temperature cho sản phẩm
**Bạn sẽ đặt temperature bao nhiêu cho chatbot hỗ trợ khách hàng, và tại sao?**
> *Chatbot hỗ trợ khách hàng nên đặt temperature thấp (0.0–0.3), vì mục tiêu là câu trả lời nhất quán, chính xác, đúng chính sách công ty — không cần "sáng tạo", và nhất quán giúp dễ kiểm soát chất lượng, tránh model bịa thông tin (hallucination) không mong muốn*

### Câu 1.3 — Đánh đổi chi phí
Kịch bản: 10.000 người dùng hoạt động mỗi ngày, mỗi người gọi API 3 lần,
mỗi lần trung bình ~350 token đầu ra.

**Ước tính GPT-4o đắt hơn GPT-4o-mini bao nhiêu lần cho workload này? Nêu một
trường hợp GPT-4o xứng đáng với chi phí và một trường hợp nên dùng mini:**
> *Theo PRICING_PER_1K_TOKENS, giá output GPT-4o là 0.010 USD/1K token, GPT-4o-mini là 0.0006 USD/1K token → GPT-4o đắt hơn khoảng 16.7 lần cho cùng số token output. Với workload 10.000 user × 3 lần/ngày × 350 token ≈ 10.5 triệu token output/ngày: dùng GPT-4o tốn ~105 USD/ngày, mini chỉ ~6.3 USD/ngày. GPT-4o đáng dùng khi task cần lý luận phức tạp, độ chính xác cao (tư vấn pháp lý, debug code phức tạp); mini phù hợp cho tác vụ đơn giản, khối lượng lớn (FAQ, phân loại intent, trả lời chào hỏi).*

---

## Block 2 — System Prompt & Token (trả lời sau Checkpoint 2)

### Câu 2.1 — Sức mạnh của persona
Gọi `chat_with_system_prompt` hai lần với cùng câu hỏi
**"Giải thích blockchain là gì?"** nhưng hai system prompt khác nhau:
- "Bạn là giáo viên tiểu học, giải thích thật đơn giản cho trẻ 8 tuổi."
- "Bạn là chuyên gia tài chính, trả lời chuyên sâu bằng thuật ngữ kỹ thuật."

**Hai phản hồi khác nhau như thế nào (độ dài, từ vựng, ví dụ)? System prompt
ảnh hưởng đến hành vi model ra sao?** (3–4 câu)
> *Với persona "giáo viên tiểu học", phản hồi ngắn gọn hơn, dùng từ đơn giản, nhiều ví dụ đời thường (so sánh blockchain với "cuốn sổ ghi chép chung"). Với persona "chuyên gia tài chính", phản hồi dài hơn, dùng thuật ngữ kỹ thuật (hash, phân tán, consensus, smart contract) và đi sâu vào ứng dụng/tác động. Điều này cho thấy system prompt định hình rất mạnh giọng văn, độ sâu và từ vựng của model dù user prompt giống hệt nhau — vì model coi system prompt là "vai diễn" chi phối toàn bộ phản hồi.*

### Câu 2.2 — tiktoken vs đếm từ
Chọn một đoạn văn tiếng Việt ~100 từ. So sánh số token theo `count_tokens`
(tiktoken) với ước lượng `số từ / 0.75` mà Part 1 đã dùng.

**Hai con số chênh nhau bao nhiêu phần trăm? Vì sao tiếng Việt thường tốn
nhiều token hơn tiếng Anh cùng độ dài?**
> *Với đoạn văn tiếng Việt ~100 từ, số token đếm bằng tiktoken thường cao hơn 40–60% so với ước lượng "số từ / 0.75" (tức khoảng 133 token ước lượng thô). Lý do: tiếng Việt có dấu thanh và ký tự Unicode tổ hợp (ví dụ "ế", "ộ") mà bộ mã hóa BPE của OpenAI (được huấn luyện chủ yếu trên tiếng Anh) phải tách thành nhiều sub-token hơn để biểu diễn, trong khi một từ tiếng Anh thường gói gọn trong 1 token*

---

## Block 3 — Streaming & Độ Bền (trả lời sau Checkpoint 3)

### Câu 3.1 — Trải nghiệm người dùng với streaming
**Streaming quan trọng nhất trong trường hợp nào, và khi nào thì
non-streaming lại phù hợp hơn?** (1 đoạn văn)
> *Streaming quan trọng nhất khi phản hồi dài hoặc độ trễ cao — người dùng thấy chữ xuất hiện ngay lập tức thay vì chờ trống màn hình, cảm giác "nhanh" hơn dù tổng thời gian không đổi (quan trọng cho chatbot, trợ lý viết văn bản dài). Non-streaming phù hợp hơn khi cần xử lý toàn bộ output trước khi dùng — ví dụ parse JSON, gọi function calling, hoặc chạy batch job không có người xem trực tiếp, vì lúc đó việc stream từng chunk không mang lại lợi ích UX nào mà còn phức tạp hóa code*

### Câu 3.2 — Vì sao backoff theo cấp số nhân?
**So với delay cố định (ví dụ luôn chờ 1 giây), exponential backoff có lợi
thế gì khi API bị quá tải? Điều gì xảy ra nếu hàng nghìn client cùng retry
với delay cố định giống nhau?**
> *Exponential backoff giãn thời gian chờ ra dần (0.1s → 0.2s → 0.4s...) nên giảm tải lên server đang quá tải theo thời gian, thay vì dội liên tục. Nếu hàng nghìn client cùng dùng delay cố định giống nhau (ví dụ luôn chờ 1s), tất cả sẽ đồng loạt retry cùng lúc theo từng "đợt" — tạo ra các đợt sóng tải (thundering herd) khiến server vừa hồi phục lại bị đánh sập tiếp, kéo dài thời gian downtime thay vì giúp hệ thống phục hồi*

---

## Block 4 — Mini-Project (trả lời sau Checkpoint 4)

### Câu 4.1 — Thiết kế persona
**Bạn chọn persona gì cho trợ lý của mình? Viết lại system prompt đó và giải
thích 1–2 lựa chọn từ ngữ quan trọng trong prompt (ví dụ: vì sao yêu cầu
"trả lời ngắn gọn", vì sao chỉ định ngôn ngữ...):**
> *Persona: "Bạn là trợ giảng thân thiện của khóa AI, trả lời ngắn gọn bằng tiếng Việt, ưu tiên ví dụ thực tế thay vì lý thuyết dài dòng." Yêu cầu "trả lời ngắn gọn" giúp giảm token output → giảm chi phí và độ trễ, đồng thời phù hợp ngữ cảnh chat CLI (người dùng đọc trên terminal, không muốn cuộn dài). Chỉ định "bằng tiếng Việt" đảm bảo tính nhất quán ngôn ngữ vì model mặc định có thể trả lời tiếng Anh nếu câu hỏi mơ hồ.*

### Câu 4.2 — Hạn chế & cải thiện
**Trợ lý của bạn hiện có hạn chế lớn nhất là gì (ví dụ: history chỉ 3 lượt,
không có bộ nhớ dài hạn, không kiểm duyệt nội dung...)? Đề xuất một cải
thiện cụ thể và mô tả ngắn cách triển khai:**
> *Hạn chế lớn nhất: history chỉ giữ 3 lượt gần nhất nên trợ lý "quên" ngữ cảnh nếu cuộc hội thoại dài hơn — user hỏi lại điều đã nói ở lượt 1 thì trợ lý không còn nhớ. Cải thiện đề xuất: thêm bước tóm tắt (summarization) — khi history sắp bị cắt, gọi một lời gọi API phụ để nén các lượt cũ thành 1 đoạn tóm tắt ngắn, lưu vào đầu history thay vì xóa hẳn, giúp giữ ngữ cảnh dài hạn mà không làm phình token mỗi lượt.*

---

## Danh Sách Kiểm Tra Nộp Bài

- [ ] `python grade.py` — xem điểm tự động, mục tiêu ≥ 75/100
- [ ] Cả 4 checkpoint pytest đều pass
- [ ] Tất cả 9 câu trong file này đã được trả lời
- [ ] Đã copy bài làm vào folder `solution/` và zip theo hướng dẫn README
