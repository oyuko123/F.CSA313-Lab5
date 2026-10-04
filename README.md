# F.CSA313 Lab 5 — API системийн тест: Postman ба Newman

## 1. Оюутны мэдээлэл

- Оюутан: Т.Оюунжаргал
- Оюутны код: B232270149

---

## 2. Лабораторийн зорилго

Энэхүү лабораторийн ажлын зорилго нь REST API систем дээр системийн түвшний тест зохиож, Postman ашиглан тестүүдийг хэрэгжүүлэх, Newman ашиглан командын мөрөөс автоматжуулан ажиллуулахад оршино.

Тестийн загварчлалд дараах 5 алхмыг ашигласан:

1. Бие даан тестлэх боломжтой функцийг тодорхойлох
2. Боломжит сонголтуудыг тодорхойлох
3. Сонголт бүрийн төлөөлөх утгыг сонгох
4. Тестийн specification тодорхойлох
5. Specification бүрийг бодит test case болгон хэрэгжүүлэх

---

## 3. Ашигласан хэрэгслүүд

- Node.js
- Postman
- Newman
- Git
- GitHub
- WSL2 Ubuntu

---

### Орчны хувилбар

```text
$ node -v
v22.22.1

$ newman -v
6.2.1
```

## 4. API систем

Лабораторид өгөгдсөн `server.js` дээр ажилладаг локал REST API ашигласан.

API хаяг:

```text
http://localhost:3000
```
### Үндсэн endpoint-ууд

| Method | Endpoint | Зориулалт |
|---|---|---|
| PUT | `/students/:id` | Оюутны мэдээлэл үүсгэх/шинэчлэх |
| PUT | `/courses/:id` | Хичээлийн prerequisite тохируулах |
| GET | `/courses/:id` | Хичээлийн мэдээлэл авах |
| POST | `/registrations` | Хичээлд бүртгүүлэх |
| GET | `/registrations` | Бүх бүртгэлийг авах |

## 5. Тестлэх үндсэн функц

Энэ лабораторийн үндсэн тестийн зорилго нь POST /registrations endpoint-оор оюутныг хичээлд бүртгэх үйлдлийг шалгах юм.

Request body:

{
  "studentID": "B231234567",
  "courseID": "CS313"
}

Амжилттай үед:

{
  "result": "OK",
  "registrationID": 1
}

алдааны үед result талбарт тухайн алдааны төрлийг буцаана.

## 6. Choice ба Representative Value

Тестийн сонголтууд болон тэдгээрийг төлөөлөх утгуудыг дараах байдлаар тодорхойлсон.

| Choice | Боломжит сонголт | Representative value |
|---|---|---|
| studentID validity | Active student | `B231234567` |
| studentID validity | Inactive student | `B239999998` |
| studentID validity | Student does not exist | `B239999999` |
| studentID validity | Missing | `studentID` талбарыг оруулахгүй |
| coursesTaken | Prerequisite met | `["CS201"]` |
| coursesTaken | Prerequisite not met | `[]` |
| courseID validity | Existing course | `CS313` |
| courseID validity | Missing/non-existing course | `CS999` |
| Course prerequisites | All prerequisites met | `["CS201"]` + student has `CS201` |
| Course prerequisites | Some prerequisites missing | `["CS201"]` + student has `[]` |
| Course prerequisites | No prerequisites | `[]` |
| Required fields | All fields present | `studentID`, `courseID` |
| Required fields | Required field missing | `courseID` only |

## 7. Test Specifications

| # | Specification | Setup / Input | Expected Status | Expected Result |
|---|---|---|---:|---|
| 1 | Happy Path | Active student, prerequisite met, existing course | 201 | `OK` |
| 2 | Student Not Found | Non-existing student | 200 | `ERROR_NO_STUDENT` |
| 3 | Inactive Student | Inactive student | 200 | `ERROR_INACTIVE_STUDENT` |
| 4 | Course Not Found | Non-existing course | 200 | `ERROR_NO_COURSE` |
| 5 | Prerequisites Missing | Required prerequisite not taken | 200 | `ERROR_PREREQUISITES` |
| 6 | Missing Student ID | `studentID` omitted | 400 | `ERROR_BAD_REQUEST` |
| 7 | Double Error | Inactive student + missing course | 200 | `ERROR_INACTIVE_STUDENT` |
| 8 | No Prerequisites | Existing course with no prerequisites | 201 | `OK` |

## 8. Double-error туршилт

Assignment-д double-error нөхцөлүүдийн precedence урьдчилан тодорхойлогдоогүй тул туршилтаар шалгасан.

Туршилтын нөхцөл:

-Student: B239999995
-Student status: inactive
-Course: CS999
-CS999 course байхгүй

Request:

{
  "studentID": "B239999995",
  "courseID": "CS999"
}

Бодит үр дүн:

HTTP 200
ERROR_INACTIVE_STUDENT

Иймээс inactive student-ийн алдаа нь missing course-ийн алдаанаас өмнө шалгагдаж байна гэж тогтоосон.

## 9. Postman тестийн бүтэц

Postman collection:

Lab05 - API System Test
│
├── 01 - Happy Path
│   ├── Setup - Active Student
│   ├── Setup - Course CS313
│   └── Register - Happy Path
│
├── 02 - Student Not Found
│   ├── Setup - Student Not Found Course
│   └── Test - Student Not Found
│
├── 03 - Inactive Student
│   ├── Setup - Inactive Student
│   ├── Setup - Inactive Student Course
│   └── Test - Inactive Student
│
├── 04 - Course Not Found
│   ├── Setup - Course Not Found Student
│   └── Test - Course Not Found
│
├── 05 - Prerequisites Missing
│   ├── Setup - Prerequisite Student
│   ├── Setup - Course CS314
│   └── Test - Prerequisites Missing
│
├── 06 - Missing Student ID
│   └── Test - Missing Student ID
│
├── 07 - Double Error
│   ├── Setup - Double Error Student
│   └── Test - Inactive Student and Missing Course
│
├── 08 - No Prerequisites
│   ├── Setup - No Prerequisites Student
│   ├── Setup - Course CS315
│   └── Test - No Prerequisites
│
└── 09 - Public API - JSONPlaceholder
    └── GET - Users

Collection variable:

baseUrl = http://localhost:3000

Тест бүр өөрийн шаардлагатай setup request-үүдтэй бөгөөд setup-ийн дараа тухайн specification-ийн registration request ажиллана.

## 10. Postman Assertions

Үндсэн test case-үүдэд HTTP status code болон response-ийн result утгыг шалгасан.

### Happy Path

pm.test("Status code is 201", function () {
    pm.response.to.have.status(201);
});

pm.test("Result is OK", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.result).to.eql("OK");
});

### Error case-ийн жишээ

pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Result is ERROR_NO_STUDENT", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData.result).to.eql("ERROR_NO_STUDENT");
});

## 11. Newman PASS тест

### Үндсэн collection:

lab05-collection.json

### Ажиллуулсан команд:

newman run lab05-collection.json 2>&1 | tee results/newman-pass.txt

### PASS үр дүн:

20 requests
29 test-scripts
20 prerequest-scripts
21 assertions

### Newman exit code:

exit=0

## 12. Newman FAIL тест

Тестийн oracle-уудын нэгийг зориудаар буруу болгож failure test хийсэн.

Зориудаар:

pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

гэж өөрчилсөн боловч API бодитоор 201 Created буцаасан.

### FAIL collection:

lab05-collection-fail.json

### Ажиллуулсан команд:

newman run lab05-collection-fail.json 2>&1 | tee results/newman-fail.txt

### Үр дүн:

20 requests
29 test-scripts
20 prerequest-scripts
21 assertions
1 failed
exit=1

### Алдаа:

Status code is 201
expected response status 200 but got 201

### Newman exit code:

exit=1

Энэ нь буруу oracle-г Newman зөв илрүүлснийг харуулж байна.

## 13. Server-down / Connectivity тест

Local server-ийг зогсоосны дараа main collection-ийг Newman ашиглан
дахин ажиллуулсан.

### Ажиллуулсан команд:

newman run lab05-collection.json 2>&1 | tee results/newman-down.txt

### Үр дүн:

- Requests: 20
- Failed requests: 19
- Assertions: 21
- Failed assertions: 18
- Exit code: 1
- Error: `ECONNREFUSED 127.0.0.1:3000`

JSONPlaceholder-ийн public API request амжилттай ажилласан тул 20
request-ийн 19 нь local server холболтын алдаагаар failed болсон.

`ECONNREFUSED` нь test oracle-ийн буруу байдал биш, харин local API
server ажиллахгүй байгаатай холбоотой connectivity/interface failure
юм. Иймээс энэ үр дүнг функциональ API-ийн defect гэж үзэхгүй.

## 14. Нэмэлт даалгавар — JSONPlaceholder Public API

Нэмэлт даалгаврын хүрээнд `https://jsonplaceholder.typicode.com/users`
public API-д GET хүсэлт илгээж тест хийсэн. Энэ хүсэлт нь local API-ийн
тестүүдээс ялгаатай нь setup шаарддаггүй бөгөөд зөвхөн response-ийн
утгуудыг шалгасан.

### Тестийн хүсэлт

**Method:** GET

**URL:** `https://jsonplaceholder.typicode.com/users`

### Ашигласан oracle-ууд

1. HTTP status code нь `200` байх.
2. Response нь array байх.
3. Response-ийн эхний хэрэглэгчийн `name` нь `Leanne Graham` байх.

Postman дээрх тестийн үр дүнд **3/3 assertion амжилттай** болсон.

JSONPlaceholder нь GET-only API бөгөөд setup шаарддаггүй тул local API-ийн
registration тестүүдээс хэрэгжүүлэхэд хялбар байсан. Харин public API
учраас гадаад сервисийн хүртээмжээс хамаардаг нь ялгаатай.

## 15. Newman-ийн бүтэн үр дүн

### 15.1 PASS — `results/newman-pass.txt`

newman

Lab05 - API System Test

❏ 01 - Happy Path
↳ Setup - Active Student
  PUT http://localhost:3000/students/B231234567 [200 OK, 178B, 32ms]

↳ Setup - Course CS313
  PUT http://localhost:3000/courses/CS313 [200 OK, 178B, 4ms]

↳ Register - Happy Path
  POST http://localhost:3000/registrations [201 Created, 202B, 2ms]
  ✓  Status code is 201
  ✓  Result is OK
  ✓  Registration ID exists and is numeric

❏ 02 - Student Not Found
↳ Setup - Student Not Found Course
  PUT http://localhost:3000/courses/CS313 [200 OK, 178B, 5ms]

↳ Test - Student Not Found
  POST http://localhost:3000/registrations [200 OK, 192B, 3ms]
  ✓  Status code is 200
  ✓  Result is ERROR_NO_STUDENT

❏ 03 - Inactive Student
↳ Setup - Inactive Student
  PUT http://localhost:3000/students/B239999998 [200 OK, 178B, 5ms]

↳ Setup - Inactive Student Course
  PUT http://localhost:3000/courses/CS313 [200 OK, 178B, 4ms]

↳ Test - Inactive Student
  POST http://localhost:3000/registrations [200 OK, 198B, 4ms]
  ✓  Status code is 200
  ✓  Result is ERROR_INACTIVE_STUDENT

❏ 04 - Course Not Found
↳ Setup - Course Not Found Student
  PUT http://localhost:3000/students/B239999997 [200 OK, 178B, 5ms]

↳ Test - Course Not Found
  POST http://localhost:3000/registrations [200 OK, 191B, 3ms]
  ✓  Status code is 200
  ✓  Result is ERROR_NO_COURSE

❏ 05 - Prerequisites Missing
↳ Setup - Prerequisite Student
  PUT http://localhost:3000/students/B239999996 [200 OK, 178B, 3ms]

↳ Setup - Course CS314
  PUT http://localhost:3000/courses/CS314 [200 OK, 178B, 2ms]

↳ Test - Prerequisites Missing
  POST http://localhost:3000/registrations [200 OK, 215B, 2ms]
  ✓  Status code is 200
  ✓  Result is ERROR_PREREQUISITES

❏ 06 - Missing Student ID
↳ Test - Missing Student ID
  POST http://localhost:3000/registrations [400 Bad Request, 202B, 2ms]
  ✓  Status code is 400
  ✓  Result is ERROR_BAD_REQUEST

❏ 07 - Double Error
↳ Setup - Double Error Student
  PUT http://localhost:3000/students/B239999995 [200 OK, 178B, 2ms]

↳ Test - Inactive Student and Missing Course
  POST http://localhost:3000/registrations [200 OK, 198B, 2ms]
  ✓  Status code is 200
  ✓  Inactive student takes precedence

❏ 08 - No Prerequisites
↳ Setup - No Prerequisites Student
  PUT http://localhost:3000/students/B239999994 [200 OK, 178B, 2ms]

↳ Setup - Course CS315
  PUT http://localhost:3000/courses/CS315 [200 OK, 178B, 2ms]

↳ Test - No Prerequisites
  POST http://localhost:3000/registrations [201 Created, 202B, 2ms]
  ✓  Status code is 201
  ✓  Result is OK
  ✓  Registration ID exists and is numeric

❏ 09 - Public API - JSONPlaceholder
↳ GET - Users
  GET https://jsonplaceholder.typicode.com/users [200 OK, 6.6kB, 93ms]
  ✓  Status code is 200
  ✓  Response is an array
  ✓  First user's name is correct

┌─────────────────────────┬──────────────────┬─────────────────┐
│                         │         executed │          failed │
├─────────────────────────┼──────────────────┼─────────────────┤
│              iterations │                1 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│                requests │               20 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│            test-scripts │               29 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│      prerequest-scripts │               20 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│              assertions │               21 │               0 │
├─────────────────────────┴──────────────────┴─────────────────┤
│ total run duration: 682ms                                    │
├──────────────────────────────────────────────────────────────┤
│ total data received: 6.09kB (approx)                         │
├──────────────────────────────────────────────────────────────┤
│ average response time: 8ms [min: 2ms, max: 93ms, s.d.: 20ms] │
└──────────────────────────────────────────────────────────────┘

### 15.2 FAIL — `results/newman-fail.txt`

newman

Lab05 - API System Test

❏ 01 - Happy Path
↳ Setup - Active Student
  PUT http://localhost:3000/students/B231234567 [200 OK, 178B, 52ms]

↳ Setup - Course CS313
  PUT http://localhost:3000/courses/CS313 [200 OK, 178B, 5ms]

↳ Register - Happy Path
  POST http://localhost:3000/registrations [201 Created, 202B, 3ms]
  1. Status code is 201
  ✓  Result is OK
  ✓  Registration ID exists and is numeric

❏ 02 - Student Not Found
↳ Setup - Student Not Found Course
  PUT http://localhost:3000/courses/CS313 [200 OK, 178B, 3ms]

↳ Test - Student Not Found
  POST http://localhost:3000/registrations [200 OK, 192B, 3ms]
  ✓  Status code is 200
  ✓  Result is ERROR_NO_STUDENT

❏ 03 - Inactive Student
↳ Setup - Inactive Student
  PUT http://localhost:3000/students/B239999998 [200 OK, 178B, 3ms]

↳ Setup - Inactive Student Course
  PUT http://localhost:3000/courses/CS313 [200 OK, 178B, 4ms]

↳ Test - Inactive Student
  POST http://localhost:3000/registrations [200 OK, 198B, 3ms]
  ✓  Status code is 200
  ✓  Result is ERROR_INACTIVE_STUDENT

❏ 04 - Course Not Found
↳ Setup - Course Not Found Student
  PUT http://localhost:3000/students/B239999997 [200 OK, 178B, 5ms]

↳ Test - Course Not Found
  POST http://localhost:3000/registrations [200 OK, 191B, 3ms]
  ✓  Status code is 200
  ✓  Result is ERROR_NO_COURSE

❏ 05 - Prerequisites Missing
↳ Setup - Prerequisite Student
  PUT http://localhost:3000/students/B239999996 [200 OK, 178B, 2ms]

↳ Setup - Course CS314
  PUT http://localhost:3000/courses/CS314 [200 OK, 178B, 2ms]

↳ Test - Prerequisites Missing
  POST http://localhost:3000/registrations [200 OK, 215B, 2ms]
  ✓  Status code is 200
  ✓  Result is ERROR_PREREQUISITES

❏ 06 - Missing Student ID
↳ Test - Missing Student ID
  POST http://localhost:3000/registrations [400 Bad Request, 202B, 3ms]
  ✓  Status code is 400
  ✓  Result is ERROR_BAD_REQUEST

❏ 07 - Double Error
↳ Setup - Double Error Student
  PUT http://localhost:3000/students/B239999995 [200 OK, 178B, 2ms]

↳ Test - Inactive Student and Missing Course
  POST http://localhost:3000/registrations [200 OK, 198B, 3ms]
  ✓  Status code is 200
  ✓  Inactive student takes precedence

❏ 08 - No Prerequisites
↳ Setup - No Prerequisites Student
  PUT http://localhost:3000/students/B239999994 [200 OK, 178B, 3ms]

↳ Setup - Course CS315
  PUT http://localhost:3000/courses/CS315 [200 OK, 178B, 3ms]

↳ Test - No Prerequisites
  POST http://localhost:3000/registrations [201 Created, 202B, 2ms]
  ✓  Status code is 201
  ✓  Result is OK
  ✓  Registration ID exists and is numeric

❏ 09 - Public API - JSONPlaceholder
↳ GET - Users
  GET https://jsonplaceholder.typicode.com/users [200 OK, 6.6kB, 84ms]
  ✓  Status code is 200
  ✓  Response is an array
  ✓  First user's name is correct

┌─────────────────────────┬──────────────────┬─────────────────┐
│                         │         executed │          failed │
├─────────────────────────┼──────────────────┼─────────────────┤
│              iterations │                1 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│                requests │               20 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│            test-scripts │               29 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│      prerequest-scripts │               20 │               0 │
├─────────────────────────┼──────────────────┼─────────────────┤
│              assertions │               21 │               1 │
├─────────────────────────┴──────────────────┴─────────────────┤
│ total run duration: 744ms                                    │
├──────────────────────────────────────────────────────────────┤
│ total data received: 6.09kB (approx)                         │
├──────────────────────────────────────────────────────────────┤
│ average response time: 9ms [min: 2ms, max: 84ms, s.d.: 20ms] │
└──────────────────────────────────────────────────────────────┘

[31m  # [39m[31m failure        [39m[31m detail                                                [39m
[90m    [39m[90m                [39m[90m                                                       [39m
 1.  AssertionError  Status code is 201                                    
                     expected response to have status code 200 but got 201 
                     at assertion:0 in test-script                         
                     inside "01 - Happy Path / Register - Happy Path"      

### 15.3 Server DOWN — `results/newman-down.txt`

newman

Lab05 - API System Test

❏ 01 - Happy Path
↳ Setup - Active Student
  PUT http://localhost:3000/students/B231234567 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Setup - Course CS313
  PUT http://localhost:3000/courses/CS313 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Register - Happy Path
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
  4. Status code is 201
  5. Result is OK
  6. Registration ID exists and is numeric

❏ 02 - Student Not Found
↳ Setup - Student Not Found Course
  PUT http://localhost:3000/courses/CS313 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Test - Student Not Found
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
  9. Status code is 200
 10. Result is ERROR_NO_STUDENT

❏ 03 - Inactive Student
↳ Setup - Inactive Student
  PUT http://localhost:3000/students/B239999998 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Setup - Inactive Student Course
  PUT http://localhost:3000/courses/CS313 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Test - Inactive Student
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
 14. Status code is 200
 15. Result is ERROR_INACTIVE_STUDENT

❏ 04 - Course Not Found
↳ Setup - Course Not Found Student
  PUT http://localhost:3000/students/B239999997 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Test - Course Not Found
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
 18. Status code is 200
 19. Result is ERROR_NO_COURSE

❏ 05 - Prerequisites Missing
↳ Setup - Prerequisite Student
  PUT http://localhost:3000/students/B239999996 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Setup - Course CS314
  PUT http://localhost:3000/courses/CS314 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Test - Prerequisites Missing
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
 23. Status code is 200
 24. Result is ERROR_PREREQUISITES

❏ 06 - Missing Student ID
↳ Test - Missing Student ID
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
 26. Status code is 400
 27. Result is ERROR_BAD_REQUEST

❏ 07 - Double Error
↳ Setup - Double Error Student
  PUT http://localhost:3000/students/B239999995 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Test - Inactive Student and Missing Course
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
 30. Status code is 200
 31. Inactive student takes precedence

❏ 08 - No Prerequisites
↳ Setup - No Prerequisites Student
  PUT http://localhost:3000/students/B239999994 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Setup - Course CS315
  PUT http://localhost:3000/courses/CS315 [errored]
     connect ECONNREFUSED 127.0.0.1:3000

↳ Test - No Prerequisites
  POST http://localhost:3000/registrations [errored]
     connect ECONNREFUSED 127.0.0.1:3000
 35. Status code is 201
 36. Result is OK
 37. Registration ID exists and is numeric

❏ 09 - Public API - JSONPlaceholder
↳ GET - Users
  GET https://jsonplaceholder.typicode.com/users [200 OK, 6.6kB, 137ms]
  ✓  Status code is 200
  ✓  Response is an array
  ✓  First user's name is correct

┌─────────────────────────┬───────────────────┬───────────────────┐
│                         │          executed │            failed │
├─────────────────────────┼───────────────────┼───────────────────┤
│              iterations │                 1 │                 0 │
├─────────────────────────┼───────────────────┼───────────────────┤
│                requests │                20 │                19 │
├─────────────────────────┼───────────────────┼───────────────────┤
│            test-scripts │                29 │                 0 │
├─────────────────────────┼───────────────────┼───────────────────┤
│      prerequest-scripts │                20 │                 0 │
├─────────────────────────┼───────────────────┼───────────────────┤
│              assertions │                21 │                18 │
├─────────────────────────┴───────────────────┴───────────────────┤
│ total run duration: 1069ms                                      │
├─────────────────────────────────────────────────────────────────┤
│ total data received: 5.65kB (approx)                            │
├─────────────────────────────────────────────────────────────────┤
│ average response time: 6ms [min: 137ms, max: 137ms, s.d.: 29ms] │
└─────────────────────────────────────────────────────────────────┘

[31m   # [39m[31m failure        [39m[31m detail                                                                  [39m
[90m     [39m[90m                [39m[90m                                                                         [39m
 01.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 02.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 03.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 04.  AssertionError  Status code is 201                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "01 - Happy Path / Register - Happy Path"                        
[90m     [39m[90m                [39m[90m                                                                         [39m
 05.  JSONError       Result is OK                                                            
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "01 - Happy Path / Register - Happy Path"                        
[90m     [39m[90m                [39m[90m                                                                         [39m
 06.  JSONError       Registration ID exists and is numeric                                   
                      "undefined" is not valid JSON                                           
                      at assertion:2 in test-script                                           
                      inside "01 - Happy Path / Register - Happy Path"                        
[90m     [39m[90m                [39m[90m                                                                         [39m
 07.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 08.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 09.  AssertionError  Status code is 200                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "02 - Student Not Found / Test - Student Not Found"              
[90m     [39m[90m                [39m[90m                                                                         [39m
 10.  JSONError       Result is ERROR_NO_STUDENT                                              
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "02 - Student Not Found / Test - Student Not Found"              
[90m     [39m[90m                [39m[90m                                                                         [39m
 11.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 12.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 13.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 14.  AssertionError  Status code is 200                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "03 - Inactive Student / Test - Inactive Student"                
[90m     [39m[90m                [39m[90m                                                                         [39m
 15.  JSONError       Result is ERROR_INACTIVE_STUDENT                                        
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "03 - Inactive Student / Test - Inactive Student"                
[90m     [39m[90m                [39m[90m                                                                         [39m
 16.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 17.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 18.  AssertionError  Status code is 200                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "04 - Course Not Found / Test - Course Not Found"                
[90m     [39m[90m                [39m[90m                                                                         [39m
 19.  JSONError       Result is ERROR_NO_COURSE                                               
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "04 - Course Not Found / Test - Course Not Found"                
[90m     [39m[90m                [39m[90m                                                                         [39m
 20.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 21.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 22.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 23.  AssertionError  Status code is 200                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "05 - Prerequisites Missing / Test - Prerequisites Missing"      
[90m     [39m[90m                [39m[90m                                                                         [39m
 24.  JSONError       Result is ERROR_PREREQUISITES                                           
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "05 - Prerequisites Missing / Test - Prerequisites Missing"      
[90m     [39m[90m                [39m[90m                                                                         [39m
 25.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 26.  AssertionError  Status code is 400                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "06 - Missing Student ID / Test - Missing Student ID"            
[90m     [39m[90m                [39m[90m                                                                         [39m
 27.  JSONError       Result is ERROR_BAD_REQUEST                                             
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "06 - Missing Student ID / Test - Missing Student ID"            
[90m     [39m[90m                [39m[90m                                                                         [39m
 28.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 29.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 30.  AssertionError  Status code is 200                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "07 - Double Error / Test - Inactive Student and Missing Course" 
[90m     [39m[90m                [39m[90m                                                                         [39m
 31.  JSONError       Inactive student takes precedence                                       
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "07 - Double Error / Test - Inactive Student and Missing Course" 
[90m     [39m[90m                [39m[90m                                                                         [39m
 32.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 33.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 34.  Error                                                                                   
                      connect ECONNREFUSED 127.0.0.1:3000                                     
                      at request                                                              
                      inside ""                                                               
[90m     [39m[90m                [39m[90m                                                                         [39m
 35.  AssertionError  Status code is 201                                                      
                      expected PostmanResponse{ …(5) } to have property 'code'                
                      at assertion:0 in test-script                                           
                      inside "08 - No Prerequisites / Test - No Prerequisites"                
[90m     [39m[90m                [39m[90m                                                                         [39m
 36.  JSONError       Result is OK                                                            
                      "undefined" is not valid JSON                                           
                      at assertion:1 in test-script                                           
                      inside "08 - No Prerequisites / Test - No Prerequisites"                
[90m     [39m[90m                [39m[90m                                                                         [39m
 37.  JSONError       Registration ID exists and is numeric                                   
                      "undefined" is not valid JSON                                           
                      at assertion:2 in test-script                                           
                      inside "08 - No Prerequisites / Test - No Prerequisites"                

## 16. Дүгнэлт

Энэ лабораторийн 5 алхмаас хамгийн их бодол шаардсан нь 4-р алхам буюу test specification тодорхойлох байсан. Учир нь representative value-уудыг сонгохоос гадна тухайн нөхцөл бүрд ямар HTTP status болон result хүлээхийг тодорхойлох шаардлагатай байсан. Мөн тест бүрийг бие даасан байлгахын тулд шаардлагатай setup үйлдлүүдийг тухайн test case-д нь тусгасан. Double-error нөхцөлд inactive student болон missing course зэрэг хоёр алдаа нэг хүсэлтэд зэрэг тохиолдсон нь давхар нөхцөлийн жишээ болсон. Assignment-д энэ нөхцөлийн precedence тодорхой заагаагүй тул бодит API дээр шалгаж, `ERROR_INACTIVE_STUDENT` түрүүлж буцаж байгааг тогтоосон. Функциональ тестүүдээр өгөгдсөн API-ийн үндсэн шаардлагад бодит согог илрээгүй. Харин зориудаар буруу oracle ашиглахад Newman assertion failure болон exit code 1-ээр алдааг зөв илрүүлсэн. Server-ийг зогсоосны дараах `ECONNREFUSED` алдааг functional defect биш, connectivity/interface failure гэдгийг ялгаж тодорхойлсон. JSONPlaceholder-ийн GET-only тест setup шаарддаггүй тул local API-ийн тестээс хялбар байсан боловч гадаад сервисийн хүртээмжээс хамаардаг болох нь ялгаатай байв.
