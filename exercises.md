# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng mẫu bên dưới mỗi câu bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Minh Quyền  Mã học viên: 2A202602438

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên cấu hình `AGENT_API_KEY` trên Railway, app sẽ dừng ngay khi khởi động thay vì phục vụ `/ask` bằng một khóa mặc định dễ đoán như `changeme`. Nhờ vậy mình thấy thiếu secret trong log và sửa trước khi người khác gọi API hoặc lợi dụng service.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

Một dòng log mình thu được khi gọi `/ask`:

```json
{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T05:49:07.742610+00:00", "user_id": "sv-test", "tokens_in": 3, "tokens_out": 37, "cost_usd": 2.265e-05}
```

Từ các trường này mình lọc được chi phí theo user và đếm token; cũng có thể lọc event/level để theo dõi số request hoặc lỗi. Dòng `print("đã trả lời xong")` không có dữ liệu có cấu trúc để truy vấn như vậy.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.73 GB (khoảng 1,772 MB) |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản một stage dùng base image `python:3.11` đầy đủ và cài packages trực tiếp trong image đó. Bản multi-stage dùng `python:3.11-slim`, builder chứa phần cài dependencies còn runtime chỉ nhận dependencies và source cần chạy, nên không mang toàn bộ base image/công cụ build sang image cuối.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi sửa `app/main.py`, layer cài dependencies vẫn lấy từ cache vì `requirements.txt` không đổi; các layer copy source và những layer sau đó phải build lại. Nếu `COPY . .` đứng trước `pip install`, thay đổi file source làm layer copy đổi và Docker phải chạy lại việc cài dependencies.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app có lỗi thực thi lệnh, kẻ tấn công có thể chạy lệnh với quyền của process. Chạy root cho họ quyền cao nhất bên trong container; nếu có cấu hình cô lập yếu, mount nhạy cảm hoặc lỗ hổng container runtime, họ có thể tìm cách tác động tới host. `USER appuser` giảm quyền ngay từ đầu, giới hạn quyền của process bị chiếm; nó không thay thế các lớp cô lập khác.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 20 request trong khoảng 2 giây: 10 request lúc 10:00:59 rồi thêm 10 request ngay khi bộ đếm reset lúc 10:01:00. Hai nhóm thuộc hai phút đồng hồ khác nhau nên đều được chấp nhận, dù thời gian thực giữa chúng chỉ khoảng một giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request theo thời gian; cost guard giới hạn tổng tiền theo user trong tháng. Ví dụ user gọi ít hơn 10 lần/phút nhưng mỗi prompt rất dài làm chi phí vượt budget thì cost guard phải chặn. Ngược lại, user còn nhiều budget nhưng gửi request thứ 11 trong một phút thì rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối thì cả ba container đều không kiểm tra được dependency và trả trạng thái lỗi. Nếu `/health` cũng kiểm tra Redis, orchestrator có thể coi cả ba process đã chết và restart chúng. Redis trở lại sau 30 giây nhưng các container vẫn đang restart hoặc chờ backoff, khiến cụm tạm thời không phục vụ. Tách `/ready` khỏi liveness giúp load balancer ngừng gửi traffic mà không restart process còn sống.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Qua ba agent, mình quan sát `history_length` lần lượt là `0, 2, 4, 6, 8` ở năm request cùng user; Redis giúp các instance dùng chung lịch sử. Nếu lưu trong dict Python, mỗi container có dict riêng, nên khi request sang instance mới độ dài có thể quay về 0 hoặc thấp hơn rồi tăng lại khi quay về instance cũ.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy Railway đầu, log báo `ValidationError: agent_api_key Field required`; `/health` vẫn 200 nhưng `/ready` và `/ask` lỗi. Mình kiểm tra deployment log và thấy `AGENT_API_KEY` chưa có trong Variables của service. Sau khi thêm biến trong Railway và deploy lại, `/health` và `/ready` trả 200, `/ask` không key trả 401, có key trả 200; kiểm tra 15 request cho 10 mã 200 rồi 5 mã 429.
