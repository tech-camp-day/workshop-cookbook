# Workshop 2: สร้างบอตเริ่มฉลาด

ใน Workshop นี้ เราจะรับบทเป็นเจ้าของร้านค้า สร้างบอตหนึ่งตัวสำหรับสมาชิกเพื่อใช้ในการเช็คคะแนนสะสม และอีกหนึ่งตัวสำหรับร้านค้าเพื่อใช้ในการจัดการสมาชิก

## Step 0: Copy โปรเจค
1. ไปที่ https://codesandbox.io/p/devbox/e-membership-starter-zxdpgs
2. กดปุ่ม "Fork" ที่มุมขวาบนของหน้า

หลังจาก Fork แล้ว โปรเจคที่เห็นในจอจะเปลี่ยนเป็นโปรเจคที่เป็นของคุณเอง และสามารถแก้ไขได้ตามต้องการ

3. กดปุ่ม "Share" ที่มุมขวาบนของหน้า
4. กดปุ่ม "Move out of Drafts"
5. ที่ "Change permissions" ให้เลือก "Unlisted" เพื่อทำให้โปรเจคสามารถเข้าถึงได้
6. กดที่กล่องสี่เหลี่ยมที่มุมซ้ายบน แล้วกด "Restart Devbox"

## Step 0.5: อธิบายโค้ดเล็กน้อยก่อนเริ่ม (ข้ามได้ถ้าเทพแล้ว)

โปรเจคที่เพิ่งก็อปไปประกอบไปด้วยไฟล์สำคัญ 4 ไฟล์ คือ
- `index.js` ไฟล์หลักของโปรเจค โปรแกรมจะเริ่มทำงานจากไฟล์นี้
- `persists.js` รวมฟังก์ชันต่างๆ ที่เกี่ยวกับการติดต่อกับฐานข้อมูล Database เอาไว้ในนั้น
- `member.js` โค้ดของบอตตัวที่หนึ่ง บอตฝั่งสมาชิก
- `merchant.js` โค้ดของบอตตัวที่สอง บอตฝั่งร้านค้า

นอกจากนั้น เรายังมีฟังก์ชันที่เอาไว้ใช้ในการส่งข้อความได้แก่
- `pushToMember` ใช้สำหรับการส่งข้อความแบบ `push` ไปยังสมาชิก
- `replyToMember` ใช้สำหรับการส่งข้อความตอบกลับไปยังสมาชิก
- `replyToMerchant` ใช้สำหรับการส่งข้อความตอบกลับไปยังร้านค้า


## Step 1: ตั้งค่าบอตตัวที่หนึ่ง บอตฝั่งสมาชิก

1. สร้างบอตใหม่ตามขั้นตอนใน[ไฟล์นี้](0_Create_LINE_bot.md)
2. หลังจากสร้างบอตเสร็จแล้ว ไปที่หน้า "Basic settings" ของบอต
3. คัดลอก Channel secret ไปแทนที่คำว่า `MEMBER_CHANNEL_SECRET` ในไฟล์ `member.js` ที่บรรทัดที่ 5
4. ไปที่หน้า "Messaging API" ของบอต
5. คัดลอก Channel access token ไปแทนที่คำว่า `MEMBER_CHANNEL_ACCESS_TOKEN` ในไฟล์เดียวกันที่บรรทัดที่ 6
6. เลื่อนลงไปที่ "LINE Official Account features" ตรง "Auto-reply messages" ให้กด "Edit" แล้วก็กดปิดฟีเจอร์ทุกอย่างยกเว้น "Webhook"
7. ใน Codesandbox, กด Save (`Ctrl` + `S`), โปรแกรมจะเริ่มใหม่โดยอัตโนมัติ
8. ใน Codesandbox จอครึ่งขวาจะเป็น Browser ที่มีลิงค์อยู่ด้านบน ให้ Copy ลิงค์นั้นไปวางในหน้า "Messaging API" ของบอต ในช่อง "Webhook URL" แล้วต่อท้ายด้วย `/webhooks/member` (ตัวอย่าง `https://www.your-url.com/webhooks/member`)
9. กด "Update" แล้วกด "Verify" เพื่อทดสอบการเชื่อมต่อระหว่าง LINE กับบอต; ถ้าทดสอบผ่านจะขึ้นคำว่า "Success"
10. กดเปิด "Use webhook" ถ้าหากยังไม่ได้เปิด แล้วลองส่งอะไรก็ได้ใน LINE ดู

## Step 2: เพิ่มสมาชิกเข้าไปในระบบเมื่อสมาชิกเพิ่มเพื่อนบอตของเรา

1. ในไฟล์ `member.js` สร้างฟังก์ชัน `handleFollowEvent` ไว้ที่บรรทัดที่ 46 เพื่อเอาไว้จัดการ event ประเภท `follow` ที่เราจะได้รับตอนมีคนเพิ่มบอตของเราเป็นเพื่อน

```js
function handleFollowEvent(event) {
    const userId = event.source.userId;

    insertMember(userId);
    pushToMember(userId, "ยินดีต้อนรับเข้าสู่ สมาชิก Bot ครับ กรุณาพิมพ์ 'เช็คคะแนน' เพื่อเช็คคะแนนของคุณ");
}
```

---
__คำอธิบาย__


ในการที่จะเพิ่มสมาชิกเข้าไปในระบบ เราต้องใช้ `userId` ที่เราสามารถเอามาได้จาก `event.source.userId`

```js
const userId = event.source.userId;
```

พอได้ `userId` มาแล้ว เราก็เพิ่มสมาชิกเข้าไปในระบบด้วยการใช้ฟังก์ชัน `insertMember` ที่เรามีอยู่แล้ว

```js
insertMember(userId);
```

เสร็จแล้วเราก็ส่งข้อความต้อนรับไปหาสมาชิกด้วยฟังก์ชัน `pushToMember` ได้เลย

```js
pushToMember(userId, "ยินดีต้อนรับเข้าสู่ สมาชิก Bot ครับ กรุณาพิมพ์ 'เช็คคะแนน' เพื่อเช็คคะแนนของคุณ");
```

---

2. ในฟังก์ชัน `handleLineEvent` เราจะเรียกใช้ฟังก์ชัน `handleFollowEvent` ที่เราพึ่งสร้างไปเมื่อกี้ เมื่อ `event.type` คือ `follow` โดยเราสามารถเพิ่มโค้ดส่วนนี้เข้าไปใน `switch` statement ได้เลย

```js
case "follow":
    return handleFollowEvent(event);
```

หน้าตาฟังก์ชัน `handleLineEvent` ตอนนี้ก็จะเป็นแบบนี้

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

3. ลองเพิ่มบอตเป็นเพื่อนดู