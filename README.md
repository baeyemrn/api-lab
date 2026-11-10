# api-lab

แลป API Testing ด้วย pytest กับ requests วิชา ITDS362 เทสต์ API ของ QuickPizza (https://quickpizza.grafana.com)

## Input domain model ของ POST /api/ratings

POST /api/ratings รับ body 2 field คือ `pizza_id` กับ `stars` และต้องมี token ใน header ระบบรับ stars เป็นจำนวนเต็ม 1 ถึง 5 ต้องเป็นพิซซ่าที่มีอยู่จริง และต้องเป็น token ของ user ที่ login แล้ว

| Characteristic | Blocks | Values in the tests |
|---|---|---|
| C1 Value of `stars` | below 1, 1 to 5 (base), above 5 | 0, 3, 6 |
| C2 Type of `stars` | whole number (base), decimal, text, missing | 3, 3.5, "3", no field |
| C3 Pizza | exists (base), does not exist | a new pizza, 999999999 |
| C4 Token | the user's own (base), none, wrong length | from `auth`, no header, `token short` |
