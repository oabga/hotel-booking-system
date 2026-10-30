# Gradle là gì — giải thích chi tiết

## 1. Gradle vs Maven — "đối thủ" đúng, nhưng Maven Central không phải là Maven

Bạn nói đúng: **Gradle và Maven là hai công cụ build cạnh tranh nhau**, giống như React vs Vue, hay MySQL vs PostgreSQL — cùng giải quyết 1 bài toán (build project Java), khác cách làm.

Nhưng có một điểm dễ nhầm: **Maven Central** không phải là "Maven" công cụ build. Đây là hai khái niệm tách biệt dù chung chữ "Maven":

| | Maven (build tool) | Maven Central (repository) |
|---|---|---|
| Là gì | Một công cụ build, đối thủ của Gradle | Một **kho lưu trữ** (giống "App Store" cho thư viện Java) |
| Dùng để | Compile code, chạy test, build project — theo file `pom.xml` | Chỉ để **chứa và phân phát** các thư viện `.jar` đã được publish |
| Ai dùng | Project nào chọn Maven làm build tool | **Mọi công cụ build** (Gradle, Maven, sbt, Ant...) đều có thể tải thư viện từ đây |

Lịch sử: Maven Central được chính team Maven tạo ra ban đầu để công cụ Maven tải thư viện. Nhưng vì nó trở thành kho công khai lớn nhất, gần như **chuẩn chung của toàn cộng đồng Java**, nên các công cụ build khác — bao gồm Gradle — cũng tải thư viện từ chính kho này. Gradle "cạnh tranh" với Maven ở vai trò *công cụ build*, nhưng cả hai đều "mượn" cùng một kho thư viện chung đó.

Ví dụ dễ hình dung: Maven Central giống như **npm registry** trong thế giới JavaScript — dù bạn dùng npm, yarn hay pnpm (3 công cụ khác nhau, cạnh tranh nhau) thì cả 3 đều tải package từ cùng kho npm registry. Gradle/Maven và Maven Central cũng là mối quan hệ tương tự.

## 2. Gradle là gì, chính xác

Gradle là một **công cụ build tự động (build automation tool)** cho project phần mềm (không chỉ Java — còn dùng cho Android, Kotlin, C++...). "Build" ở đây nghĩa là toàn bộ quá trình biến mã nguồn `.java` thành một chương trình chạy được, bao gồm:
- Tải các thư viện bên ngoài mà code cần dùng.
- Biên dịch (`compile`) mã nguồn.
- Chạy test tự động.
- Đóng gói (`package`) thành file chạy được (`.jar`, `.war`...).

Gradle **không phải**:
- Một ngôn ngữ lập trình.
- Một thư viện bạn import vào code.
- Một IDE (IntelliJ chỉ là nơi *tích hợp* Gradle cho tiện, bạn vẫn dùng Gradle được từ terminal, không cần IntelliJ).

## 3. Cấu trúc thư mục chuẩn Gradle mong đợi

```
my-project/
├── build.gradle          ← file cấu hình chính của project
├── settings.gradle       ← khai tên project (và các sub-project nếu có nhiều module)
├── gradlew               ← script "Gradle Wrapper" (xem mục 6)
├── gradlew.bat
└── src/
    ├── main/
    │   └── java/         ← code thật của ứng dụng
    └── test/
        └── java/         ← code unit test
```
Gradle tự nhận biết quy ước thư mục này — bạn không cần khai báo "code nằm ở đâu", chỉ cần đặt đúng chỗ.

## 4. File `build.gradle` — từng dòng nghĩa là gì

```groovy
plugins {
    id 'java'
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
}
```

- **`plugins { id 'java' }`** — báo cho Gradle: "đây là project Java". Gradle sẽ tự nạp sẵn các task chuẩn: compile, test, build jar...
- **`java { toolchain { languageVersion = ... } }`** — chỉ định dùng JDK bản nào (ở đây là 21) để compile và chạy. Nếu máy bạn chưa có đúng bản, Gradle có thể tự tải JDK đó về (tùy cấu hình).
- **`repositories { mavenCentral() }`** — khai "khi cần tải thư viện, tìm ở kho Maven Central" (xem mục 1). Có thể khai thêm nhiều kho khác nếu cần (ví dụ kho riêng của công ty).
- **`dependencies { ... }`** — danh sách thư viện bên ngoài project cần dùng. Mỗi dòng có dạng `group:artifact:version`, ví dụ `org.junit.jupiter:junit-jupiter:5.10.0` nghĩa là "thư viện JUnit Jupiter, version 5.10.0, do nhóm org.junit.jupiter publish". Gradle đọc dòng này, tự lên Maven Central tìm đúng file `.jar` đó, tải về, gắn vào project — bạn không cần tự tay download file nào.
  - Tiền tố `testImplementation` nghĩa là "chỉ cần thư viện này khi compile/chạy test", khác với `implementation` (cần cho cả code chính lúc chạy thật).

## 5. Gradle thực sự "làm" gì khi chạy

Gradle tổ chức công việc thành các **task** (nhiệm vụ nhỏ), mỗi task phụ thuộc task khác. Khi bạn gõ:
```
./gradlew build
```
Gradle tự chạy một dây chuyền, ví dụ:
1. `compileJava` — gọi trình biên dịch Java, biên dịch toàn bộ file trong `src/main/java` thành `.class`.
2. `compileTestJava` — biên dịch code trong `src/test/java`.
3. `test` — chạy toàn bộ unit test, báo pass/fail. Nếu test fail, `build` sẽ dừng lại báo lỗi.
4. `jar` — đóng gói các file `.class` đã compile thành 1 file `.jar` chạy được.

Bạn không gọi từng bước tay — Gradle tự biết thứ tự phụ thuộc, và biết **bỏ qua bước nào không cần làm lại** (ví dụ file nào không đổi thì không compile lại — gọi là "incremental build", giúp các lần build sau nhanh hơn nhiều).

Một số task hay dùng khác:
- `./gradlew test` — chỉ chạy test, không build jar.
- `./gradlew run` — chạy thử ứng dụng (nếu có plugin `application`).
- `./gradlew clean` — xóa hết file build cũ (thư mục `build/`), để build lại từ đầu.

## 6. `gradlew` (Gradle Wrapper) là gì

Khi tạo project Gradle, luôn có kèm 2 file `gradlew` (Linux/Mac) và `gradlew.bat` (Windows) cùng thư mục `gradle/wrapper/`. Đây gọi là **Gradle Wrapper** — một script nhỏ giúp:
- Bạn **không cần tự cài Gradle** trước lên máy. Lần đầu chạy `./gradlew ...`, script tự tải đúng version Gradle mà project yêu cầu (ghi trong `gradle/wrapper/gradle-wrapper.properties`) về một chỗ cache, rồi dùng bản đó.
- Đảm bảo **mọi người trong team dùng đúng cùng 1 version Gradle** — tránh lỗi kiểu "build chạy được trên máy tôi nhưng lỗi trên máy bạn" do khác version.

Vì vậy quy ước chung: luôn chạy `./gradlew ...`, không chạy lệnh `gradle ...` trực tiếp (trừ khi bạn biết rõ mình đang làm gì).

## 7. Vì sao không chỉ cần `javac` cho đơn giản

Với 1-2 file nhỏ, `javac` tay là đủ. Nhưng ứng dụng thật như hệ thống hotel booking sẽ có:
- Hàng chục/hàng trăm file `.java` phụ thuộc lẫn nhau theo thứ tự phức tạp.
- Nhiều thư viện ngoài (Spring Boot, Flyway, driver kết nối database, JUnit...) với version cụ thể cần quản lý.
- Cần đóng gói ra 1 file để deploy lên server.
- Cần chạy test tự động mỗi lần build (CI/CD).

Gradle (hoặc Maven) giải quyết toàn bộ bằng vài dòng khai báo khai báo trong `build.gradle`, đó là lý do mọi project Spring Boot thực tế đều dùng Gradle hoặc Maven, không ai gọi `javac` tay cho project thật.

## Tóm tắt nhanh
- Gradle = công cụ tự động hóa việc build (compile, test, đóng gói) project.
- `build.gradle` = nơi khai báo "dùng JDK nào, cần thư viện gì, lấy thư viện từ kho nào".
- Maven Central = kho thư viện chung của cộng đồng Java, không phải là công cụ Maven — Gradle cũng tải thư viện từ đây.
- `gradlew` = script đảm bảo ai mở project cũng dùng đúng version Gradle, không cần tự cài Gradle trước.
