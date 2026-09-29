# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Hồ Thái Hòa |
| Mã học viên | 2A202602915 |
| Repo | https://github.com/thaihoaho-code/K4-L3B-DAY12-HoThaiHoa-2A202602915-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3b-hothaihoa-2a202602915-cloud-and-deploy-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 29/09/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Redis add-on của platform |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-l3b-hothaihoa-2a202602915-cloud-and-deploy-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-l3b-hothaihoa-2a202602915-cloud-and-deploy-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k4-l3b-hothaihoa-2a202602915-cloud-and-deploy-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-l3b-hothaihoa-2a202602915-cloud-and-deploy-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-l3b-hothaihoa-2a202602915-cloud-and-deploy-production.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
# 1. Liveness
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:34:00 GMT
Server: railway-hikari
x-railway-request-id: _Lb14WnwRxyloZX-6WHkDg
Content-Length: 57
x-hikari-trace: sin1.nzn2
x-railway-edge: sin1
Connection: keep-alive

{"status":"ok","service":"day12-agent","version":"1.0.0"}

# 2. Readiness
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:34:19 GMT
Server: railway-hikari
x-railway-request-id: KJl64F4bQpitpkv_nPRhug
Content-Length: 31
x-hikari-trace: sin1.tr00
x-railway-edge: sin1
Connection: keep-alive

{"status":"ready","redis":true}

# 3. Không có API key
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:35:22 GMT
Server: railway-hikari
x-railway-request-id: Md-xUW49QhOH-lXjwUFZXw
Content-Length: 39
x-hikari-trace: sin1.tr00
x-railway-edge: sin1
Connection: keep-alive

{"detail":"invalid or missing API key"}


# 4. Có API key — mong đợi 200 kèm câu trả lời
HTTP 200 OK
x-railway-request-id: 5NnphkLBRV-ve9bGnpoFkQ
x-hikari-trace: hkg1.aebn,hnd1.2zrd
x-railway-edge: hkg1
vary: accept-encoding
Connection: keep-alive
Content-Length: 338
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:45:04 GMT
Server: railway-hikari

{
    "answer":  "Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud. (Mình đang nhớ 6 lượt trao đổi trước đó.)",
    "user_id":  "sv-test",
    "history_length":  6,
    "cost_usd":  4.785E-05,
    "tokens":  {
                   "in":  139,
                   "out":  45
               }
}

# 5. Rate limit test (15 lần)
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

