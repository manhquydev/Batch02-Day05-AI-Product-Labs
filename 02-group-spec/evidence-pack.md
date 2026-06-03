# Evidence Pack — AI Movie Recommendation Web App

Nộp kèm thin SPEC cuối Day 05.

## 1. Nhóm và track

**Tên nhóm:** Nhóm 4 người — 2A202600643 / 2A202600737 / 2A202600892 / 2A202600981  
**Track:** Entertainment / AI Recommendation  
**Product/app đã chọn:** ReAct Agent Movie Recommendation (từ lab Day 03 — Streamlit prototype)  
**Build slice đang nghĩ:** Nâng cấp Streamlit demo thành web app (React + FastAPI), cho phép người dùng VN hỏi phim bằng tiếng tự nhiên, nhận gợi ý có poster + streaming availability, và admin theo dõi trace/metrics.

---

## 2. Self-use evidence

Nhóm tự dùng app Streamlit hiện tại và ghi lại điểm gãy.

| Observation | Screenshot/link | Path liên quan | Điều học được |
|---|---|---|---|
| Nhập "tôi buồn muốn xem gì" → agent chạy 5-8 bước, spinner không có tiến trình rõ → user không biết đang chờ gì | `src/app.py:245` spinner "Đang suy luận..." | Low-confidence | Cần feedback realtime: "đang tìm phim... đang kiểm tra Netflix VN..." |
| Mood filter chỉ nhận 6 giá trị cứng (happy/sad/relaxed/excited/romantic/scary) — nhập "hồi hộp" hay "mệt mỏi" → trả lỗi JSON | `src/tools/mood_config.py` ALLOWED_MOODS | Failure | Cần map tự nhiên → enum, hoặc dùng LLM parse mood trước |
| App không có lịch sử phiên — refresh là mất toàn bộ chat | `src/app.py:44` init_session() chỉ dùng st.session_state | Failure | Backend cần lưu session/history nếu muốn UX liên tục |
| Streaming availability check (`check_streaming_availability`) trả "không có provider" với nhiều phim phổ biến ở VN dù thực tế có trên Netflix | `src/tools/movie_tools.py:116` | Low-confidence | TMDB watch-provider data thiếu VN — cần disclaimer rõ hoặc fallback link |
| So sánh 2+ models song song (comparison mode) hữu ích nhưng UI gom kết quả vào bảng nhỏ khó đọc trên màn hình thường | `src/app.py:213` render_comparison_result | Failure | Admin panel cần layout tốt hơn để đọc trace và metrics |

---

## 3. User / review / social evidence

| Quote / review / observation | Nguồn | User là ai? | Pain/failure mode |
|---|---|---|---|
| "App gợi ý nhưng không biết có trên Netflix VN không, cứ phải search thêm" | Nhóm phỏng vấn nhanh 3 người dùng thực tế trong lớp | Người dùng 20-30 tuổi, có Netflix/Disney+ | Streaming availability không đáng tin, phải double-check ngoài app |
| "Tôi không biết nên hỏi câu gì để app hiểu tôi muốn phim loại nào" | Quan sát live khi bạn cùng lớp dùng app lần đầu | First-time user | Onboarding yếu — không có gợi ý câu hỏi mẫu đủ đa dạng |
| "Phim gợi ý hay nhưng app trông xấu, không muốn chia sẻ link cho bạn" | Feedback trực tiếp sau khi test app Streamlit | Người dùng có ý kiến về UX | Streamlit UI không đủ polished để dùng như product thật |
| Review App Store của các app recommendation tương tự: "Tôi muốn app nhớ tôi đã xem gì để không gợi lại" | Review Letterboxd / Trakt trên App Store (quan sát public) | Heavy watcher, 20+ phim/tháng | Không có user state / watched history |

> **Lưu ý:** Evidence về review App Store là quan sát analog, không phải từ app nhóm. Nhóm sẽ validate pain "streaming availability không đáng tin" với thêm 2-3 người dùng VN trước checkpoint M1 Day 06.

---

## 4. Competitor / analog evidence

| App / mô hình tham khảo | Họ xử lý task này thế nào? | Pattern học được | Có áp dụng trong 1 ngày không? |
|---|---|---|---|
| **Letterboxd** | Diary + social proof + tag/list; không có AI recommendation real-time | Visual grid poster + rating badge hiệu quả | Có — học layout card phim |
| **IMDB** | Filter đa chiều (genre, year, rating) nhưng không có chat/AI; overwhelming với newbie | Breadcrumb filter giúp narrow down | Có — học filter pattern cho admin |
| **Netflix "Because you watched X"** | Black-box algo, không giải thích; user không biết tại sao được gợi ý | Transparency (show reasoning) là differentiator | Có — prototype hiển thị ReAct trace cho admin |
| **Perplexity AI (search với trace)** | Hiển thị source + reasoning bên cạnh answer | Trace panel song song với answer tạo trust | Có — clone layout col_chat + col_trace đã có trong app |

---

## 5. Evidence → Insight

```
Evidence nổi bật nhất:
- ReAct agent có thể trả lời câu hỏi tự nhiên ("tôi buồn muốn xem gì") 
  nhưng UI Streamlit không thể hiện được hành trình reasoning → user lo không biết app "đang làm gì".
- Streaming availability là điểm gãy lớn nhất trong happy path: 
  gợi ý hay nhưng không xem được → không có giá trị thực tế.
- App hiện tại không phân biệt guest vs admin — một UI phục vụ cả hai vai trò 
  → quá nhiều thông tin kỹ thuật (trace, metrics) hiện ra với end user.

Insight:
User không chỉ cần một danh sách gợi ý phim.
Thật ra họ cần cảm giác "được hiểu" (app hiểu mood, ngữ cảnh)
và tin tưởng vào output (streaming thật, không hallucinate tên phim).

Opportunity:
AI có thể giúp bằng cách nhận câu hỏi tự nhiên → 
tự động chọn tools TMDB phù hợp (search, mood, similar, streaming) → 
trả kết quả có poster + streaming badge VN + giải thích ngắn tại sao gợi ý phim này.
```

---

## 6. Evidence đổi SPEC như thế nào?

- [x] Đổi user chính. *(ban đầu nghĩ user là developer/học viên, sau evidence đổi thành người dùng VN thông thường, 18-35 tuổi, muốn xem phim tối nay)*
- [x] Đổi pain statement. *(từ "app cần thêm tool" → "app cần UX đáng tin và streaming info chuẩn")*
- [x] Đổi build slice. *(từ "thêm tool mới" → "nâng UI/API để guest dùng được, admin monitor được")*
- [ ] Đổi Auto/Aug decision.
- [x] Đổi 4 paths. *(thêm path "streaming không có VN" và "AI không rõ mood")*
- [x] Đổi failure mode. *(failure mode nguy hiểm nhất không phải "agent crash" mà là "gợi ý phim không stream được VN")*
- [x] Đổi owner/test plan. *(tách rõ guest UI / admin UI / backend)*

```
Trước evidence, nhóm định build thêm tool mới (watchlist, review) vào Streamlit app.

Sau evidence, nhóm đổi thành: tách guest UI (React, clean, mobile-first) + admin UI (React, 
trace/metrics dashboard) + FastAPI backend wrapping agent hiện có.

Lý do: pain lớn nhất không phải thiếu tính năng mà là UI không đủ tin tưởng và không phân 
biệt được guest vs admin. Streamlit không phù hợp để ship như product thật.
```
