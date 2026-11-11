# Лаборатори №5 — API систем тест (Postman ба Newman)

## Оюутан ба орчин

- Оюутны нэр: **Ундрах**
- Оюутны код: **B252270135**
- Node.js: `v18.20.8`
- Newman: `6.2.3`
- API: `http://localhost:3000`
- Сервер: Teams-ээс ирсэн `server.rar` доторх `server.js`; нэмэлт Node сан шаардлагагүй.

## Ажиллуулах

Төслийн хавтаст Newman dependency-г суулгаад серверээ тусдаа Terminal-д асаана:

```sh
npm install
node server.js
```

Дараа нь collection-уудыг ажиллуулна:

```sh
mkdir -p results
npx newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt
npx newman run lab05-collection-fail.json 2>&1 | tee results/newman-fail.txt
```

FAIL collection нь амжилттай registration-ийн хүлээгдэх HTTP кодыг зориудаар `200` болгосон. DOWN гаралтыг авахын өмнө серверийг `Ctrl+C`-ээр зогсоогоод PASS collection-ийг дахин ажиллуулна:

```sh
npx newman run lab05-collection.json 2>&1 | tee results/newman-down.txt
```

zsh дээр pipeline-ийн жинхэнэ exit code-ийг `tee`-ийн дараа `echo "exit=${PIPESTATUS[1]}"` гэж тэмдэглэж болно. Энэ системд PASS `0`, FAIL `1`, DOWN `1` гарах ёстой.

## Тест дизайны 5 алхам

1. **Бие даан тестлэгдэх функц:** `POST /registrations` оюутан хичээлийн урьдач нөхцөл хангаж байгаа эсэхийг шалгана.
2. **Сонголтууд:** Оюутны ID/төлөв, үзсэн хичээлүүд, course ID байгаа эсэх, урьдач хичээлийн жагсаалт.
3. **Төлөөлөх утгууд:** Доорх хүснэгтэд төлөв болон урьдач нөхцөлийн зөвшөөрөгдөх ангилал бүрээс жишээ сонгов.
4. **Спецификаци:** Хүлээгдэх HTTP status болон `result` кодыг 10 кейсээр тогтоов. Алдааг давхар өгөхөд сервер эхлээд оюутныг шалгадаг.
5. **Бодит тест кейс:** Collection-ийн кейс бүр өөрийн PUT setup-тай; бүртгэлийн дугаарыг яг тэнцүүгээр шалгахгүй, зөвхөн number төрөл эсэхийг шалгана.

### Сонголт ба төлөөлөх утга

| Хэмжээс | Сонголт | Төлөөлөх утга / кейс |
|---|---|---|
| `studentID`-гийн хүчинтэй байдал | Идэвхтэй, идэвхгүй, байхгүй | `B231234567`; `B231234568`; `NO-STUDENT-01` |
| Оюутны үзсэн хичээл | Шаардлагыг хангана / хангахгүй | `CS201` үзсэн / `CS301` үзээгүй |
| `courseID`-гийн хүчинтэй байдал | Байгаа / байхгүй | `CS313`; `NO-COURSE-01` |
| Урьдач нөхцөл | Бүгд үзсэн / огт үзээгүй / заримыг үзсэн | `CS201`; `CS301`; `{CS101, CS201}`-оос зөвхөн `CS201` |

### Спецификаци ба хүлээгдэх үр дүн

| # | Тестийн кейс | Төлөөлөх нөхцөл | HTTP | `result` |
|---:|---|---|---:|---|
| 1 | Happy path | Идэвхтэй оюутан, `CS201` prereq хангана | 201 | `OK` |
| 2 | Байхгүй оюутан | Хичээл байна, student ID байхгүй | 200 | `ERROR_NO_STUDENT` |
| 3 | Идэвхгүй оюутан | Хичээл байна, student inactive | 200 | `ERROR_INACTIVE_STUDENT` |
| 4 | Байхгүй хичээл | Student active, course ID байхгүй | 200 | `ERROR_NO_COURSE` |
| 5 | Урьдачийг огт үзээгүй | Course нэг prereq-тэй, student үзээгүй | 200 | `ERROR_PREREQUISITES` |
| 6 | Урьдачийн заримыг үзсэн | Хоёр prereq-ээс нэгийг үзсэн | 200 | `ERROR_PREREQUISITES` |
| 7 | Давхар алдаа | Идэвхгүй student + байхгүй course; student check түрүүлнэ | 200 | `ERROR_INACTIVE_STUDENT` |
| 8 | Давхар алдаа | Байхгүй student + байхгүй course; student lookup түрүүлнэ | 200 | `ERROR_NO_STUDENT` |
| 9 | Хязгаар | `studentID` талбар дутуу | 400 | `ERROR_BAD_REQUEST` |
| 10 | Хязгаар | `courseID` талбар дутуу | 400 | `ERROR_BAD_REQUEST` |

## Newman үр дүн

- `results/newman-pass.txt`: **30 request; 60 assertion, 0 fail; exit `0`**.
- `results/newman-fail.txt`: **60 assertion, 1 fail; exit `1`**. Нэг буруу HTTP status oracle илэрсэн.
- `results/newman-down.txt`: сервер зогссон үед `ECONNREFUSED`; exit `1`. Энэ нь API-ийн хариуны oracle буруу гэсэн үг биш, Newman сервертэй холбогдож чадаагүй интерфейс/холболтын алдаа юм.

## 8 өгүүлбэртэй дүгнэлт

1. Teams-ээс ирсэн серверийг нэмэлт сангүйгээр Node.js дээр ажиллуулсан.
2. Тест дизайны таван алхмаар бүртгэлийн функцийг сонгож, гол төлөв ба урьдач нөхцөлийг төлөөлөх утгаар хувиргасан.
3. Collection-д нэг happy path, дөрвөн үндсэн алдааны төрөл, хоёр давхар алдаа, хоёр дутуу талбарын кейс орсон.
4. Кейс бүр оюутан болон хичээлээ PUT-ээр setup хийдэг тул өөр кейсийн өгөгдөлд найдахгүй.
5. Амжилтын oracle `201` болон `result: OK`-г шалгаж, `registrationID` зөвхөн number төрөлтэй эсэхийг шалгасан.
6. PASS run 60 assertion-ийг бүгдийг давж, exit code `0` өгсөн.
7. HTTP status oracle-ыг зориуд буруу болгосон run нэг assertion унагаж, exit code `1` өгсөн.
8. Серверийг унтраасан run `ECONNREFUSED` гаргасан нь холболтын алдаа болохоос бизнес oracle-ийн алдаа биш гэдгийг харуулсан.
