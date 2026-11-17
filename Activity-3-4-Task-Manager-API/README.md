# Task Manager API — Activities 3 & 4

**درس:** مباحث ویژه  
**فعالیت ۳:** طراحی Resourceها، Users و Tasks، تعریف Endpointها و HTTP Methodها، تعیین Status Codeها و رسم دیاگرام معماری.  
**فعالیت ۴:** Containerize کردن Back-End، ساخت Dockerfile و `.dockerignore`، Build کردن Image، اجرای Container و Port Mapping.

این پروژه هر دو فعالیت را در یک برنامه واحد نگه می‌دارد. هیچ‌یک از خواسته‌های اصلی فعالیت‌ها حذف نشده‌اند؛ امکانات تکمیلی مانند اعتبارسنجی، Pagination، فیلتر، تست و Healthcheck نیز اضافه شده‌اند.

## 1. معرفی و Resourceها

این برنامه یک REST API ساده برای مدیریت کاربران و وظایف آن‌هاست.

- **Users:** اطلاعات کاربران شامل `id`، `name` و `email`.
- **Tasks:** وظایف شامل `id`، `title`، `description`، `userId` و `completed`.
- هر Task با `userId` به یک User متصل است.

شناسه‌ها از نوع UUID هستند. برای ساده نگه داشتن تمرین، داده‌ها در حافظه (in-memory) نگهداری می‌شوند؛ بنابراین با راه‌اندازی مجدد برنامه، تغییرات ایجادشده در زمان اجرا باقی نمی‌مانند. این نسخه از پایگاه داده دائمی استفاده نمی‌کند.

## 2. API Contract — فعالیت ۳

تمام درخواست‌ها و پاسخ‌های بدنه‌دار با JSON هستند. مسیرهای API دارای پیشوند `/api` هستند.

### 2.1 Users Endpoints

| Method | Endpoint | کاربرد | Request Body | Success | خطاهای اصلی |
|---|---|---|---|---|---|
| GET | `/api/users` | دریافت فهرست کاربران | ندارد؛ Query اختیاری `page`, `limit` | 200 | 400 |
| GET | `/api/users/:id` | دریافت یک کاربر | ندارد | 200 | 400, 404 |
| POST | `/api/users` | ایجاد کاربر | `name`, `email` | 201 | 400 |
| PUT | `/api/users/:id` | جایگزینی کامل اطلاعات قابل ویرایش کاربر | `name`, `email` | 200 | 400, 404 |
| PATCH | `/api/users/:id` | ویرایش بخشی از اطلاعات کاربر | یکی از `name`, `email` یا هر دو | 200 | 400, 404 |
| DELETE | `/api/users/:id` | حذف کاربر بدون وظیفه وابسته | ندارد | 200 | 400, 404, 409 |
| GET | `/api/users/:id/tasks` | دریافت وظایف یک کاربر | ندارد | 200 | 400, 404 |

### 2.2 Tasks Endpoints

| Method | Endpoint | کاربرد | Request Body / Query | Success | خطاهای اصلی |
|---|---|---|---|---|---|
| GET | `/api/tasks` | دریافت فهرست وظایف | Query اختیاری `page`, `limit`, `userId` | 200 | 400, 404 |
| GET | `/api/tasks/:id` | دریافت یک وظیفه | ندارد | 200 | 400, 404 |
| POST | `/api/tasks` | ایجاد وظیفه | `title`, `userId`؛ `description`, `completed` اختیاری | 201 | 400 |
| PUT | `/api/tasks/:id` | جایگزینی کامل اطلاعات قابل ویرایش وظیفه | `title`, `userId`؛ `description`, `completed` اختیاری با مقدار پیش‌فرض | 200 | 400, 404 |
| PATCH | `/api/tasks/:id` | ویرایش بخشی از وظیفه | هر ترکیبی از `title`, `description`, `userId`, `completed` | 200 | 400, 404 |
| DELETE | `/api/tasks/:id` | حذف وظیفه | ندارد | 200 | 400, 404 |

**توضیح PUT و PATCH:** در این پروژه PUT اطلاعات قابل ویرایش منبع را به‌طور کامل جایگزین می‌کند؛ PATCH فقط فیلدهای ارسال‌شده را تغییر می‌دهد. هر دو برای شفافیت تفاوت این دو Method ارائه شده‌اند.

### 2.3 نمونه Requestها

ساخت کاربر:

```json
{
  "name": "Reza",
  "email": "reza@example.com"
}
```

ساخت وظیفه (برای نمونه از UUID کاربر موجود در پاسخ GET استفاده کنید):

```json
{
  "title": "تمرین Docker",
  "description": "ساخت Image و اجرای Container",
  "userId": "11111111-1111-4111-8111-111111111111",
  "completed": false
}
```

ویرایش بخشی از وظیفه:

```json
{
  "completed": true
}
```

### 2.4 Status Codeها

| Code | نام | کاربرد در پروژه |
|---|---|---|
| 200 | OK | دریافت، ویرایش یا حذف موفق |
| 201 | Created | ایجاد موفق User یا Task |
| 400 | Bad Request | JSON، شناسه، Pagination یا داده ورودی نامعتبر |
| 404 | Not Found | مسیر، User یا Task پیدا نشد |
| 409 | Conflict | حذف کاربری که هنوز Task وابسته دارد |
| 413 | Payload Too Large | بدنه درخواست بزرگ‌تر از حد مجاز |
| 429 | Too Many Requests | عبور از محدودیت تعداد درخواست |
| 500 | Internal Server Error | خطای پیش‌بینی‌نشده سرور |

### 2.5 قالب پاسخ‌ها

پاسخ موفق:

```json
{
  "success": true,
  "data": {}
}
```

پاسخ خطا:

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User not found"
  }
}
```

### 2.6 قواعد اعتبارسنجی

- نام کاربر و عنوان وظیفه باید رشته غیرخالی باشند؛ فاصله‌های ابتدا و انتها حذف می‌شوند.
- ایمیل باید قالب معتبر داشته باشد و تکراری نباشد.
- شناسه‌ها UUID معتبر هستند؛ شناسه با قالب اشتباه `400` و شناسه معتبرِ پیدا‌نشده `404` می‌گیرد.
- `completed` فقط Boolean واقعی (`true` یا `false`) می‌پذیرد؛ رشته‌های متنی پذیرفته نمی‌شوند.
- `userId` وظیفه باید به کاربر موجود اشاره کند.
- کاربری که Task دارد حذف نمی‌شود و پاسخ `409 Conflict` دریافت می‌کند.
- Pagination با `page` مثبت و `limit` بین 1 تا 100 انجام می‌شود.

## 3. دیاگرام معماری — فعالیت ۳

```text
                  +-------------------------+
                  | Client / Postman / Web  |
                  +------------+------------+
                               |
                         HTTP + JSON
                               |
                               v
                  +-------------------------+
                  | Docker Host :8080       |
                  | Port Mapping 8080:3000  |
                  +------------+------------+
                               |
                               v
             +--------------------------------------+
             | Docker Container                    |
             | Node.js + Express :3000              |
             | Helmet + Rate Limiting + JSON Parser |
             +------------------+-------------------+
                                |
                     +----------+----------+
                     |                     |
                     v                     v
             +---------------+     +---------------+
             | Users Routes  |     | Tasks Routes  |
             | /api/users   |     | /api/tasks    |
             +-------+-------+     +-------+-------+
                     |                     |
                     v                     v
             +---------------+     +---------------+
             | Users Store   |<--->| Tasks Store   |
             | In-Memory     | userId | In-Memory   |
             +---------------+     +---------------+

             Healthcheck: GET /health
```

## 4. ساختار فایل‌ها

```text
Activity-3-4-Task-Manager-API/
├── app.js
├── server.js
├── package.json
├── Dockerfile
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── src/
│   ├── data/
│   │   └── store.js
│   ├── middleware/
│   │   └── errors.js
│   └── routes/
│       ├── users.js
│       └── tasks.js
├── tests/
│   └── api.test.js
└── screenshots/
    └── (تصاویر اجرای واقعی را پس از آزمایش اضافه کنید)
```

## 5. اجرای محلی

پیش‌نیازها: Node.js نسخه 22 یا بالاتر و npm.

```bash
npm install
npm start
```

سرور روی پورت 3000 اجرا می‌شود.

آزمایش در مرورگر یا Postman:

- `GET http://localhost:3000/`
- `GET http://localhost:3000/health`
- `GET http://localhost:3000/api/users`
- `GET http://localhost:3000/api/tasks`

نمونه فیلتر و صفحه‌بندی:

- `GET http://localhost:3000/api/users?page=1&limit=10`
- `GET http://localhost:3000/api/tasks?userId=11111111-1111-4111-8111-111111111111`

## 6. اجرای تست‌ها

```bash
npm test
```

تست‌های خودکار موارد زیر را بررسی می‌کنند: دریافت کاربران، اعتبارسنجی نام و ایمیل، جلوگیری از ایمیل تکراری، نوع Boolean، جلوگیری از حذف کاربر دارای وظیفه، امکان حذف کاربر بدون وظیفه، رفتار PUT و PATCH، فیلتر وظایف، شناسه نامعتبر، JSON خراب، مسیر ناموجود، Pagination نامعتبر و Healthcheck.

## 7. Containerize کردن Back-End — فعالیت ۴

### 7.1 Dockerfile

Dockerfile از Image رسمی Node.js 22 Alpine استفاده می‌کند، وابستگی‌های تولیدی تعریف‌شده در `package.json` را نصب می‌کند، برنامه را با کاربر غیر Root اجرا می‌کند و Healthcheck دارد. نصب وابستگی‌ها در حال حاضر از `package.json` انجام می‌شود؛ برای Build کاملاً تکرارپذیر در نسخه بعدی باید `package-lock.json` نیز به مخزن اضافه و دستور نصب به `npm ci` تغییر داده شود. فایل `.dockerignore` پوشه تست‌ها و فایل‌های محلی غیرضروری را از Build Context کنار می‌گذارد.

### 7.2 .dockerignore

فایل `.dockerignore` مواردی مانند `node_modules`، فایل‌های محیطی، پوشه Git، تست‌ها و تصاویر محلی را از Build Context کنار می‌گذارد تا Image غیرضروری بزرگ نشود. فایل `.env.example` برای مستندسازی نمونه متغیرها نگه داشته می‌شود؛ فایل واقعی `.env` نباید وارد Image شود.

### 7.3 ساخت Image

از داخل پوشه اصلی پروژه اجرا کنید:

```bash
docker build -t task-manager-api:1.0 .
```

معنی دستور:
- `docker build`: ساخت Image
- `-t`: تعیین نام و Tag
- `task-manager-api:1.0`: نام و نسخه Image
- `.`: استفاده از پوشه فعلی به‌عنوان Build Context

### 7.4 اجرای Container و Port Mapping

```bash
docker run -d --name task-manager-container -p 8080:3000 task-manager-api:1.0
```

- `-d`: اجرای پس‌زمینه
- `--name`: نام Container
- `-p 8080:3000`: اتصال پورت 8080 میزبان به پورت 3000 داخل Container
- `task-manager-api:1.0`: Image مورد استفاده

پس از اجرا، آدرس میزبان:

`http://localhost:8080`

Endpointهای آزمایش:

- `GET http://localhost:8080/`
- `GET http://localhost:8080/health`
- `GET http://localhost:8080/api/users`
- `GET http://localhost:8080/api/tasks`

### 7.5 بررسی وضعیت Container

```bash
docker ps
docker logs task-manager-container
docker inspect --format='{{.State.Health.Status}}' task-manager-container
```

در `docker ps` باید Container در وضعیت Up دیده شود. پس از گذشت زمان شروع Healthcheck، وضعیت سلامت باید `healthy` شود.

### 7.6 توقف، حذف و اجرای مجدد

```bash
docker stop task-manager-container
docker rm task-manager-container
docker run -d --name task-manager-container -p 8080:3000 task-manager-api:1.0
```

اگر نام Container از قبل وجود داشته باشد، ابتدا آن را متوقف و حذف کنید یا نام جدید انتخاب کنید.

## 8. محدودیت‌ها و ملاحظات

- داده‌ها در حافظه هستند؛ برای نگهداری دائمی باید پایگاه داده اضافه شود.
- این نسخه احراز هویت و نقش‌های کاربری ندارد؛ بنابراین برای استفاده عمومی/تولیدی مناسب نیست.
- Rate limiting و Helmet لایه‌های پایه محافظتی هستند و جایگزین احراز هویت، کنترل دسترسی و بازبینی امنیتی نیستند.
- تست خودکار، رفتارهای اصلی API را بررسی می‌کند؛ با این حال اجرای واقعی Docker و گرفتن تصویر خروجی باید روی سیستم دارای Docker انجام شود.

## 9. چک‌لیست تحویل فعالیت‌ها

- [x] شناسایی Resourceهای پروژه
- [x] طراحی Users و Tasks Endpointها
- [x] تعیین HTTP Method و Status Code برای Endpointها
- [x] رسم دیاگرام معماری
- [x] آماده‌سازی Dockerfile و `.dockerignore`
- [x] آماده‌سازی دستور Build کردن Image
- [x] آماده‌سازی دستور اجرای Container و Port Mapping
- [x] مستندسازی README و دستورات بررسی Container
- [ ] اجرای واقعی Build روی سیستم تحویل‌دهنده
- [ ] اجرای واقعی Container و ثبت تصویر `docker ps`
- [ ] ثبت تصویر پاسخ موفق API و وضعیت Healthcheck

## 10. نتیجه

در فعالیت ۳، Resourceهای Users و Tasks، Endpointها، Methodها، Status Codeها و معماری API طراحی شده‌اند. در فعالیت ۴، Back-End با Docker قابل بسته‌بندی است؛ Image با دستور Build ساخته و Container با Port Mapping اجرا می‌شود. فایل README مراحل نصب، اجرا، تست و بررسی را توضیح می‌دهد.
