# Thin SPEC — AI Movie Recommendation Web App
# Nhóm Day 05 · Batch 02

---

## 1. Track, product/app và user

**Track:** Entertainment / AI Recommendation  
**Product/app thật:** ReAct Agent Movie Recommendation — nâng cấp từ Streamlit prototype (Day 03) thành React + FastAPI web app  
**User cụ thể:** Người dùng Việt Nam, 18–35 tuổi, có ít nhất 1 subscription streaming (Netflix/Disney+/Galaxy Play), muốn chọn phim tối nay mà không phải browse lâu  
**Nhóm có phải user thật không?** Có — tất cả 4 thành viên đều dùng Netflix/Disney+ và đã tự test app. Khác ở chỗ: nhóm quen với UI kỹ thuật (trace/JSON), user thông thường thì không — cần che trace khỏi guest view.

---

## 2. Evidence summary

| Evidence | Nguồn | User/pain nói lên điều gì? | SPEC phải đổi gì? |
|---|---|---|---|
| ReAct agent chạy 5-8 bước, spinner mơ hồ, user không biết đang chờ gì | Self-use app Streamlit Day 03 | User cần progress feedback trong khi agent reasoning | Guest UI phải hiện step-by-step "đang tìm phim… kiểm tra Netflix VN…" |
| `check_streaming_availability` trả empty providers với nhiều phim phổ biến ở VN | Self-use — `src/tools/movie_tools.py:116` | Streaming info không đáng tin → gợi ý mất giá trị thực tế | Hiện badge "Chưa xác nhận" thay vì im lặng; không hide thông tin thiếu |
| Nhập "hồi hộp" → mood tool báo lỗi vì chỉ nhận 6 enum cứng | Self-use — `src/tools/mood_config.py` | Mood filter UX gãy với ngôn ngữ tự nhiên VN | Agent phải map trước ("hồi hộp" → excited) trước khi gọi tool |
| "App kỹ thuật quá, không muốn chia sẻ link" | Feedback 3 người dùng trong lớp | Streamlit không đủ polished làm product thật | Tách guest UI (clean, không trace) vs admin UI (có trace/metrics) |
| Letterboxd: visual grid poster + rating badge hoạt động tốt cho browse | Analog competitor | Layout card phim tốt hơn danh sách text | Guest UI dùng poster grid, không dùng chat bubble cho kết quả |

---

## 3. Pain statement

```
User Việt Nam 18-35 tuổi đang gặp khó ở bước "chọn phim tối nay",
vì các app recommendation hoặc không hiểu ngôn ngữ tự nhiên / tâm trạng,
hoặc gợi ý phim không biết có stream được ở VN không,
dẫn tới mất thêm 10-15 phút search ngoài app rồi không tin vào gợi ý.
Bằng chứng chính là: self-use (streaming empty, mood error) + 
feedback "gợi ý hay nhưng không biết xem trên đâu".
```

---

## 4. Build slice

```
Cho người dùng VN có Netflix/Disney+ đang muốn chọn phim tối nay,
prototype sẽ dùng AI ReAct agent (9 TMDB tools) để nhận câu hỏi tự nhiên
("tôi buồn muốn xem phim nhẹ nhàng", "phim Sci-Fi trending tuần này"),
tự chọn tool phù hợp (filter_by_mood / search_movies / get_trending_movies + 
check_streaming_availability),
tạo ra: poster grid 3-5 phim với streaming badge VN + 1-2 câu giải thích tại sao gợi ý,
và xử lý failure mode "streaming không rõ VN" bằng badge "Chưa xác nhận platform VN" 
kèm link search thêm — không im lặng, không hallucinate.
```

---

## 5. Auto/Aug decision

- [x] **Augmentation:** AI gợi ý/draft/phân loại, user quyết cuối.

**Lý do chọn:** TMDB data live — agent không tự sinh phim mà chỉ search/filter. User vẫn là người chọn xem phim nào thật sự. Automation toàn phần (tự play phim) nằm ngoài scope Day 06.  
**Human role:** decider — user đọc gợi ý và tự quyết định xem phim nào.

---

## 6. Four paths

| Path | Prototype phải thể hiện gì? |
|---|---|
| **Happy** | User nhập "tôi buồn muốn xem phim nhẹ nhàng" → agent gọi `filter_by_mood("sad")` + `check_streaming_availability` → trả 3-5 phim có poster, điểm TMDB, badge streaming VN, câu giải thích ngắn. Latency ≤ 15 giây. |
| **Low-confidence** | User nhập câu mơ hồ ("xem gì đi") → agent hỏi lại "Bạn đang cảm thấy thế nào?" hoặc hiện 3 mood suggestion để user chọn; không tự đoán sai. |
| **Failure** | TMDB trả empty result / API timeout → agent báo lỗi rõ ràng + gợi ý thay thế ("Thử mood khác?", "Xem trending tuần này?"); không crash silent. |
| **Correction** | User không thích gợi ý đầu ("phim này tôi xem rồi") → có thể nhập tinh chỉnh ("gợi ý phim khác của đạo diễn này"); agent chạy lại với context mới. |

---

## 7. Failure mode nguy hiểm nhất

```
Nếu user hỏi "phim này có trên Netflix VN không?",
AI có thể trả lời "có" dựa trên TMDB provider data cũ/thiếu,
hậu quả là user mở Netflix tìm không thấy → mất tin vào app hoàn toàn.

Prototype sẽ xử lý bằng:
- Hiển thị badge "Chưa xác nhận" thay vì khẳng định có/không khi TMDB trả empty providers.
- Luôn thêm link "Kiểm tra trực tiếp trên JustWatch VN" bên cạnh streaming info.
- Không để LLM tự khẳng định streaming availability — chỉ relay data từ TMDB API.

Owner kiểm thử path này: Nguyễn Mạnh Quý (2A202600643) — admin UI trace viewer.
```

---

## 8. Owner plan cho sáng Day 06

| Thành viên | Mã HV | Việc phụ trách | Bằng chứng cần có trong repo |
|---|---|---|---|
| Trịnh Thị Lan Anh | 2A202600737 | **Guest UI (React):** chat input, poster grid, streaming badge, mood selector, step-by-step progress | `frontend/src/components/GuestChat.tsx`, `MovieCard.tsx`, `StreamingBadge.tsx` |
| Nguyễn Mạnh Quý | 2A202600643 | **Admin UI (React):** trace viewer panel, metrics dashboard (latency, tool calls, steps), model selector | `frontend/src/components/AdminDashboard.tsx`, `TracePanel.tsx`, `MetricsTable.tsx` |
| Nguyễn Thanh Anh Quân | 2A202600892 | **FastAPI endpoints:** `POST /recommend`, `GET /search`, `GET /trending`, `GET /movie/{id}`, wrap ReAct agent + tools | `backend/app/routers/recommend.py`, `search.py`, `trending.py` |
| Nguyễn Đình Bảo Long | 2A202600981 | **FastAPI infra:** project setup, CORS, .env config, Pydantic schemas, error handling, deployment script | `backend/app/main.py`, `schemas/`, `core/config.py`, `Dockerfile` / `README-deploy.md` |

---

## 9. Tech stack Day 06

| Layer | Tech | Ghi chú |
|---|---|---|
| Backend | Python 3.11 + FastAPI + Uvicorn | Giữ nguyên agent/tools từ Day 03 |
| LLM | OpenAI GPT-4o-mini (mặc định) / DeepSeek / Ollama | Giữ factory pattern hiện có |
| TMDB | TMDB API v3 (live data) | Giữ `src/tools/tmdb_client.py` |
| Frontend | React 18 + TypeScript + Vite | Mới hoàn toàn |
| Styling | Tailwind CSS | Nhanh cho 1 ngày |
| State | React useState / useEffect (no Redux) | KISS — không phức tạp hóa |
| Auth | Không có (Day 06) | Guest và admin phân biệt bằng route `/` vs `/admin` |

---

## 10. Định nghĩa "done" cho Day 06 demo

- [ ] Guest có thể nhập câu tự nhiên và nhận poster grid ≤ 15 giây
- [ ] Streaming badge hiện đúng (hoặc "Chưa xác nhận" — không im lặng)
- [ ] Failure path "không rõ mood" → agent hỏi lại thay vì crash
- [ ] Admin có thể xem trace và latency của request vừa chạy
- [ ] FastAPI `POST /recommend` trả JSON có `movies[]`, `trace[]`, `metrics{}`
- [ ] Repo có README với hướng dẫn chạy local (`.env.example`, `npm run dev`, `uvicorn`)
