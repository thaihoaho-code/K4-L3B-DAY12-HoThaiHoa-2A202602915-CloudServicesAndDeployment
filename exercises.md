# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> Họ và tên: Hồ Thái Hòa  Mã học viên: 2A202602915

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định `"changeme"`, khi deploy lên Cloud mà quên setup biến môi trường, app vẫn khởi động bình thường. Tuy nhiên, bất cứ ai biết được key mặc định `"changeme"` cũng có thể qua mặt lớp xác thực và gọi API thả ga. Việc thiết kế Fail-Fast giúp ứng dụng văng lỗi và ngừng khởi động ngay lúc deploy, bắt buộc dev phải khai báo key chuẩn xác thì mới chạy.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T10:00:00+00:00", "user_id": "sv01", "cost_usd": 0.0001}`
> Hai việc làm được:
> 1. Truy vấn, tính toán dễ dàng trên các hệ thống log management (Datadog, Kibana): ví dụ như dùng lệnh query để tìm tổng chi phí (`cost_usd`) của toàn bộ request trong ngày.
> 2. Có thể dễ dàng filter log theo `level` (như lọc các log error) hoặc filter theo đúng tên user, điều mà `print()` thuần văn bản phải dùng biểu thức Regex cực kỳ phức tạp để bắt.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~ 1000 MB |
| Multi-stage | ~ 300 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch là những công cụ build tool (như trình biên dịch gcc, make), các file cache tải xuống của pip, mã nguồn gốc của hệ điều hành và các thư viện C vốn chỉ cần để cài đặt package. Ở dạng multi-stage, ta build mọi thứ ở stage đầu, sau đó chỉ copy những file thực thi đã biên dịch xong sang stage runtime (rất mỏng) ở cuối cùng, vứt bỏ toàn bộ những rác sinh ra trong quá trình cài đặt.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Các layer cài đặt hệ thống và đặc biệt là `COPY requirements.txt` cùng `RUN pip install` sẽ được dùng lại nguyên vẹn từ cache. Chỉ có layer `COPY . .` và lệnh CMD phía dưới mới phải chạy lại.
> Ngược lại, nếu đặt `COPY . .` trước `RUN pip install`, thì khi sửa bất kì 1 ký tự code nào, Docker sẽ thấy context thay đổi, làm layer `COPY` thay đổi (vỡ cache), dẫn đến layer `RUN pip install` ngay phía sau cũng bị mất cache và phải tải/cài lại toàn bộ hàng chục thư viện từ đầu rất tốn thời gian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Chuỗi sự kiện: Ứng dụng dính lỗ hổng thực thi mã từ xa (RCE) ➜ Hacker chạy mã độc từ xa ➜ Do app đang dùng quyền root, hacker thao túng được quyền root bên trong container ➜ Kẻ tấn công lợi dụng lỗ hổng escape container để thoát ra máy chủ host bên ngoài, lúc này chúng mang nguyên đặc quyền root đó để chiếm quyền toàn bộ máy chủ thật. 
> Lệnh `USER appuser` cắt đứt ở bước 3: Dù hacker có chạy được mã độc thì chúng cũng chỉ ở quyền user thường `appuser` bên trong container, không đủ quyền hạn để phá hoại, khai thác nhân hệ thống hay escape ra máy host.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Một người có thể gửi tối đa **20 request** trong vỏn vẹn 2 giây.
> Giải thích: Giả sử hạn mức là 10 req/phút. Kẻ tấn công canh thời điểm lúc 10:00:59 để bắn 10 request (lúc này chúng vẫn trong phút 0). Đúng 1 giây sau, đồng hồ chuyển sang 10:01:00, bộ đếm phút bị reset về 0, kẻ tấn công lập tức xả tiếp 10 request nữa (thuộc phút 1). Kết quả là chỉ trong nháy mắt (2s), server phải gánh 20 request hợp lệ theo luật, dẫn tới việc hệ thống có nguy cơ sập.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> - **Khác biệt:** Rate Limit chặn theo số "lượt" gọi trong khoảng thời gian rất ngắn (chống spam hệ thống). Cost Guard chặn theo tổng "tiền" gọi trong cả 1 tháng (chống cạn ví tiền túi).
> - **Tình huống Rate Limit cho qua, Cost Guard chặn:** User mới gửi 1 câu hỏi đầu tiên trong phút này (không vi phạm Rate Limit), nhưng câu hỏi đó quá dài tốn 1 triệu token, đẩy tổng chi phí tháng vượt mốc 10 USD ➜ Bị Cost Guard chặn ngay lập tức (mã 402).
> - **Tình huống ngược lại:** User mới lập acc, hỏi những câu cực ngắn (rất rẻ, dư sức qua ải Cost Guard), nhưng họ lại dùng tool auto spam gọi 100 câu trong 1 phút ➜ Lập tức bị Rate Limit chặn (mã 429).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> 1. Redis mất kết nối 30s. 
> 2. Do endpoint đã gộp có kiểm tra Redis, nên healthcheck của cả 3 container đồng loạt báo lỗi (Liveness rớt).
> 3. Cloud Orchestrator tưởng cả 3 container đang bị treo cứng, nên ra lệnh KILL và khởi động lại toàn bộ. 
> 4. Trong lúc 3 container đang khởi động lại từ đầu, hệ thống không có ai phục vụ khách hàng, user gặp lỗi sập web diện rộng. Dù Redis có hồi phục lúc đó thì cũng đã muộn. (Sự cố nhỏ bị khuếch đại thành sập toàn bộ hệ thống).

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Con số `history_length` sẽ trồi sụt bất thường (ví dụ: 1, 1, 1, 2, 2, 3...) thay vì tăng dần đều.
> Lý do là vì Load Balancer phân phát các câu hỏi của bạn xoay vòng ngẫu nhiên vào 3 container (A, B, C). Nếu lưu bằng dict Python, mỗi container sẽ có "vùng nhớ" riêng biệt. Câu 1 vào container A, dict A lưu là 1. Câu 2 vào container B, B không có dict của A nên tưởng là người dùng mới, trả về length là 1, tạo ra hiện tượng "mất trí nhớ ngẫu nhiên".

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> **Thông báo lỗi:** Lỗi mã `301 Moved Permanently` khi gõ lệnh curl.
> **Nguyên nhân tìm ra:** Lệnh `curl` theo hướng dẫn gọi vào domain không ghi cụ thể giao thức, nên nó chạy bằng HTTP thường. Trong khi đó, Load Balancer của Railway mặc định bắt buộc sử dụng mạng bảo mật nên nó từ chối và chặn lại bắt chuyển hướng (mã 301).
> **Cách sửa:** Gắn thêm tiền tố `https://` vào trước URL trong lệnh curl hoặc truyền thêm cờ `-L` để curl tự động theo dấu link redirect. Kết quả đã trả về `200 OK` thành công 100%.

