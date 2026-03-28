# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. Menurut aku, di kasus BambangShop ini sebenarnya nggak wajib pakai interface (trait) untuk Subscriber. Soalnya, di implementasi sekarang, Subscriber cuma berisi data sederhana seperti url dan name, dan behavior nya juga blom kompleks atau bervariasi gitu.

Kalau di teori Observer Pattern, Subscriber dibuat sebagai interface supaya fleksibel (bisa bnyk jenis subscriber dengan behavior berbeda). Tapi di kasus ini, semua subscriber "berperilaku sama", yaitu menerima notifikasi lewat HTTP request. Jadi, pakai satu struct Subscriber aja sudah cukup.

Tapi kalau ke depannya ada banyak jenis subscriber dengan cara menerima notifikasi yang berbeda-beda, baru deh trait bisa dipakai supaya lebih fleksibel dan sesuai prinsip Open-Closed.

2. Menurut aku, penggunaan Vec sebenarnya kurang cocok untuk kasus ini. Soalnya id di Program dan url di Subscriber itu harus unik.

Kalau pakai Vec, kita bakal harus:
- ngecek manual apakah data sudah ada atau belum
- pencarian jadi lebih lambat (O(n))

Sedangkan kalau pakai DashMap (atau HashMap):
- key langsung unik (otomatis enforce uniqueness)
- akses data lebih cepat (O(1))
- lebih efisien untuk operasi add, delete, dan lookup

Jadi menurut aku, penggunaan DashMap itu lebih tepat dan efisien dibanding Vec untuk kasus ini.

3. Menurut aku, Singleton dan DashMap itu beda tujuan, jadi nggak bisa saling menggantikan.

- Singleton --> memastikan cuma ada satu instance (misalnya satu database global)
- DashMap --> memastikan data bisa diakses secara thread-safe

Di kasus ini, kita memang butuh:
- Satu instance global (sudah dipenuhi dengan lazy_static, mirip Singleton)
- Thread-safe access (ini yang dipenuhi oleh DashMap)

Jadi walaupun kita pakai konsep Singleton, tetap butuh DashMap untuk handle concurrency. Kalau cuma pakai Singleton tanpa DashMap, bisa terjadi race condition saat banyak request masuk bersamaan.

Kesimpulannya, kita tetap butuh DashMap, dan Singleton saja tidak cukup.

#### Reflection Publisher-2
1. Menurut aku, walaupun di konsep MVC klasik itu Model nya mencakup data + logic, tapi di praktik modern kita sekarang ini itu biasanya memisahkan Service dan Repository dari Model supaya kode nya itu jadi lebih rapi dan mudah dikelola gitu.

Kalau semua logic dimasukin ke Model, nanti Model jadi terlalu "gemuk" (fat model) gitu, isinya campur antara:
- struktur data
- logic bisnis
- akses database

Dengan dipisah:
- Model --> cuma representasi data
- Repository --> fokus ke data storage (CRUD)
- Service --> fokus ke business logic

Ini sesuai dengan prinsip Single Responsibility Principle, jadi tiap bagian punya tanggung jawab yang jelas. Selain itu, jadi lebih mudah untuk testing, maintenance, dan kalau mau ganti implementasi (misalnya ganti database), gak perlu ubah semua bagian.

2. Kalau kita cuman pake Model doang, menurut aku kode nya bakal jadi lebih kompleks dan susah di maintain.

Misalnya:
- Program harus tahu cara kirim notifikasi
- Subscriber harus tahu cara subscribe/unsubscribe
- Notification harus tahu cara akses database dan kirim request

Akhirnya semua model jadi saling bergantung satu sama lain (tight coupling). Kalau ada perubahan di satu model, bisa berdampak ke banyak model lain.

Selain itu:
- logic jadi tersebar dan campur aduk
- susah dibaca
- susah debug

Dengan adanya Service dan Repository ini itu, interaksi antar model nya bakal jadi lebih terstruktur:

Controller --> Service --> Repository --> Model

Jadi tiap bagian itu punya peran yg jelas dan gk saling "overlap" gituu.

3. Menurut aku, Postman ini itu sangat membantu banget buat ngetes API yg lagi aku buat.

Dengan Postman, aku bisa:
- kirim request HTTP (GET, POST, dll)
- ngirim body JSON dgn mudah
- lihat response langsung tanpa perlu frontend

Untuk project ini, Postman membantu aku buat:
- test endpoint /notification/subscribe
- test /unsubscribe
- memastikan API udh jalan sesuai yg diharapkan

Fitur Postman yg menurut aku paling berguna:
- Collection --> bisa simpan semua endpoint jadi rapi
- Environment variables --> bisa ganti base URL (localhost, dll)
- Pretty JSON viewer --> response jadi lebih mudah dibaca
- Save request --> gk perlu ngetik ulang terus

Ke depannya, menurut aku Postman bakal sangat kepakai di:
- group project
- testing API sebelum frontend jadi
- debugging kalau ada error di backend

#### Reflection Publisher-3
1. Menurut aku, di tutorial ini kita bakal pakai Push Model.

Soalnya, dari implementasinya kelihatan kalau publisher (BambangShop) langsung ngirim data notifikasi ke subscriber lewat HTTP POST. Jadi subscriber gk perlu minta data sendiri, tapi langsung "didorong" (push) oleh publisher setiap ada event seperti create, delete, atau promotion.

2. Kalau kita bayangin pakai Pull Model gituu, artinya subscriber itu harus ambil data sendiri dari publisher nya (misalnya nge request API secara berkala gitu).

Menurut aku:

Kelebihan Pull Model:
- Lebih fleksibel --> subscriber bisa ambil data sesuai kebutuhan
- Publisher jadi lebih sederhana (gak perlu kirim ke semua subscriber)
- Bisa mengurangi beban publisher kalau subscriber banyak

Kekurangan Pull Model:
- Tidak real-time --> harus polling (ngecek berkala)
- Lebih boros resource di subscriber (karena request terus-menerus)
- Implementasi lebih ribet (harus handle kapan fetch, caching, dll)

Sedangkan di kasus ini, menurut aku Push Model lebih cocok karena:
- Notifikasi jadi langsung (real time)
- Lebih simpel implementasinya
- Cocok untuk event-based system kayak ini

3. Kalau kita tidak pakai multi-threading, menurut aku program bakal jadi lebih lambat dan kurang efisien.

Soalnya sekarang setiap notifikasi dikirim pakai:
- thread::spawn(...)

Artinya:
- tiap subscriber dikirim notif secara paralel

Kalau tanpa multi-threading:
- notifikasi dikirim satu per satu (sequential)
- kalau ada banyak subscriber, bakal lamaa banget
- kalau satu subscriber lambat (misalnya server down), semua proses ikut ke block

Dampaknya:
- response API jadi lebih lama
- performa turun
- user experience jadi jelek

Jadi menurut aku, multi-threading di sini itu penting supaya:
- pengiriman notif bisa paralel
- lebih cepat
- tidak blocking proses utama