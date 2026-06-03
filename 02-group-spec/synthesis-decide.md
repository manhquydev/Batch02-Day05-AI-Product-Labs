# Toolkit — Từ Evidence Đến Build Slice
# AI Movie Recommendation Web App — Nhóm Day 05

---

## 1. Gom evidence thành cụm

| Cụm pain | Evidence thuộc cụm |
|---|---|
| **"Không biết app đang làm gì"** | ReAct agent chạy 5-8 bước không có progress feedback; spinner mơ hồ |
| **"Gợi ý phim xem không được"** | check_streaming_availability thiếu data VN; user phải tự kiểm tra ngoài app |
| **"App kỹ thuật quá, không muốn dùng"** | Streamlit UI không polished; trace/JSON lộ ra với end user; không chia sẻ được |
| **"Không nhớ tôi đã xem gì"** | Không có session persistence; refresh mất toàn bộ history |
| **"Mood filter quá cứng"** | Chỉ 6 enum; "hồi hộp", "mệt mỏi" → error; không map tự nhiên |

---

## 2. Viết insight

```
User Việt Nam 18-35 tuổi muốn tìm phim tối nay không chỉ cần một danh sách gợi ý.
Họ thật ra cần cảm giác "app hiểu mình" (nhận ngôn ngữ tự nhiên, hiểu tâm trạng)
và tin tưởng output (phim thật, stream được trên Netflix/Disney+ VN),
vì evidence cho thấy điểm gãy lớn nhất là: gợi ý hay nhưng không xem được.
```

---

## 3. Viết opportunity

```
Cơ hội là dùng AI ReAct agent để nhận yêu cầu tự nhiên → tự chọn TMDB tools phù hợp
(search, mood, similar, streaming) → trả danh sách phim có poster + streaming badge VN + 
giải thích ngắn tại sao gợi ý phim này,
giúp user chọn được phim trong vòng 30 giây thay vì browse mỏi tay,
trong khi vẫn kiểm soát rủi ro hallucinate bằng cách 100% data lấy từ TMDB API live,
không để LLM tự sinh tên phim hay streaming info.
```

---

## 4. Chọn build slice

Build slice đã qua 5 câu hỏi kiểm tra:

| Câu hỏi | Trả lời | Đạt? |
|---|---|---|
| User cụ thể chưa? | Người dùng VN, 18-35 tuổi, có Netflix/Disney+, muốn chọn phim tối nay trong 30 giây | ✅ |
| Task đủ hẹp chưa? | Nhập 1 câu tự nhiên → nhận 3-5 gợi ý có poster + streaming + giải thích ngắn. Demo 3 phút | ✅ |
| AI decision rõ chưa? | Agent tự chọn tools: dùng `filter_by_mood` hay `get_similar_movies` hay `get_trending_movies` tùy context | ✅ |
| Failure path rõ chưa? | Case: mood không rõ → agent hỏi lại; streaming không có VN → hiện badge "Không rõ platform" | ✅ |
| Có evidence không? | Self-use + 3 người phỏng vấn trong lớp + analog review Letterboxd/IMDB | ✅ |

---

## 5. Quyết định: giữ, giảm scope, hay đổi hướng?

| Tình huống | Quyết định nhóm |
|---|---|
| Có sẵn Python agent + 9 TMDB tools từ Day 03 | Giữ toàn bộ — wrap FastAPI, không rewrite |
| History/watchlist là pain thật nhưng cần DB | Đưa vào **backlog** — Day 06 không build |
| Admin trace viewer đã có logic trong Streamlit | Giữ — clone sang React admin panel |
| Mood enum cứng gây failure | Giảm scope: Day 06 chỉ thêm map 2-3 mood tự nhiên phổ biến (hồi hộp → excited) |
| Mobile UI | Backlog — focus desktop responsive trước |

---

## 6. Câu chốt cuối

```
Dựa trên self-use evidence (streaming thiếu VN, UI không tin được, mood cứng)
và feedback 3 người dùng trong lớp (muốn biết phim có stream được không),

nhóm sẽ build: React + FastAPI web app bọc ReAct agent TMDB hiện có,
  - Guest UI: chat box nhập tự nhiên → poster grid + streaming badge VN
  - Admin UI: trace viewer + metrics (tool calls, latency, steps)

cho người dùng VN 18-35 tuổi có Netflix/Disney+, muốn chọn phim tối nay,

để giải quyết pain: "gợi ý hay nhưng không biết xem trên đâu" + "app kỹ thuật quá",

bằng cách AI ReAct agent tự chọn tools TMDB (mood/search/similar/streaming) 
và trả kết quả structured với poster + streaming info + 1-2 câu giải thích,

và sẽ test failure path: agent gợi ý phim không có trên streaming VN →
  hiện badge "Chưa xác nhận platform VN" + link tìm thêm.
```

---

## 7. Backlog (không build Day 06)

- Watchlist / watched history (cần DB + auth)
- Mobile app / PWA
- Social sharing (link gợi ý)
- Review / rating của user
- Multi-language UI (English)
- Personalization từ lịch sử xem
- Real-time streaming data (JustWatch API thay TMDB provider)
- Push notification "phim mới trending"
