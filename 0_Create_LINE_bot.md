# วิธีสร้าง LINE Bot

1. ล็อกอิน LINE ที่ [LINE Developers Console](https://developers.line.biz/en/)
2. คลิกที่ "Create a new provider" และกรอกข้อมูลตามที่ต้องการ _(ถ้ามี provider อยู่แล้ว สามารถข้ามขั้นตอนนี้ได้)_
3. คลิกที่ "Create a new channel" และเลือก "Messaging API"
4. กรอกข้อมูลตามที่ต้องการ และคลิกที่ "Create"

ณ ตอนนี้ คุณได้สร้าง LINE Bot แล้ว แต่ต้องทำการตั้งค่าเพิ่มเติมก่อนดังนี้
1. ในหน้า Channel ของบอท ไปที่แถบ "Messaging API"
2. เลื่อนลงไปจนเจอ "Greeting messages", กด Edit ขวามือ จะได้หน้า Response settings ขึ้นมา
3. ตั้งค่าตามนี้
  - Chat: ปิด
  - Greeting message: ปิด
  - Webhooks: **เปิด**
  - Auto-response messages: ปิด
  - Response hours: ปิด
4. ปิดหน้า Response settings, กลับมาที่หน้าเดิม เลื่อนลงไปล่างสุดที่ Channel access token ให้กด "Issue"

เป็นอันเสร็จสิ้น Bot ของคุณพร้อมใช้งานแล้ว