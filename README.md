# Parcel Tracking System

Hệ thống quản lý và theo dõi hành trình bưu gửi qua nhiều điểm xử lý.

## Kiến trúc hệ thống

```
Operator Client -> Tracking API -> Database
                         |
                         | Event
                         v
                      RabbitMQ -> Notification Service -> Customer Client
```

## Công nghệ sử dụng

- **Language**: C# / .NET 10
- **API**: ASP.NET Core Minimal API
- **Message Broker**: RabbitMQ
- **Database**: PostgreSQL (planned)
- **Containerization**: Docker & Docker Compose

## Cấu trúc dự án (dự kiến)

```
ParcelTrackingSystem/
├── src/
│   ├── Tracking.Api/           # Minimal API service
│   ├── Notification.Service/   # Notification consumer service
│   └── Shared/                 # Shared models, DTOs, events
├── docker-compose.yml
├── .gitignore
└── README.md
```

## API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | /shipments | Tạo bưu gửi mới |
| GET | /shipments/{trackingCode} | Tra cứu trạng thái hiện tại |
| POST | /shipments/{trackingCode}/events | Cập nhật sự kiện/trạng thái |
| GET | /shipments/{trackingCode}/history | Tra cứu lịch sử hành trình |

## Trạng thái bưu gửi

- `Created` - Đã tạo
- `InTransit` - Đang vận chuyển
- `AtHub` - Tại trung tâm
- `Delivered` - Đã giao
- `Failed` - Thất bại

## Yêu cầu hệ thống

- .NET 10 SDK
- Docker & Docker Compose
- RabbitMQ (chạy qua Docker)

## Chạy dự án

```bash
# Build và chạy tất cả services
docker-compose up --build

# Chạy ở background
docker-compose up -d --build
```

## Cấu hình môi trường

Tạo file `.env` từ mẫu:

```bash
cp .env.example .env
```

Sau đó chỉnh sửa các giá trị phù hợp với môi trường local.

## Phát triển

```bash
# Restore packages
dotnet restore

# Build solution
dotnet build

# Chạy tests
dotnet test
```

## License

Dự án bài tập lớn môn Phát triển ứng dụng phân tán.