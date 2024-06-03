# Microservices.Tutorial.Outbox.Inbox.Design.Pattern

Microservices.Tutorial.Outbox.Inbox.Design.Pattern , sipariş yönetimi, stok yönetimi ve mesajlaşma süreçlerini idare eden bir .NET Core mikroservis uygulamasıdır. Bu proje, siparişlerin oluşturulması, güncellenmesi ve tamamlanması gibi işlemleri yönetir. Ayrıca, Outbox Pattern kullanarak tekrarlanabilir işlemleri güvenli bir şekilde gerçekleştirir ve RabbitMQ aracılığıyla servisler arasında mesajlaşma sağlar.


## Özellikler

- Sipariş oluşturma ve yönetimi
- Outbox Pattern ile idempotent işlemler
- RabbitMQ ile mesajlaşma
- Quartz.NET ile zamanlanmış görevler
- Swagger ile API dokümantasyonu

## Teknolojiler

- .NET 8 SDK
- Entity Framework Core
- Dapper
- MassTransit
- RabbitMQ
- Quartz.NET
- MSSQL Server

## Kurulum

### Gereksinimler

Bu projeyi çalıştırmak için aşağıdaki araçlar gereklidir:
- .NET 8 SDK
- MSSQL Server
- RabbitMQ

### Adımlar

1. Depoyu klonlayın:

    ```bash
    git clone https://github.com/kullaniciadi/order-management-system.git
    cd order-management-system
    ```

2. Gerekli bağımlılıkları yükleyin:

    ```bash
    dotnet restore
    ```

3. MSSQL Server ve RabbitMQ bağlantı ayarlarını `appsettings.json` dosyasında yapılandırın:

    ```json
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information",
          "Microsoft.AspNetCore": "Warning"
        }
      },
      "AllowedHosts": "*",
      "RabbitMQ": "amqps://your_rabbitmq_connection_string",
      "ConnectionStrings": {
        "MSSQLServer": "Server=localhost, 1433;Database=OrderDB;User ID=SA;Password=your_password;TrustServerCertificate=True"
      }
    }
    ```

4. Veritabanı migrasyonlarını uygulayın:

    ```bash
    dotnet ef database update -p Order.API
    ```

5. Uygulamayı çalıştırın:

    ```bash
    dotnet run --project Order.API
    dotnet run --project Order.Outbox.Table.Publisher.Service
    dotnet run --project Stock.Service
    ```

Opsiyonel olarak Docker kullanarak çalıştırabilirsiniz:

```bash
docker-compose up
```

### Kullanım

1. Sipariş Oluşturma
POST isteği ile /create-order endpointine sipariş bilgilerini gönderin:

```json
curl -X POST "https://localhost:5001/create-order" -H "Content-Type: application/json" -d '{
  "BuyerId": 1,
  "OrderItems": [
    {
      "ProductId": 1,
      "Count": 2,
      "Price": 10.5
    },
    {
      "ProductId": 2,
      "Count": 1,
      "Price": 20.0
    }
  ]
}'
```

2. Sipariş Oluşturma İş Akışı

~~a =>~~ Order.API'de /create-order endpointine istek gönderildiğinde, sipariş oluşturulur ve OrderOutbox tablosuna bir kayıt eklenir.
~~b =>~~ Order.Outbox.Table.Publisher.Service'de tanımlanan Quartz.NET job'ı, OrderOutbox tablosundaki yeni kayıtları kontrol eder ve RabbitMQ'ya OrderCreatedEvent mesajını gönderir.
~~c =>~~ Stock.Service bu mesajı tüketir ve stok bilgilerini günceller.

3. Mimari

Proje, mikroservis mimarisi ile tasarlanmıştır ve her bir servis kendi veritabanına sahiptir. Outbox Pattern kullanarak idempotent işlemler gerçekleştirilir ve RabbitMQ kullanılarak servisler arasında asenkron iletişim sağlanır. Quartz.NET ile zamanlanmış görevler gerçekleştirilir.

4. Proje Yapısı

# Order.API

## Contexts:

**OrderDbContext:** Veritabanı bağlamı ve DbSet tanımları içerir.
Entities:

**Order:** Sipariş bilgilerini temsil eder.
**OrderItem:**Sipariş kalemi bilgilerini temsil eder.
OrderOutbox: Outbox Pattern için kullanılır.

## ViewModels:

**CreateOrderVM:** Sipariş oluşturma için ViewModel.
**CreateOrderItemVM:** Sipariş kalemi oluşturma için ViewModel.
**Program.cs:** Uygulamanın başlangıç noktası. RabbitMQ ve Entity Framework Core yapılandırması yapılır.

# Order.Outbox.Table.Publisher.Service

## Entities:

**OrderOutbox:** Outbox Pattern için kullanılır.

## Jobs:

**OrderOutboxPublishJob:** Outbox Pattern ile oluşturulan kayıtları RabbitMQ'ya gönderir.
**OrderOutboxSingletonDatabase:** Singleton olarak kullanılan ve Dapper ile veritabanı işlemlerini gerçekleştiren sınıf.

**Program.cs:** Uygulamanın başlangıç noktası. Quartz.NET kullanılarak zamanlanmış görevler tanımlanır.

# Stock.Service

## Contexts:

**StockDbContext:** Veritabanı bağlamı ve DbSet tanımları içerir.
Entities:

**OrderInbox:** Idempotent işlemler için kullanılır.

## Consumers:

**OrderCreatedEventConsumer:** OrderCreatedEvent mesajlarını tüketir ve işleme alır.
**Program.cs:*** Uygulamanın başlangıç noktası. RabbitMQ'dan gelen mesajları tüketmek için MassTransit kullanılır.

# Shared

## Datas:

**OrderItem:** Sipariş kalemi bilgilerini içerir.

## Events:

**OrderCreatedEvent:** Sipariş oluşturulduğunda tetiklenen olay.
## Settings:

**RabbitMQSettings:** RabbitMQ ile ilgili ayarları içerir.
