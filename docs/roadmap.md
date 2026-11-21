# Roadmap Parcel Tracking System

## M0: Project Setup (đang làm)
- [x] **M0-1** `chore/m0-safety-setup`: sửa .gitignore, thêm .gitattributes, .editorconfig, docs/recovery.md
- [ ] **M0-2** `docs/m0-roadmap`: tạo docs/roadmap.md (M0-M9, MVP checklist, Definition of Done)
- [ ] **M0-3** `docs/m0-restore-architecture` commit 1: khôi phục architecture.md từ 861c529
- [ ] **M0-4** `docs/m0-restore-architecture` commit 2: tách MVP/M8, thêm MVP design
- [ ] **M0-5** `chore/m0-git-hooks`: .githooks/pre-commit (chặn path cấm, regex secret, dotnet build)
- [ ] **M0-6** Test hook: chặn .env, detai.md, password=...; qua .env.example, docs/roadmap.md
- [ ] **M0-7** `ci/m0-secret-scan`: .github/workflows/ci.yml (build, test, gitleaks)
- [ ] **M0-8** `docs/m0-readme`: cập nhật README, hướng dẫn hook, cài gitleaks
- [ ] **M0-10** Tag `checkpoint/m0-done`, push, báo cáo Definition of Done

## M1: Infrastructure Foundation
**MVP**: docker-compose.yml (postgres, rabbitmq, healthcheck, volume, network), .env.example đầy đủ, logging mặc định .NET có kèm trackingCode, project test xUnit. Tự kết nối lại RabbitMQ/DB, retry đơn giản giới hạn, log kèm trackingCode.
- [ ] docker-compose.yml
- [ ] .env.example cập nhật
- [ ] Logging cơ bản có trackingCode
- [ ] Project test xUnit
- [ ] Tự reconnect RabbitMQ/DB, retry giới hạn

**Definition of Done M1**: `docker compose up -d` chạy postgres + rabbitmq healthy; `dotnet build` + `dotnet test` pass; log có trackingCode.

## M2: Domain & Database
**MVP**: Entity Shipment, ShipmentEvent, enum Status; EF Core + migration; sinh tracking code PKG-YYYYMMDD-XXXXXXXX không trùng.
- [ ] Entity Shipment, ShipmentEvent, enum Status
- [ ] EF Core packages + DbContext
- [ ] Migration 5 bảng Tracking + 2 bảng Notification
- [ ] Tracking code generator unique

**Definition of Done M2**: Migration chạy thành công; tracking code không trùng (unique constraint + retry).

## M3: Tracking API Endpoints
**MVP**: 4 endpoint bắt buộc (POST /shipments, GET /shipments/{code}, POST /shipments/{code}/events, GET /shipments/{code}/history), validation, 400/404/409, Swagger.
- [ ] POST /shipments
- [ ] GET /shipments/{code}
- [ ] POST /shipments/{code}/events
- [ ] GET /shipments/{code}/history
- [ ] Validation + error handling
- [ ] Swagger/OpenAPI

**Definition of Done M3**: 4 endpoint chạy đúng contract; Swagger UI accessible.

## M4: RabbitMQ Event Publishing
**MVP**: Event contract ShipmentStatusChanged; publisher durable/persistent; tự kết nối lại RabbitMQ/DB; retry đơn giản giới hạn; log kèm trackingCode.
- [ ] Event contract
- [ ] Publisher với durable/persistent
- [ ] Auto-reconnect RabbitMQ
- [ ] Retry giới hạn + log trackingCode

**Definition of Done M4**: Event publish thành công; restart RabbitMQ -> publisher tự reconnect.

## M5: Notification Service
**MVP**: Consumer ack/nack; SignalR push realtime (group per trackingCode); retry đơn giản giới hạn; tự phục hồi.
- [ ] Consumer ack/nack
- [ ] SignalR Hub + group per trackingCode
- [ ] Retry giới hạn
- [ ] Tự phục hồi khi component down

**Definition of Done M5**: Event consume -> SignalR push -> Customer Client nhận được; restart service -> tự reconnect.

## M6: Clients
**MVP**: Operator.Client tạo bưu gửi, cập nhật trạng thái, xem lịch sử; Customer.Client tra cứu theo mã, xem hành trình, nhận thông báo realtime.
- [ ] Operator.Client: tạo bưu gửi, cập nhật trạng thái, xem lịch sử
- [ ] Customer.Client: tra cứu, xem hành trình, nhận realtime
- [ ] Tự kết nối lại SignalR khi disconnect

**Definition of Done M6**: Operator tạo shipment -> Customer nhận realtime update; restart client -> tự reconnect.

## M7: Containerization
**MVP**: 5 Dockerfile (Operator.Client, Customer.Client trong compose profile), healthcheck, restart policy, test tắt/bật thành phần.
- [ ] 5 Dockerfile (multi-stage build)
- [ ] Healthcheck mỗi service
- [ ] Restart policy
- [ ] Test tắt/bật postgres, rabbitmq, api, notification

**Definition of Done M7**: `docker compose up -d` chạy toàn hệ thống; tắt/bật bất kỳ service nào -> tự hồi phục.

## M8: Extensions
- [ ] Outbox pattern (outbox_messages, Outbox Publisher)
- [ ] 3 Retry Queue + DLQ (RabbitMQ topology)
- [ ] Idempotent consumer (processed_events, claim/lease)
- [ ] Ordering guarantee (shipment_processing_state)
- [ ] JWT Authentication (operators, HS256, 1h)
- [ ] SubscriptionToken + customer_subscriptions + Internal API
- [ ] Correlation ID middleware
- [ ] Rate limiting
- [ ] 2 DbContext + migration ops script
- [ ] Serilog structured logging JSON
- [ ] Health check endpoints
- [ ] Retention policies, DLQ replay

**Definition of Done M8**: Toàn bộ M8 features chạy; test end-to-end với retry/DLQ.

## M9: Handover
- [ ] README hoàn chỉnh
- [ ] Sơ đồ kiến trúc (Mermaid trong README/docs)
- [ ] CHANGELOG.md
- [ ] Tag v1.0.0
- [ ] Kịch bản demo

**Definition of Done M9**: Repo sẵn sàng bàn giao; demo chạy được.