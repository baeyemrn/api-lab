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

## เทสต์ที่ครอบแต่ละ block (base choice coverage)

เทสต์อยู่ใน `test_create_rating.py` ทั้งหมด แต่ละเทสต์จะเปลี่ยนแค่ characteristic เดียวออกจาก base ส่วนที่เหลือใช้ค่า base เหมือนเดิม

| Characteristic | Block | Test |
|---|---|---|
| C1 | below 1 | `test_stars_out_of_range[0]` |
| C1 | 1 to 5 (base) | `test_base_choice_is_saved` |
| C1 | above 5 | `test_stars_out_of_range[6]` |
| C2 | whole number (base) | `test_base_choice_is_saved` |
| C2 | decimal | `test_stars_decimal` |
| C2 | text | `test_stars_text` |
| C2 | missing | `test_stars_missing` |
| C3 | exists (base) | `test_base_choice_is_saved` |
| C3 | does not exist | `test_pizza_does_not_exist` |
| C4 | the user's own (base) | `test_base_choice_is_saved` |
| C4 | none | `test_no_token` |
| C4 | wrong length | `test_token_wrong_length` |

นับได้ 1 + 2 + 3 + 1 + 2 = 9 เทสต์ (base 1 ตัว บวก block ที่ไม่ใช่ base ของแต่ละ characteristic)

ข้อสังเกต ตอนไม่ส่ง field stars เลย (missing) ระบบตอบ error เดียวกับตอน stars เกินช่วง น่าจะเพราะค่าที่ไม่ส่งมากลายเป็น 0
