# Workshop 2: สร้างบอตเริ่มฉลาด

ใน Workshop นี้ เราจะรับบทเป็นเจ้าของร้านค้า สร้างบอตหนึ่งตัวสำหรับสมาชิกเพื่อใช้ในการเช็คคะแนนสะสม และอีกหนึ่งตัวสำหรับร้านค้าเพื่อใช้ในการจัดการสมาชิก

## Step 0: Copy โปรเจค
1. ไปที่ https://codesandbox.io/p/devbox/e-membership-template-th-rlh7h5
2. กดปุ่ม "Fork" ที่มุมขวาบนของหน้า

หลังจาก Fork แล้ว โปรเจคที่เห็นในจอจะเปลี่ยนเป็นโปรเจคที่เป็นของคุณเอง และสามารถแก้ไขได้ตามต้องการ

3. กดปุ่ม "Share" ที่มุมขวาบนของหน้า
4. ที่ "Change permissions" ให้เลือก "Unlisted" เพื่อทำให้โปรเจคสามารถเข้าถึงได้
5. กดที่กล่องสี่เหลี่ยมที่มุมซ้ายบน แล้วกด "Restart Devbox"

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
3. คัดลอก Channel secret ไปแทนที่คำว่า `MEMBER_CHANNEL_SECRET` ในไฟล์ `member.js`
4. ไปที่หน้า "Messaging API" ของบอต
5. คัดลอก Channel access token ไปแทนที่คำว่า `MEMBER_CHANNEL_ACCESS_TOKEN` ในไฟล์เดียวกัน
6. ใน Codesandbox, กด Save (`Ctrl` + `S`), โปรแกรมจะเริ่มใหม่โดยอัตโนมัติ
7. ใน Codesandbox จอครึ่งขวาจะเป็น Browser ที่มีลิงค์อยู่ด้านบน ให้ Copy ลิงค์นั้นไปวางในหน้า "Messaging API" ของบอต ในช่อง "Webhook URL" แล้วต่อท้ายด้วย `/webhooks/member` 
> ตัวอย่าง `https://www.your-url.com/webhooks/member`
> ![image](https://github.com/tech-camp-day/workshop-cookbook/assets/47054457/2d70f3b5-6900-4c88-a892-0c107df5701f)

8. กด "Update" แล้วกด "Verify" เพื่อทดสอบการเชื่อมต่อระหว่าง LINE กับบอต; ถ้าทดสอบผ่านจะขึ้นคำว่า "Success"
9. กดเปิด "Use webhook" ถ้าหากยังไม่ได้เปิด
10. เชคให้มั่นใจว่าเราตั้งค่าบอทของเราตาม[ไฟล์นี้](0_2_Config_LINE_bot.md)

## Step 2: เพิ่มสมาชิกเข้าไปในระบบเมื่อสมาชิกเพิ่มเพื่อนบอตของเรา

1. ในไฟล์ `member.js` สร้างฟังก์ชัน `handleFollowEvent` ไว้ก่อนฟังก์ชัน `handleUnknownEvent` เพื่อเอาไว้จัดการ event ประเภท `follow` ที่เราจะได้รับตอนมีคนเพิ่มบอตของเราเป็นเพื่อน

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

## Step 3: ลบสมาชิกออกจากระบบเมื่อสมาชิกบล็อกบอตของเรา

1. ในไฟล์ `member.js` สร้างฟังก์ชัน `handleUnfollowEvent` ไว้ก่อนฟังก์ชัน `handleUnknownEvent` เพื่อเอาไว้จัดการ event ประเภท `unfollow` ที่เราจะได้รับตอนมีคนบล็อกเราหรือลบเราออกจากเพื่อน

```js
function handleUnfollowEvent(event) {
  const userId = event.source.userId;

  deleteMember(userId);
}
```

---
__คำอธิบาย__


ในการที่จะลบสมาชิกออกจากระบบ เราต้องใช้ `userId` ที่เราสามารถเอามาได้จาก `event.source.userId`

```js
const userId = event.source.userId;
```

พอได้ `userId` มาแล้ว เราก็ลบสมาชิกออกจากระบบด้วยการใช้ฟังก์ชัน `deleteMember` ที่เรามีอยู่แล้วได้เลย

```js
deleteMember(userId);
```

---

2. ในฟังก์ชัน `handleLineEvent` เราจะเรียกใช้ฟังก์ชัน `handleUnfollowEvent` ที่เราพึ่งสร้างไปเมื่อกี้ เมื่อ `event.type` คือ `unfollow` โดยเราสามารถเพิ่มโค้ดส่วนนี้เข้าไปใน `switch` statement ได้เลย

```js
case "unfollow":
    return handleUnfollowEvent(event);
```

หน้าตาฟังก์ชัน `handleLineEvent` ตอนนี้ก็จะเป็นแบบนี้

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    case "unfollow":
      return handleUnfollowEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

## Step 3.1 สมัครสมาชิกด้วยการ Block/Unblock
1. เนื่องจากฟังก์ชัน `handleFollowEvent` จะทำงานก็ต่อเมื่อเพิ่มเพื่อน เราจึงต้อง block บอทก่อนเหมือนเป็นการ unfollow
2. เข้าไปที่ LINE Bot ของเรา เลือก 3 ขีด
3. เลือก block
4. เลือก unblock เพื่อทำการเพิ่มเพื่อน บอทจะสวัสดีอีกครั้ง

## Step 4: ทำให้สมาชิกสามารถเช็คคะแนนสะสมได้

> เมื่อสมาชิกส่งคำว่า `เช็คคะแนน` ให้ส่งข้อความกลับไปหาสมาชิกว่า `คุณมีคะแนนสะสม XXX คะแนน`

1. ในไฟล์ `member.js` สร้างฟังก์ชัน `handleMessageEvent` ไว้ก่อนฟังก์ชัน `handleUnknownEvent` เพื่อเอาไว้จัดการ event ประเภท `message` ที่เราจะได้รับตอนมีคนส่งข้อความมาหาเรา

```js
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "เช็คคะแนน") {
    const member = getMemberByUid(event.source.userId);
    return replyToMember(event, `คุณมีคะแนนสะสม ${member.points} คะแนน`);
  }
}
```

---
__คำอธิบาย__

ในการที่เราจะเช็คว่าสมาชิกส่งคำว่า `เช็คคะแนน` มารึเปล่า เราจะใช้ข้อความ `text` ที่เราเอามาได้จาก `event.message.text`

```js
const text = event.message.text;
```

พอได้ข้อความมาแล้ว เราก็เอามาเช็คว่ามันตรงกับคำว่า `เช็คคะแนน` มั้ย 

```js
if (text === "เช็คคะแนน") {
    // some other code
}

```

ในการที่เราจะหาสมาชิก เราก็ต้องใช้ `userId` ที่เราหาได้จาก `event.source.userId` เหมือนเดิม

```js
const userId = event.source.userId;
```

พอได้ `userId` แล้ว เราก็หาสมาชิกได้ด้วยการเรียกใช้ฟังก์ชัน `getMemberByUid` ที่เรามีอยู่แล้ว

```js
const member = getMemberByUid(userId);
```

เสร็จแล้ว เราก็ส่งข้อมูลเกี่ยวกับคะแนนสะสมกลับไปให้สมาชิกได้เลย โดยการใช้ฟังก์ชัน `replyToMember`

```js
return replyToMember(event, `คุณมีคะแนนสะสม ${member.points} คะแนน`);
```

---

2. ในฟังก์ชัน `handleLineEvent` เราจะเรียกใช้ฟังก์ชัน `handleMessageEvent` ที่เราพึ่งสร้างไปเมื่อกี้ เมื่อ `event.type` คือ `message` โดยเราสามารถเพิ่มโค้ดส่วนนี้เข้าไปใน `switch` statement ได้เลย

```js
case "message":
    return handleMessageEvent(event);
```

หน้าตาฟังก์ชัน `handleLineEvent` ตอนนี้ก็จะเป็นแบบนี้

```js
function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    case "unfollow":
      return handleUnfollowEvent(event);
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}
```

3. ลองส่งข้อความหาน้องบอตดู

## Step 4.5: Member Cheat code

สำหรับคนที่ตามไม่ทัน ก็อปโค้ดนี้ไปแทนที่โค้ดในไฟล์ `member.js`, เปลี่ยน `MEMBER_CHANNEL_SECRET` และ `MEMBER_CHANNEL_ACCESS_TOKEN` แล้วกด Save แล้วลองเพิ่มเพื่อน บล็อก และปลดบล็อคดู

<details>
<summary>โค้ดเต็ม <code>member.js</code></summary>

```js
const express = require("express");
const line = require("@line/bot-sdk");
const { insertMember, deleteMember, getMemberByUid } = require("../persists");
const { replier, pusher } = require("./util");

const memberConfig = {
  channelSecret: "MEMBER_CHANNEL_SECRET",
  channelAccessToken:
    "MEMBER_CHANNEL_ACCESS_TOKEN",
};


const memberWebhookMiddleware = line.middleware(memberConfig);
const memberClient = new line.messagingApi.MessagingApiClient(memberConfig);
const replyToMember = replier(memberClient);
const pushToMember = pusher(memberClient);

const router = express.Router();

router.post("/", memberWebhookMiddleware, handlePostRequest);

/**
 * จัดการ request ที่เข้ามาที่ /webhooks/member
 * @param {Object} request - คำขอที่เข้ามา
 * @param {Object} response - การตอบกลับของคำขอ
 * @returns {Promise} - ส่งผลจากการจัดการ event กลับไป
 */
async function handlePostRequest(request, response) {
  // หยิบ events ออกมาจาก request body
  const { events } = request.body;

  // วนเรียกฟังก์ชัน handleLineEvent ที่ event แต่ละตัว
  const eventHandledPromises = events.map(handleLineEvent);

  // รอให้ event ทั้งหมดถูกจัดการ
  const result = await Promise.all(eventHandledPromises);

  // ส่งผลจากการจัดการ event กลับไป
  return response.send(result);
}

function handleLineEvent(event) {
  switch (event.type) {
    case "follow":
      return handleFollowEvent(event);
    case "unfollow":
      return handleUnfollowEvent(event);
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}

function handleFollowEvent(event) {
  const userId = event.source.userId;

  const { id: memberId } = insertMember(userId);
  pushToMember(
    userId,
    "ยินดีต้อนรับเข้าสู่ สมาชิก Bot ครับ กรุณาพิมพ์ 'ดูคะแนน' เพื่อเช็คคะแนนของคุณ",
    `เลขสมาชิกของคุณคือ ${memberId}`
  );
}

function handleUnfollowEvent(event) {
  const userId = event.source.userId;

  deleteMember(userId);
}

function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "ดูคะแนน") {
    const userId = event.source.userId;
    const member = getMemberByUid(userId);

    if (!member) {
      return replyToMember(
        event,
        "ขออภัย ตอนนี้คุณยังไม่ได้เป็นสมาชิกร้านค้า กรุณา Block แล้ว Unblock บอทเพื่อเริ่มใหม่"
      );
    }
    return replyToMember(event, `คุณมีคะแนนสะสม ${member.points} คะแนน`);
  } else {
    return replyToMember(event, "ไม่เข้าใจคำสั่งจ้า");
  }
}

function handleUnknownEvent(event) {
  return replyToMember(event, "ขอโทษครับ ฉันไม่เข้าใจคำสั่งของคุณ");
}

module.exports = {
  router,
  pushToMember,
};
```

</details>

## Step 5: ตั้งค่าบอตตัวที่หนึ่ง บอตฝั่งร้านค้า

1. สร้างบอตใหม่ตามขั้นตอนใน[ไฟล์นี้](0_Create_LINE_bot.md)
2. หลังจากสร้างบอตเสร็จแล้ว ไปที่หน้า "Basic settings" ของบอต
3. คัดลอก Channel secret ไปแทนที่คำว่า `MERCHANT_CHANNEL_SECRET` ในไฟล์ `merchant.js`
4. ไปที่หน้า "Messaging API" ของบอต
5. คัดลอก Channel access token ไปแทนที่คำว่า `MERCHANT_CHANNEL_ACCESS_TOKEN` ในไฟล์เดียวกัน
6. เลื่อนลงไปที่ "LINE Official Account features" ตรง "Auto-reply messages" ให้กด "Edit" แล้วก็กดปิดฟีเจอร์ทุกอย่างยกเว้น "Webhook"
7. ใน Codesandbox, กด Save (`Ctrl` + `S`), โปรแกรมจะเริ่มใหม่โดยอัตโนมัติ
8. ใน Codesandbox จอครึ่งขวาจะเป็น Browser ที่มีลิงค์อยู่ด้านบน ให้ Copy ลิงค์นั้นไปวางในหน้า "Messaging API" ของบอต ในช่อง "Webhook URL" แล้วต่อท้ายด้วย `/webhooks/merchant` 
> ตัวอย่าง `https://www.your-url.com/webhooks/merchant`
9. กด "Update" แล้วกด "Verify" เพื่อทดสอบการเชื่อมต่อระหว่าง LINE กับบอต; ถ้าทดสอบผ่านจะขึ้นคำว่า "Success"
10. กดเปิด "Use webhook" ถ้าหากยังไม่ได้เปิด แล้วลองส่งอะไรก็ได้ใน LINE ดู

## Step 5.5: อธิบายโค้ดเล็กน้อยก่อนเริ่ม (ข้ามได้ถ้าเทพแล้ว)

ใน `merchant.js` เรามีโค้ดอยู่แล้วบ้างเช่น

1. ฟังก์ชัน `handleLineEvent` ที่เราเอาไว้จัดการ event จาก Line
2. ฟังก์ชัน `handleMessageEvent` ที่เราเอาไว้จัดการ event ประเภท `message` ซึ่งเราได้ทำส่วน `จัดการสมาชิก` และการจัดการถ้าข้อความที่ส่งมาไม่ใช่สิ่งที่เรากำหนดไว้

## Step 5: ทำให้ร้านค้าสามารถเพิ่มคะแนนสะสมให้สมาชิกได้

1. ใน `merchant.js` สร้างฟังก์ชัน `handlePointIncrease` ไว้ก่อนฟังก์ชัน `handleUnknownEvent` เอาไว้จัดการการเพิ่มคะแนนสะสมของสมาชิก

```js
function handlePointIncrease(event, text) {
  const [_, memberId, pointToAddText] = text.split(" ");

  const member = getMemberById(memberId);

  if (!member) {
    return replyToMerchant(event, `ไม่พบสมาชิกหมายเลข ${memberId}`);
  }

  const pointToAdd = parseInt(pointToAddText);

  if (isNaN(pointToAdd)) {
    return replyToMerchant(event, "กรุณาใส่จำนวนคะแนนที่ต้องการเพิ่มเป็นตัวเลขเท่านั้น");
  }

  updatePointForMember(memberId, member.points + pointToAdd);
  pushToMember(member.uid, `คุณได้รับคะแนน ${pointToAdd} คะแนน`);

  return replyToMerchant(event, `เพิ่มคะแนน ${pointToAdd} คะแนนให้สมาชิกหมายเลข ${memberId} แล้ว`);
}
```

---
__คำอธิบาย__

เนื่องจากรูปแบบคำสั่งเพิ่มคะแนนให้สมาชิก จะมาในรูปแบบ `เพิ่มคะแนน <รหัสสมาชิก> <จำนวนแต้มที่ต้องการจะเพิ่ม>` เราจะต้องหยิบรหัสสมาชิก และแต้มที่ต้องการจะเพิ่มก่อน โดยการใช้

```js
const [_, memberId, pointToAddText] = text.split(" ");
```

ในการที่เราจะเพิ่มแต้ม เราจะต้องเอาแต้มที่มีอยู่ มาบวกกับ แต้มที่มีอยู่แล้ว `แต้มใหม่ = แต้มที่มีอยู่แล้ว + แต้มที่ต้องการจะเพิ่ม`  
เราก็เลยต้องหาสมาชิกที่เราต้องการจะเพิ่มแต้ม โดยการใช้ฟังก์ชัน `getMemberById` ที่เรามีอยู่แล้ว

```js
const member = getMemberById(memberId);
```

ถ้าเราหาสมาชิกคนนั้นไม่เจอ เราก็ไม่ควรจะเพิ่มแต้มได้ เราก็จะส่งข้อความไปหาร้านค้าว่า `ไม่พบสมาชิกหมายเลข XXXX` โดยใช้ฟังก์ชัน `replyToMerchant` ที่เรามีอยู่แล้วเพื่อตอบกลับไปที่ร้านค้า

```js
if (!member) {
    return replyToMerchant(event, `ไม่พบสมาชิกหมายเลข ${memberId}`);
}
```

ตอนนี้คะแนนที่เราได้มาจากคำสั่งของร้านค้าจะยังเป็นข้อความอยู่ เราก็ต้องแปลงมันให้กลายเป็นตัวเลขก่อน โดยการใช้ `parseInt`   แล้วถ้าสิ่งที่เรารับมาไม่ใช่ตัวเลข เราก็ส่งข้อความตอบกลับไปที่ร้านค้าว่า `กรุณาใส่จำนวนคะแนนที่ต้องการเพิ่มเป็นตัวเลขเท่านั้น`

```js
const pointToAdd = parseInt(pointToAddText);

if (isNaN(pointToAdd)) {
    return replyToMerchant(event, "กรุณาใส่จำนวนคะแนนที่ต้องการเพิ่มเป็นตัวเลขเท่านั้น");
}
```

พอเรามีทั้งรหัสสมาชิก และคะแนนใหม่ที่ต้องการจะเปลี่ยนแล้ว เราสามารถใช้ฟังก์ชัน `updatePointForMember` ในการอัพเดทคะแนนสะสมให้สมาชิกได้

```js
updatePointForMember(memberId, member.points + pointToAdd);
```

หลังจากนั้น เราก็สามารถที่จะส่งข้อความไปแจ้งเตือนสมาชิกได้ โดยการใช้ฟังก์ชัน `pushToMember`

```js
pushToMember(member.uid, `คุณได้รับคะแนน ${pointToAdd} คะแนน`);
```

แล้วสุดท้าย เราก็ส่งข้อความกลับไปหาร้านค้าว่า `เพิ่มคะแนน XXXX คะแนนให้สมาชิกหมายเลข YYY แล้ว`

```js
return replyToMerchant(event, `เพิ่มคะแนน ${pointToAdd} คะแนนให้สมาชิกหมายเลข ${memberId} แล้ว`);
```

---

2. ในฟังก์ชัน `handleMessageEvent` ให้เพิ่มเงื่อนไขที่เอาไว้เช็คว่ามีข้อความ เพิ่มคะแนน ส่งมามั้ย แล้วก็เรียกใช้ฟังก์ชัน `handlePointIncrease` ที่เราสร้างไปเมื่อสักครู่

```js
if (text.includes("เพิ่มคะแนน")) {
    return handlePointIncrease(event, text);
}
```

ใน `handleMessageEvent` ก็จะหน้าตาประมาณ

```js
function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "จัดการสมาชิก") {
    const members = getAllMembers();
    const memberInfoMessages = members.map((member) => `สมาชิกหมายเลข ${member.id} มีคะแนน ${member.points} คะแนน`);

    return replyToMerchant(event, ...memberInfoMessages);
  }

  if (text.includes("เพิ่มคะแนน")) {
    return handlePointIncrease(event, text);
  }

  return replyToMerchant(
    event,
    "กรุณาพิมพ์ 'จัดการสมาชิก' เพื่อดูข้อมูลสมาชิก เพื่อเพิ่มคะแนนให้สมาชิก",
    "หรือพิมพ์ 'เพิ่มคะแนน <รหัสสมาชิก> <คะแนน>' เพื่อเพิ่มคะแนนของสมาชิก",
  );
}
```

## Step 5.5: Merchant Cheat code

สำหรับคนที่ตามไม่ทัน ก็อปโค้ดนี้ไปแทนที่โค้ดในไฟล์ `merchant.js`, เปลี่ยน `MERCHANT_CHANNEL_SECRET` และ `MERCHANT_CHANNEL_ACCESS_TOKEN` แล้วกด Save แล้วลองเล่นดู

<details>
<summary>โค้ดเต็ม <code>merchant.js</code></summary>

```js
const express = require("express");
const line = require("@line/bot-sdk");
const {
  getMemberById,
  getAllMembers,
  updatePointForMember,
  getMemberByUid,
} = require("../persists");
const { replier } = require("./util");
const {
  pushToMember,
} = require("./member");

const merchantConfig = {
  channelSecret: "MERCHANT_CHANNEL_SECRET",
  channelAccessToken:
    "MERCHANT_ACCESS_TOKEN",
};

const merchantWebhookMiddleware = line.middleware(merchantConfig);
const merchantClient = new line.messagingApi.MessagingApiClient(merchantConfig);
const replyToMerchant = replier(merchantClient);

const router = express.Router();

router.post("/", merchantWebhookMiddleware, handleRequest);

/**
 * จัดการ request ที่เข้ามาที่ /webhooks/member
 * @param {Object} request - คำขอที่เข้ามา
 * @param {Object} response - การตอบกลับของคำขอ
 * @returns {Promise} - ส่งผลจากการจัดการ event กลับไป
 */
async function handleRequest(request, response) {
  // หยิบ events ออกมาจาก request body
  const { events } = request.body;

  // วนเรียกฟังก์ชัน handleLineEvent ที่ event แต่ละตัว
  const eventHandledPromises = events.map(handleLineEvent);

  // รอให้ event ทั้งหมดถูกจัดการ
  const result = await Promise.all(eventHandledPromises);

  // ส่งผลจากการจัดการ event กลับไป
  return response.send(result);
}

function handleLineEvent(event) {
  switch (event.type) {
    case "message":
      return handleMessageEvent(event);
    default:
      return handleUnknownEvent(event);
  }
}

function handleMessageEvent(event) {
  const text = event.message.text;

  if (text === "จัดการสมาชิก") {
    const members = getAllMembers();

    if (!members || members.length === 0) 
      return replyToMerchant(event, "ขณะนี้คุณไม่มีสมาชิก");
    const memberInfoMessages = members.map((member) => `สมาชิกหมายเลข ${member.id} มีคะแนน ${member.points} คะแนน`);

    return replyToMerchant(event, ...memberInfoMessages);
  }

  if (text.includes("เพิ่มคะแนน")) {
    return handlePointIncrease(event, text);
  }

  return replyToMerchant(
    event,
    "กรุณาพิมพ์ 'จัดการสมาชิก' เพื่อดูข้อมูลสมาชิก เพื่อเพิ่มคะแนนให้สมาชิก",
    "หรือพิมพ์ 'เพิ่มคะแนน <รหัสสมาชิก> <คะแนน>' เพื่อเพิ่มคะแนนของสมาชิก",
  );
}

function handlePointIncrease(event, text) {
  const [_, memberId, pointToAddText] = text.split(" ");

  if (!memberId || !pointToAddText)
    return replyToMerchant(event, "คำสั่งไม่ถูกต้อง เพิ่มคะแนนด้วยคำสั่ง 'เพิ่มคะแนน <รหัสสมาชิก> <คะแนน>'");

  const member = getMemberById(memberId);

  if (!member) {
    return replyToMerchant(event, `ไม่พบสมาชิกหมายเลข ${memberId}`);
  }

  const pointToAdd = parseInt(pointToAddText);

  if (isNaN(pointToAdd)) {
    return replyToMerchant(event, "กรุณาใส่จำนวนคะแนนที่ต้องการเพิ่มเป็นตัวเลขเท่านั้น");
  }

  updatePointForMember(memberId, member.points + pointToAdd);
  pushToMember(member.uid, `คุณได้รับคะแนน ${pointToAdd} คะแนน`);

  return replyToMerchant(event, `เพิ่มคะแนน ${pointToAdd} คะแนนให้สมาชิกหมายเลข ${memberId} แล้ว`);
}

/**
 * จัดการ event ที่ไม่รู้จัก
 * @param {Object} event - event ที่จะถูกจัดการ
 */
function handleUnknownEvent(event) {
  return replyToMerchant(event, "ขอโทษครับ ฉันไม่เข้าใจคำสั่งของคุณ");
}

module.exports = router;
```

</details>
