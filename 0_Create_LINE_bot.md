# วิธีสร้าง LINE Bot

1. ล็อกอิน LINE ที่ [LINE Developers Console](https://developers.line.biz/en/)
2. คลิกที่ "Create a new provider" และกรอกข้อมูลตามที่ต้องการ _(ถ้ามี provider อยู่แล้ว สามารถข้ามขั้นตอนนี้ได้)_  
3. คลิกที่ "Create a Messaging API channel" และเลือก "Create a LINE Official Account"
4. กรอกข้อมูลตามที่ต้องการ และคลิกที่ "Create"

ณ ตอนนี้ คุณได้สร้าง LINE Bot แล้ว แต่ต้องตั้งค่าเพิ่มเติมก่อนหลังจากทำ Step 0 ของแต่ละ Workshop เสร็จแล้ว ดังนี้

> หากเคยทำขั้นตอนด้านล่างครบหมดแล้วสำหรับ Workshop แรก ใน Workshop ต่อ ๆ ไปสามารถทำแค่ขั้นตอนที่ 2 ได้ โดยทำการเปลี่ยนลิงก์ให้เป็น CodeSandbox ของแต่ละ Workshop แทน

1. ในหน้า Account ของบอต ไปที่แถบ "ตั้งค่า" เลือก "Messaging API" และคลิกที่ "ใช้ Messaging API" โดยทำการเลือก LINE Official Account ที่ได้สร้างไว้ก่อนหน้านี้
2. กรอกลิงก์ที่ได้จาก CodeSandbox ในช่อง "ลิงก์ Webhook"
3. กลับไปหน้า LINE Developers Console เลือกที่ Channel ของบอค ไปที่แถบ "Messaging API"
4. เลื่อนลงไปจนเจอ "Greeting messages", กด Edit ขวามือ จะได้หน้า Response settings ขึ้นมา
5. ตั้งค่าตามนี้
    - แชท: ปิด
    - ข้อความทักทายเพื่อนใหม่: ปิด
    - Webhooks: **เปิด**
    - ข้อความตอบกลับอัตโนมัติ: ปิด
6. ปิดหน้า Response settings, กลับมาที่หน้าเดิม เลื่อนลงไปล่างสุดที่ Channel access token ให้กด "Issue"

เป็นอันเสร็จสิ้น Bot ของคุณพร้อมใช้งานแล้ว
