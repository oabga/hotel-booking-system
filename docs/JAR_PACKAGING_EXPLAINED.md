# File `.jar` là gì, và cách Gradle đóng gói project

## 0. Ba loại đuôi file bạn sẽ gặp: `.java`, `.class`, `.jar`

Đây là 3 "dạng sống" khác nhau của cùng một đoạn code, theo đúng thứ tự biến đổi:

```
Room.java  ──(trình biên dịch javac)──▶  Room.class  ──(gói lại bằng zip)──▶  app.jar
 (code                                    (bytecode,                          (nhiều .class
  con người                                máy JVM                             gói chung
  đọc được)                                đọc được)                           + metadata)
```

| Đuôi file | Là gì | Ai đọc được | Khi nào xuất hiện |
|---|---|---|---|
| `.java` | **Mã nguồn** — code bạn tự viết bằng tay, chữ, cú pháp Java thông thường | Con người đọc trực tiếp được | Ngay khi bạn tạo file, viết code |
| `.class` | **Bytecode** — kết quả sau khi `javac` biên dịch 1 file `.java`. Không còn là text thường, là dạng nhị phân (mở bằng text editor sẽ thấy toàn ký tự lạ) | Chỉ JVM (Java Virtual Machine) đọc và thực thi được, con người không đọc trực tiếp | Sau khi compile (bấm Run trong IntelliJ, hoặc chạy `javac`, hoặc Gradle task `compileJava`) |
| `.jar` | **Gói lưu trữ** — nhiều file `.class` (có thể hàng trăm, từ nhiều class khác nhau, kể cả từ thư viện ngoài) được nén chung vào 1 file zip, kèm thêm metadata | Công cụ `java -jar` đọc để biết chạy từ đâu | Sau bước "đóng gói" (Gradle task `jar`/`bootJar`), dùng để phân phát/deploy |

Quy tắc số lượng: **1 file `.java` → thường ra 1 (hoặc nhiều, nếu có inner class) file `.class`**. Rồi **nhiều file `.class`** (của toàn bộ project, cộng thêm `.class` của các thư viện ngoài nếu là fat jar) **→ được gói chung vào 1 file `.jar` duy nhất**.

Ví dụ cụ thể với project của bạn:
```
src/main/java/com/hotel/Room.java      ← bạn viết tay
src/main/java/com/hotel/Booking.java   ← bạn viết tay
src/main/java/com/hotel/Main.java      ← bạn viết tay

        │  compile (javac / Gradle compileJava)
        ▼

build/classes/com/hotel/Room.class     ← JVM đọc được, con người không đọc trực tiếp
build/classes/com/hotel/Booking.class
build/classes/com/hotel/Main.class

        │  đóng gói (Gradle jar/bootJar)
        ▼

build/libs/app.jar                     ← 1 file duy nhất, chứa cả 3 .class trên
```

**Lưu ý nhỏ dễ nhầm:** khi bạn bấm nút "Run" màu xanh trong IntelliJ để chạy thử `Main.java`, IntelliJ **âm thầm làm bước compile thành `.class` trước** rồi mới chạy — bạn không thấy bước đó vì IDE giấu đi cho tiện, chứ không phải nó chạy trực tiếp file `.java`. Thư mục chứa các file `.class` sinh ra này thường là `out/` (nếu IntelliJ tự quản) hoặc `build/classes/` (nếu dùng Gradle).

## 1. `.jar` là gì, về bản chất

`.jar` (Java ARchive) chỉ là **1 file `.zip` đổi tên đuôi**, chứa:
- Các file `.class` (code Java đã biên dịch, không còn là `.java` nữa).
- 1 file đặc biệt `META-INF/MANIFEST.MF` — ghi chú metadata, trong đó quan trọng nhất là dòng `Main-Class: ...` nói cho Java biết "nếu chạy file jar này, hãy bắt đầu chạy từ class nào".

Bạn có thể tự kiểm chứng: đổi đuôi 1 file `.jar` thành `.zip` rồi giải nén bằng công cụ zip thường — sẽ thấy đúng là các file `.class` nằm trong đó.

```
app.jar  (thực chất là 1 file zip)
├── META-INF/
│   └── MANIFEST.MF        ← ghi "Main-Class: com.hotel.Main"
├── com/
│   └── hotel/
│       ├── Main.class
│       ├── Room.class
│       └── Booking.class
```

Chạy `java -jar app.jar` → Java mở file zip này, đọc `MANIFEST.MF` biết chạy class nào, rồi thực thi.

## 2. Vấn đề: jar "thường" không chứa thư viện ngoài

Giả sử code của bạn có dùng JUnit hoặc sau này Spring Boot (thư viện ngoài, khai trong `dependencies` của `build.gradle`). Khi Gradle build jar "thường" (plugin `java` mặc định), nó **chỉ gói code của chính bạn**, KHÔNG gói thư viện ngoài vào trong:

```
app.jar  (jar "thường" — thin jar)
├── META-INF/MANIFEST.MF
└── com/hotel/*.class        ← chỉ có code của bạn

Thư viện ngoài (JUnit, Spring...) KHÔNG nằm trong file này!
```

Nếu đem file này chạy ở máy khác (không có sẵn thư viện đó trong classpath) → lỗi `java.lang.ClassNotFoundException` hoặc `NoClassDefFoundError`, vì Java tìm không thấy code của thư viện.

Đây là lý do khi chạy ở máy dev, IntelliJ "chạy được" — vì IntelliJ tự thêm hết các thư viện đã tải vào classpath khi chạy giúp bạn. Nhưng nếu chỉ lấy đúng file `app.jar` đó mang đi máy khác chạy độc lập, sẽ lỗi.

## 3. Giải pháp: "Fat jar" / "Executable jar" — gói tất cả vào 1 file

Để 1 file `.jar` tự chạy được độc lập ở bất kỳ máy nào (chỉ cần có Java), người ta gói luôn **cả thư viện ngoài vào trong cùng file jar đó**. Đây gọi là **fat jar** (hay "uber jar", "executable jar").

```
app.jar  (FAT jar — tự chạy độc lập được)
├── META-INF/MANIFEST.MF
├── com/hotel/*.class              ← code của bạn
└── BOOT-INF/lib/                  ← các thư viện ngoài, gói kèm luôn
    ├── spring-boot-3.x.jar
    ├── spring-web-6.x.jar
    ├── tomcat-embed-core-....jar  ← cả 1 web server nhúng sẵn!
    └── ... (toàn bộ dependency khác)
```

Với project Spring Boot, plugin `org.springframework.boot` sẽ tự thay task `jar` mặc định để tạo ra đúng loại fat jar này — bên trong còn có **Tomcat nhúng sẵn** (embedded web server), nên file jar đó khi chạy `java -jar app.jar` sẽ tự khởi động luôn 1 web server REST API hoàn chỉnh, không cần cài Tomcat/server nào trên máy chạy.

## 4. Sơ đồ toàn bộ pipeline: từ code đến chạy được

```
 src/main/java/*.java                 (code bạn viết)
          │
          │  Gradle task: compileJava  (gọi javac)
          ▼
 build/classes/*.class                (code đã biên dịch)
          │
          │  Gradle task: test          (chạy unit test, nếu fail thì dừng)
          ▼
      [ test pass ]
          │
          │  Gradle task: jar / bootJar (đóng gói)
          ▼
 build/libs/app.jar                   (file jar — fat jar nếu Spring Boot)
          │
          │  lệnh: java -jar app.jar
          ▼
   Ứng dụng đang CHẠY
   (nếu là Spring Boot: 1 web server REST API
    đang lắng nghe ở port 8080, sẵn sàng nhận request)
```

So sánh 2 task đóng gói trong Gradle:

| Task | Plugin | Kết quả | Chạy độc lập được? |
|---|---|---|---|
| `jar` | `plugins { id 'java' }` | Thin jar — chỉ code của bạn | Không (thiếu thư viện ngoài) |
| `bootJar` | `plugins { id 'org.springframework.boot' }` | Fat jar — code + toàn bộ thư viện + server nhúng | Có, chạy ngay bằng `java -jar` |

## 5. Vì sao điều này quan trọng với dự án hotel booking

Khi dự án build xong và cần **deploy lên server thật** (hoặc đóng gói vào Docker image), người ta sẽ dùng chính file fat jar (`bootJar`) này — copy 1 file duy nhất lên server, chạy `java -jar app.jar`, xong, không cần cài đặt gì thêm (không cần cài Tomcat, không cần cài từng thư viện). Đây là lý do Spring Boot rất phổ biến để deploy — gọn trong 1 file.

## Tóm tắt nhanh
- `.jar` = file zip chứa `.class` + metadata, chạy bằng `java -jar`.
- Jar "thường" chỉ chứa code của bạn — thiếu thư viện ngoài, không tự chạy độc lập được.
- "Fat jar" (Spring Boot gọi task này là `bootJar`) = gói luôn thư viện ngoài + server nhúng vào 1 file → tự chạy độc lập, dùng để deploy.
- Pipeline: `.java` → (`compileJava`) → `.class` → (`test`) → (`jar`/`bootJar`) → `app.jar` → (`java -jar`) → ứng dụng chạy thật.
