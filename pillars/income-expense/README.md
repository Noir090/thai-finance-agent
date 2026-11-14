# Pillar 1 รายรับรายจ่าย + Pillar 5 แจ้งเตือนบิลและ Subscription

ส่วนของ muk ใน Thai Finance Agent พร้อมแดชบอร์ด The Money Journal

- **System-1** (`agent/`): Python ล้วน รันในเครื่อง ไม่ต่อเน็ต จัดหมวดแต่ละรายการพร้อม confidence มั่นใจ ≥ 90% บันทึกเอง 60–89% ส่งให้ Codex คิดต่อ ต่ำกว่า 60% ถามผู้ใช้ จำคำตอบไว้ใน `rules.csv`
- **Skills** (`.agents/skills/`): `income-expense` อ่านสลิปและ statement, `subscription` หารายจ่ายประจำและเตือนก่อนครบกำหนด, `dashboard` สร้าง `my-dashboard.html`
- **แดชบอร์ด**: `dashboard-journal.html` (ปฏิทินวันจ่าย กราฟหมวดรายจ่าย คิวรอตรวจ) ดับเบิลคลิกเปิดได้เลย มีข้อมูลตัวอย่างฝังอยู่ หน้า Tax และ Investment จะแสดงข้อมูลเมื่อ pillar ของเพื่อนอยู่ในโฟลเดอร์เดียวกัน
- **ข้อมูลกลางให้ pillar อื่นอ่าน**: `agent_state.json` (schema `pillar1-state/1`): รายรับรายจ่ายรายเดือน, รายจ่ายประจำ, การแจ้งเตือน, ภาระต่อเดือน, คิวรอตรวจ

## ลองใช้

เปิดโฟลเดอร์นี้ใน Codex หรือ Claude Code แล้วพิมพ์ "สวัสดี" หรือแนบสลิปได้เลย

ลองรอบการทำงานด้วยข้อมูลสมมติ:

```sh
mkdir -p /tmp/try && cp demo/database.example.csv /tmp/try/database.csv
python3 agent/cli.py --dir /tmp/try --today 2026-11-12 ingest demo/statement-2026-11.example.csv
python3 agent/cli.py --dir /tmp/try review
python3 agent/cli.py --dir /tmp/try --today 2026-11-12 alerts --days 7
python3 agent/cli.py --dir /tmp/try --today 2026-11-12 dashboard --open
```

`database.csv` ของโฟลเดอร์นี้ใช้ 11 คอลัมน์ (ไม่มี `model,token_input,token_output` ของ root) ข้อมูลจริง สลิป และ `my-dashboard.html` อยู่ในเครื่องเท่านั้น `.gitignore` กันไว้แล้ว
