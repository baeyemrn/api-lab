# API lab results

## Part 0
- Step 0.1, the number of pizza names: 18 ชื่อ (ตั้งแต่ Americana ถึง Siciliana)
- Step 0.2, status line กับ headers: ได้ `HTTP/2 200` มี `content-type: application/json` และมี `set-cookie: AWSALB=...` ตามที่ชีทบอก
- Step 0.3, status code of the request without a token: 401 ได้ body เป็น `{"error":"authentication failed"}` พอใส่ header จาก auth.txt แล้วถึงได้รายชื่อ tools มา
- Step 0.4, the name of your pizza: A Mean Romana (ใช้ Knife ตัด ingredient ทุกตัวเป็น vegetarian true หมด)
- Step 0.5, ลองใน Python shell: ได้ 200 เหมือนตอนใช้ curl ชื่อแรกคือ Americana ส่วนพิซซ่าที่สุ่มได้รอบนี้คือ The Giant Frutti di Mare
- Step 0.9, สร้าง user เองใน shell: ได้ user `lab-e7b352816832` (201) ล็อกอินแล้วดู ratings ได้ list ว่าง ใน session มี cookie `AWSALB` กับ `AWSALBCORS`
- Step 0.12, results of the three runs without a session (optional): รอบ 1 fail, รอบ 2 pass, รอบ 3 fail รอบที่ fail สร้าง user ได้ 201 ปกติ แต่ตอน login ได้ 401 (`assert 401 == 200`) เพราะไม่มี cookie AWSALB เลยโดนส่งไปเครื่องที่ไม่รู้จัก user นี้ รอบ 2 น่าจะบังเอิญไปตกเครื่องเดิม เป็นเทสต์ flaky ชัดๆ เลยลบ test_no_session.py ทิ้งไปแล้ว
- Step 0.16, blocks of "how many of the three tools are excluded": แบ่งได้ 4 block ตามจำนวน tool ที่ตัดออก
  - ตัด 0 อัน ใช้ `[]` (เพิ่มเอง) ผ่าน
  - ตัด 1 อัน ใช้ `["Knife"]` กับ `["Scissors"]` (มีให้ในชีทอยู่แล้ว) ผ่าน
  - ตัด 2 อัน ใช้ `["Knife", "Scissors"]` (เพิ่มเอง) ผ่าน
  - ตัด 3 อัน ใช้ `["Knife", "Pizza cutter", "Scissors"]` อันนี้ได้ 500 body ว่าง เลยย้ายไปเป็นเทสต์แยก `test_all_tools_excluded_is_refused` แล้ว mark xfail ไว้

## Defect report
- Title: Defect 1 ตัด tool ออกครบทั้ง 3 อันแล้วได้ 500 กับ body ว่าง
- Request: `POST https://quickpizza.grafana.com/api/pizza` ใส่ header `Authorization: token ...` body เป็น `{"excludedTools": ["Knife", "Pizza cutter", "Scissors"]}` ลองด้วย curl คือ `curl -i -H "@auth.txt" -d "@no-tools.json" https://quickpizza.grafana.com/api/pizza`
- Expected result: ไม่มีพิซซ่าไหนทำได้ถ้าไม่มี tool เลย ตามที่ product owner บอกควรได้ 400 พร้อม error message เป็น JSON ไม่ควรได้ 500
- Actual result: ได้ `HTTP/2 500` และ `content-length: 0` คือไม่มี body เลย ไม่มีบอกว่าผิดอะไร
- Severity: Medium คิดว่าไม่ได้ทำให้ระบบล่มหรือข้อมูลเสีย request อื่นยังใช้ได้ปกติ แต่ลูกค้าที่ไม่มีอุปกรณ์อะไรเลยจะเจอ server error แทนที่จะได้คำอธิบาย ฝั่งแอปก็บอก user ไม่ได้ว่าเกิดอะไรขึ้น

## Part A
- Step A4, status codes when the other user reads, changes and deletes:
  - อ่าน (GET): 404 ได้ `{"error":"not found"}`
  - แก้ (PUT): 400 ได้ `{"error":"rating ID <id> not found"}`
  - ลบ (DELETE): 400 ได้ `{"error":"rating ID <id> not found"}`
  - หลังโดนลองทั้ง 3 แบบ เจ้าของยังอ่าน rating ได้ 200 และ stars ยังเป็น 3 เหมือนเดิม

## Part B
1. failure ที่เทสต์กับ server จริงเจอได้แต่ unit test ที่ใช้ stub เจอไม่ได้ อย่างแรกคือปัญหาตอนหลายส่วนทำงานร่วมกัน เช่นเรื่อง load balancer ใน Step 0.12 ที่ส่ง login ไปเครื่องที่ไม่รู้จัก user หรือ error ของ JSON decoder ฝั่ง Go ตอนส่ง 3.5 ไป ซึ่ง stub ไม่มีทางตอบแบบนี้ อย่างที่สองคือปัญหาที่มาจากข้อมูลหรือ config จริงบน server เช่น 500 ตอนตัด tool ออกครบ 3 อันใน Step 0.16 เพราะ stub จะตอบแค่สิ่งที่เราเขียนให้มันตอบ ส่วนเหตุผลที่ยังควรเขียน unit test ด้วยคือมันเร็ว รันซ้ำได้ผลเหมือนเดิมทุกรอบ ไม่ต้องพึ่งเน็ตหรือข้อมูลของคนอื่น และพอพังก็รู้เลยว่าพังที่ฟังก์ชันไหน แถมยังทดสอบกรณีที่ทำผ่าน API ยากๆ ได้ เช่น database ล่ม
2. เพราะ rating ที่สร้างด้วย public token เป็นของทุกคน ถ้าใช้ token นั้นคนอื่นอาจมาแก้หรือลบข้อมูลตอนที่เทสต์เรากำลังรันอยู่ เทสต์ก็จะพังแบบงงๆ และเราก็อาจไปทำข้อมูลคนอื่นพังด้วย พอแต่ละเทสต์สร้าง user ใหม่ของตัวเอง ข้อมูลก็เป็นของเทสต์นั้นคนเดียว รันเดี่ยวๆ หรือสลับลำดับก็ผ่านเหมือนเดิม สำหรับ Project 2 จะใช้วิธีเดียวกัน คือให้แต่ละเทสต์สร้าง customer, cart, product หรือ booking ของตัวเองโดยตั้งชื่อไม่ซ้ำแบบ `lab-<uuid>` ใช้แค่ข้อมูลที่ตัวเองสร้าง แล้วลบทิ้งใน fixture หลัง `yield` และจะไม่ไปยุ่งกับข้อมูลของกลุ่มอื่นเลย
3. QuickPizza มีหลายเครื่องอยู่หลัง load balancer และแต่ละเครื่องเก็บ user ของตัวเองแยกกัน response แรกจะส่ง cookie AWSALB มาบอกว่าให้ไปเครื่องไหน `requests.Session` จะเก็บ cookie นี้ไว้แล้วส่งกลับไปทุก request ทำให้ทุก request ในรอบนั้นไปเครื่องเดียวกัน user ที่สร้างไว้ก็ login ต่อได้ ตอนทำ Step 0.12 รันไป 3 รอบ ได้ fail, pass, fail รอบที่ fail สร้าง user ได้ 201 แต่ login ได้ 401 ส่วนรอบที่ผ่านน่าจะบังเอิญไปตกเครื่องเดิม ผลออกมาไม่เหมือนกันทั้งที่ไม่ได้แก้โค้ดเลย ก็คือเทสต์ flaky นั่นเอง
4. block ที่เจอ defect คือ "ตัดออกครบ 3 อัน" สาเหตุที่ 500 body ว่างแย่กว่า 400 ที่มี message เพราะแอปที่เรียก API ไม่รู้เลยว่าผิดที่ request ของตัวเองหรือ server พัง เลยบอก user ไม่ได้ว่าเกิดอะไรขึ้น อาจจะ retry ไปเรื่อยๆ ทั้งที่ยังไงก็ไม่สำเร็จ หรือแอปอาจ crash ตอนพยายามอ่าน body ว่างเป็น JSON ส่วน 400 ที่มี message บอกชัดว่าขอแบบนี้ไม่ได้เพราะอะไร ที่เก็บเทสต์ไว้เป็น xfail แทนการลบเพราะอยากให้ defect ยังโผล่ให้เห็นทุกครั้งที่รันและอยู่ใน report แต่ไม่ทำให้ทั้ง suite แดง แล้วพอใส่ `strict=True` ไว้ ถ้าวันไหน QuickPizza แก้บั๊กแล้ว รันจะ fail ทันทีเพื่อเตือนให้เอา mark ออก จากนั้นเทสต์นี้ก็จะคอยกันไม่ให้บั๊กกลับมาอีก
5. model มี 4 characteristic จำนวน block คือ 3, 4, 2, 3 ถ้าเป็น base choice ต้องมี base test 1 ตัว บวกกับ block ที่ไม่ใช่ base ของแต่ละตัว ได้ 1 + (3-1) + (4-1) + (2-1) + (3-1) = 1 + 2 + 3 + 1 + 2 = 9 เทสต์ ถ้าเป็น all combinations ต้องคูณกันหมด ได้ 3 x 4 x 2 x 3 = 72 เทสต์
6. ได้ 404 ตอนอ่าน และได้ 400 ตอนแก้กับตอนลบ ที่คาดไว้คือควรได้ code เดียวกันทั้ง 3 แบบ อาจเป็น 403 Forbidden (มีอยู่จริงแต่ไม่ใช่ของคุณ) หรือ 404 Not Found ซึ่งดีกว่าเพราะไม่บอกคนอื่นว่ามี rating id นี้อยู่ ส่วน 400 Bad Request ไม่ค่อยถูก เพราะตัว request ไม่ได้ผิด format อะไร แค่คนส่งไม่ใช่เจ้าของ ถามว่าเป็น defect มั้ย คิดว่าเป็น defect เล็กๆ เรื่อง status code ไม่ตรงกันระหว่าง GET กับ PUT/DELETE แต่ไม่ใช่ปัญหาด้าน security เพราะระบบก็ยังปฏิเสธทุกครั้งและ rating ไม่ได้เปลี่ยน แค่ client ที่เขียนรับ 403/404 ไว้จะเข้าใจผิดว่า request ตัวเองผิด
7. ได้ `HTTP/2 200` กลับมาพร้อม list ของ ratings เลย เช่น `{"ratings":[{"id":1,"stars":5,"pizza_id":1},{"id":2,"stars":1,"pizza_id":2}, ...]}` ทั้งที่ token `0000000000000000` เราแต่งขึ้นเองไม่ได้ได้มาจากการ login ดูเหมือนระบบจะรับ token อะไรก็ได้ที่ยาว 16 ตัว อันนี้คิดว่าเป็น defect แน่นอน คือข้ามการยืนยันตัวตนได้ ถ้าเป็นระบบจริงคนแปลกหน้าก็แค่แต่ง token ขึ้นมาเองแล้วเข้าไปดูข้อมูลคนอื่นได้ และถ้าใช้ token เดียวกันก็สร้าง แก้ หรือลบข้อมูลได้ด้วยโดยไม่ต้อง login เลย (ตอนลองใช้แค่อ่านอย่างเดียว ไม่ได้สร้างหรือแก้อะไร)
8. สิ่งที่ API test เช็กได้ดีกว่าคือ status code, error message และ field ใน response แบบละเอียด รวมถึง input ที่หน้าเว็บไม่ยอมให้พิมพ์ เช่น stars เป็นทศนิยม ไม่ส่ง field บาง field หรือ pizza id ที่ไม่มีอยู่จริง แล้วก็รันเร็วกว่าเพราะไม่ต้องเปิด browser ส่วนสิ่งที่ต้องเทสต์ผ่านหน้าเว็บเท่านั้นคือสิ่งที่ user เห็นและกดจริงๆ เช่น หน้าเว็บแสดงผลถูกมั้ย ปุ่ม ฟอร์ม ลิงก์ใช้ได้มั้ย และ error message ขึ้นบนจอให้ user เห็นจริงหรือเปล่า
