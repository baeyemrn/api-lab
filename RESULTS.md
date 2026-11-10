# API lab results

## Part 0
- Step 0.1, the number of pizza names: 18 ชื่อ (ตั้งแต่ Americana ถึง Siciliana)
- Step 0.2, status line กับ headers: ได้ `HTTP/2 200` มี `content-type: application/json` และมี `set-cookie: AWSALB=...` ตามที่ชีทบอก
- Step 0.3, status code of the request without a token: 401 ได้ body เป็น `{"error":"authentication failed"}` พอใส่ header จาก auth.txt แล้วถึงได้รายชื่อ tools มา
- Step 0.4, the name of your pizza: A Mean Romana (ใช้ Knife ตัด ingredient ทุกตัวเป็น vegetarian true หมด)
- Step 0.5, ลองใน Python shell: ได้ 200 เหมือนตอนใช้ curl ชื่อแรกคือ Americana ส่วนพิซซ่าที่สุ่มได้รอบนี้คือ The Giant Frutti di Mare
- Step 0.9, สร้าง user เองใน shell: ได้ user `lab-e7b352816832` (201) ล็อกอินแล้วดู ratings ได้ list ว่าง ใน session มี cookie `AWSALB` กับ `AWSALBCORS`
- Step 0.12, results of the three runs without a session (optional): รอบ 1 fail, รอบ 2 pass, รอบ 3 fail รอบที่ fail สร้าง user ได้ 201 ปกติ แต่ตอน login ได้ 401 (`assert 401 == 200`) เพราะไม่มี cookie AWSALB เลยโดนส่งไปเครื่องที่ไม่รู้จัก user นี้ รอบ 2 น่าจะบังเอิญไปตกเครื่องเดิม เป็นเทสต์ flaky ชัดๆ เลยลบ test_no_session.py ทิ้งไปแล้ว
- Step 0.16, blocks of "how many of the three tools are excluded":

## Defect report
- Title:
- Request:
- Expected result:
- Actual result:
- Severity:

## Part A
- Step A4, status codes when the other user reads, changes and deletes:

## Part B
1.
2.
3.
4.
5.
6.
7.
8.
