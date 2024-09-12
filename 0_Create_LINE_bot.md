# วิธีสร้าง LINE Bot

1. ลงชื่อเข้าใช้ LINE ที่ [LINE Developers Console](https://developers.line.biz/en/)
2. คลิกที่ "Create a new provider" และกรอกข้อมูลที่จำเป็น _(ถ้าคุณมี provider อยู่แล้ว คุณสามารถข้ามขั้นตอนนี้ไปได้)_
![Create a new provider](images/create-new-provider.png)
3. คลิกที่ "Create a new channel" และเลือก "Messaging API"
![alt text](images/create-new-messaging-api-channel.png)
4. คลิกที่ "Create a LINE Official Account"
5. กรอกข้อมูลที่จำเป็นและคลิก "Create"
![alt text](images/filling-form.png))
6. เมื่อมาถึงขั้นตอนที่ 3 ของการสร้าง Channel แล้ว ให้เลื่อนลงและคลิก "Later"
![alt text](images/later-to-manager-page.png)
7. คลิกที่ "Settings" ที่มุมขวาบน
![alt text](images/top-right-setting.png)
8. คลิกที่ "Messaging API" และจากนั้นคลิก "Enable Messaging API"
![alt text](images/enable-messaging-api.png)
9. เลือก provider ที่เราสร้างขึ้นในข้อ 1
![alt text](images/choose-provider.png)
10. ในหน้า "Privacy Policy and Terms of Use" คลิก "OK" ได้เลย
![alt text](images/policy-and-terms.png)
11. กลับไปที่ [LINE Developers Console](https://developers.line.biz/console)
12. เลือก Channel ที่คุณเพิ่งสร้าง
![alt text](images/select-created-channel.png)
13. ใน Basic settings คลิก issue "Channel secret"
14. ใน Messaging API คลิก issue "Channel access token"

ณ ตอนนี้ คุณได้สร้าง LINE Bot แล้ว

เดี๋ยวเราจะกลับมาตั้งค่า bot ของเราเพิ่มทีหลัง
