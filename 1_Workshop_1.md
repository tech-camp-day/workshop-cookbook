# Workshop 1: สร้างบอทอ่อนๆ

ใน Workshop นี้ เราจะสร้างบอทที่สามารถทำนายอนาคตของคุณ โดยการสุ่มคำทำนายจากข้อมูลที่เรากำหนดไว้

## Step 0: Copy โปรเจค

1. ไปที่ https://codesandbox.io/p/devbox/fortune-teller-bot-starter-32fy4v
2. กดปุ่ม "Fork" ที่มุมขวาบนของหน้า

หลังจาก Fork แล้ว โปรเจคที่เห็นในจอจะเปลี่ยนเป็นโปรเจคที่เป็นของคุณเอง และสามารถแก้ไขได้ตามต้องการ

3. กดปุ่ม "Share" ที่มุมขวาบนของหน้า
4. กดปุ่ม "Move out of Drafts"
5. ที่ "Change permissions" ให้เลือก "Unlisted" เพื่อทำให้โปรเจคสามารถเข้าถึงได้
6. กดที่กล่องสี่เหลี่ยมที่มุมซ้ายบน แล้วกด "Restart Devbox"


## Step 1: ตั้งค่าบอท

1. สร้างบอทใหม่ตามขั้นตอนใน[ไฟล์นี้](0_Create_LINE_bot.md)
2. หลังจากสร้างบอทเสร็จแล้ว ไปที่หน้า "Basic settings" ของบอท
3. คัดลอก Channel secret ไปแทนที่คำว่า `CHANNEL_SECRET` ในไฟล์ `index.js` ที่บรรทัดที่ 6
4. ไปที่หน้า "Messaging API" ของบอท
5. คัดลอก Channel access token ไปแทนที่คำว่า `CHANNEL_ACCESS_TOKEN` ในไฟล์เดียวกันที่บรรทัดที่ 7
6. ใน Codesandbox, กด Save (`Ctrl` + `S`), โปรแกรมจะเริ่มใหม่โดยอัตโนมัติ
7. ใน Codesandbox จอครึ่งขวาจะเป็น Browser ที่มีลิงค์อยู่ด้านบน ให้ Copy ลิงค์นั้นไปวางในหน้า "Messaging API" ของบอท ในช่อง "Webhook URL"
8. กด "Update" แล้วกด "Verify" เพื่อทดสอบการเชื่อมต่อระหว่าง LINE กับบอท; ถ้าทดสอบผ่านจะขึ้นคำว่า "Success"
9. กดเปิด "Use webhook" ถ้าหากยังไม่ได้เปิด


## Step 1.5: อธิบายโค้ดเล็กน้อยก่อนเริ่ม (ข้ามได้ถ้าเทพแล้ว)

โปรเจคที่เพิ่งก็อปไปประกอบไปด้วยไฟล์สำคัญ 2 ไฟล์ คือ
- `index.js` ไฟล์หลักของโปรเจค ที่เราจะเขียนโค้ดของบอท โปรแกรมจะเริ่มทำงานจากไฟล์นี้
- `fortune.json` เก็บข้อมูลคำทำนายเพื่อใช้ส่งกลับไปยังผู้ใช้


## Step 2: ทำให้ "ดูดวง Bot" ตอบข้อความได้

1. ในไฟล์ `index.js`, เพิ่มโค้ดต่อไปนี้ใต้ตัวแปร `lineConfig` บรรทัดที่ 9

```javascript
const messagingClient = new line.messagingApi.MessagingApiClient(lineConfig);
```

`messagingClient` ที่เพิ่งสร้างนี้เป็นตัวช่วยในการใช้งาน LINE เพื่อส่งข้อความกลับไปยังผู้ใช้

2. เพิ่มโค้ดต่อไปนี้ที่ล่างสุดของไฟล์ `index.js`

```javascript
function reply(event, ...messages) {
  return messagingClient.replyMessage({
    replyToken: event.replyToken,
    messages: messages.map(message => ({
      type: "text",
      text: message,
    })),
  });
}
```

ฟังก์ชั่น `reply` เรียกใช้ตัวช่วย `messagingClient` ให้เราส่งข้อความกลับไปยังผู้ใช้ได้ง่ายขึ้น

3. และเพิ่มโค้ดต่อไปนี้ที่ล่างสุดของไฟล์ `index.js` ต่อจากโค้ดที่เพิ่งเพิ่มไป

```javascript
async function handleLineEvent(event) {
  return reply(event, "สวัสดีจากดูดวง Bot");
}
```

4. กด Save แล้วลองส่งอะไรก็ได้ใน LINE ดู


## Step 3: สุ่มคำทำนาย

1. สร้าง constant แทนค่าดวงแต่ละประเภทเพื่อความสะดวกในการใช้งาน

```js
const FortuneCategory = {
  Love: "love",
  Career: "career",
  Finance: "finance",
  Health: "health",
};
```

2. สร้างฟังก์ชั่น `getFortune` สำหรับการสุ่มดวงในประเภทนั้นๆ

```js
function getFortune(category) {
  const index = Math.floor(Math.random() * fortunes[category].length);
  return fortunes[category][index];
}
```

3. สร้างตัวแปร `counter` ไว้เช็คว่าเราดูดวงไปกี่รอบแล้ว

```js
let count = 1;
```

4. สร้างฟังก์ชัน `getAllFortuneMessages` สำหรับสุ่มดวงประเภทละอัน แล้วเอามาทำเป็นข้อความ Line

```js
function getAllFortuneMessages() {
  const fortuneMessages = [
    `ดูดวงประจำวันครั้งที่ ${count++}`,
    `ด้านความรัก\n${getFortune(FortuneCategory.Love)}`,
    `ด้านการงาน\n${getFortune(FortuneCategory.Career)}`,
    `ด้านการเงิน\n${getFortune(FortuneCategory.Finance)}`,
    `ด้านสุขภาพ\n${getFortune(FortuneCategory.Health)}`,
  ];

  return fortuneMessages;
}
```

5. เปลี่ยนฟังก์ชั่น `handleLineEvent` ให้ตอบข้อความด้วยดวงที่เราพึ่งสุ่มมา

```js
async function handleLineEvent(event) {
  return reply(event, ...getAllFortuneMessages());
}
```

6. กด Save แล้วลองส่งอะไรก็ได้ใน LINE ดู

### Step 4: ส่งดวงกลับมาแค่ตอนที่เราพิมพ์ไปว่า "ดูดวง"

ถึงตอนนี้ บอทจะให้ดวงกลับมาทุกครั้งที่เราส่งข้อความไป แต่เราต้องการให้บอทตอบดวงกลับมาเฉพาะเมื่อเราพิมพ์ไปว่า "ดูดวง" เท่านั้น

1. เปลี่ยนฟังก์ชั่น `handleLineEvent` ให้เช็คก่อนว่าข้อความที่ถูกส่งมาคือคำว่า "ดูดวง" รึเปล่า

```js
async function handleLineEvent(event) {
  // หยิบเอา text จาก message ใน event ออกมา
  const { text } = event.message;

  if (text === "ดูดวง") {
    return reply(event, ...getAllFortuneMessages());
  }
}
```

2. กด Save แล้วลองส่งอะไรก็ได้ใน LINE ดู; ถ้าไม่ได้ส่ง "ดูดวง" บอทจะไม่ตอบอะไรกลับมา

3. เปลี่ยนฟังก์ชั่น `handleLineEvent` อีกครั้งให้ส่งอย่างอื่นกลับไปถ้าข้อความที่ถูกส่งมาไม่ใช่คำว่า "ดูดวง"

```js
async function handleLineEvent(event) {
  // หยิบเอา text จาก message ใน event ออกมา
  const { text } = event.message;

  if (text === "ดูดวง") {
    return reply(event, ...getAllFortuneMessages());
  } else {
    return reply(event, ???); // << ใส่ข้อความตรง ???
  }
}
```

4. กด Save แล้วลองส่งอะไรก็ได้ใน LINE ดู



## Step 99: ถ้าไม่ทัน

<details>
<summary>โค้ดเต็ม index.js</summary>

```javascript
const line = require("@line/bot-sdk");
const express = require("express");
const fortunes = require("./fortune.json");

const lineConfig = {
  channelSecret: "CHANNEL_SECRET",
  channelAccessToken: "CHANNEL_ACCESS_TOKEN",
};
const messagingClient = new line.messagingApi.MessagingApiClient(lineConfig);

const app = express();
app.post("/", line.middleware(lineConfig), handlePostRequest);
app.listen(3000);

async function handlePostRequest(req, res) {
  const { events } = req.body;

  const eventHandledPromises = events.map(handleLineEvent);

  const result = await Promise.all(eventHandledPromises);

  return res.send(result);
}

function reply(event, ...messages) {
  return messagingClient.replyMessage({
    replyToken: event.replyToken,
    messages: messages.map((message) => ({
      type: "text",
      text: message,
    })),
  });
}

async function handleLineEvent(event) {
  const { text } = event.message;

  if (text === "ดูดวง") {
    return reply(event, ...getAllFortuneMessages());
  } else {
    return reply(event, "ไม่เข้าใจจ้า พิมพ์ 'ดูดวง' เพื่อดูดวง");
  }
}

const FortuneCategory = {
  Love: "love",
  Career: "career",
  Finance: "finance",
  Health: "health",
};

function getFortune(category) {
  const index = Math.floor(Math.random() * fortunes[category].length);
  return fortunes[category][index];
}

let count = 1;

function getAllFortuneMessages() {
  const fortuneMessages = [
    `ดูดวงประจำวันครั้งที่ ${count++}`,
    `ด้านความรัก\n${getFortune(FortuneCategory.Love)}`,
    `ด้านการงาน\n${getFortune(FortuneCategory.Career)}`,
    `ด้านการเงิน\n${getFortune(FortuneCategory.Finance)}`,
    `ด้านสุขภาพ\n${getFortune(FortuneCategory.Health)}`,
  ];

  return fortuneMessages;
}
```

ก็อปโค้ดนี้ไปแทนที่โค้ดในไฟล์ `index.js`, เปลี่ยน `CHANNEL_SECRET` และ `CHANNEL_ACCESS_TOKEN` บรรทัดที่ 6-7 แล้วกด Save แล้วลองส่งอะไรก็ได้ใน LINE ดู

</details>