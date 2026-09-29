# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục         | Nội dung                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------------------- |
| Họ và tên   | Nguyễn Minh Ngọc                                                                                               |
| Mã học viên | 2A202602530                                                                                                    |
| Repo        | https://github.com/nguyenminhngoc234it-gif/K4-L3B-DAY12-NguyenMinhNgoc-L3B202602530-CloudServicesAndDeployment |

## Service

| Mục         | Nội dung                                |
| ----------- | --------------------------------------- |
| Public URL  | https://day12-agent-7iu7.onrender.com   |
| Platform    | Railway / Render / Cloud Run — (Render) |
| Ngày deploy | (29/09/2026)                            |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến                    | Đã set | Ghi chú                                           |
| ----------------------- | ------ | ------------------------------------------------- |
| `PORT`                  | ✅     | platform tự gán                                   |
| `AGENT_API_KEY`         | ✅     | đặt trong dashboard, không nằm trong repo         |
| `REDIS_URL`             | ✅     | (điền: Redis add-on của platform / Upstash / ...) |
| `RATE_LIMIT_PER_MINUTE` | ✅     | 10                                                |
| `MONTHLY_BUDGET_USD`    | ✅     | 10.0                                              |
| `LOG_LEVEL`             | ✅     | INFO                                              |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-7iu7.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-7iu7.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-7iu7.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl- i -X POST https://day12-agent-7iu7.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-7iu7.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
# 1. Liveness — mong đợi 200 {"status":"ok"}
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:34:35 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
cf-cache-status: DYNAMIC
rndr-id: 81b10c38-5caf-43e0
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
CF-RAY: a42846b97c4ee2f7-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}
# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
HTTP/1.1 200 OK
Date: Tue, 29 Sep 2026 04:36:21 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 39ce4bf6-c94d-4e30
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a428494e8b99eb0f-HKG
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}
# 3. Không có API key — mong đợi 401
HTTP/1.1 401 Unauthorized
Date: Tue, 29 Sep 2026 04:58:00 GMT
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
rndr-id: 103297c4-4e35-41c7
Server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
CF-RAY: a4286903096c0984-HKG
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing API key"}
# 4. Có API key — mong đợi 200 kèm câu trả lời
HTTP/1.1 200 OK
content-type: application/json
{"answer":"Ngắn gọn: Deploy là gì phụ thuộc vào ba yếu tố — cấu hình qua biến môi trường, health check để orchestrator biết trạng thái, và giới hạn tài nguyên.","user_id":"sv-test","history_length":2,"cost_usd":0.0000357,"tokens":{"in":50,"out":47}}
# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
(điền lý do nếu dùng phương án dự phòng, ngược lại xóa mục này)
```
