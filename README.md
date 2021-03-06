# D.I.Y : FSM Library for Unity
Finite State Machine Library for Unity

## Update #1 - 2015.09.24 11.28 AM
- ปรับโครงสร้างภายใน FSM Library
- สร้าง Invoker สำหรับการ Invoke Method แบบมี Delay เพื่อลด Overhead เวลามีการเรียกใช้ Invoke หรือ Coroutine หลายๆ ที่ พร้อมกัน
- สร้าง State Pool เพื่อ Reuse ใช้ State ที่เคยสร้างไว้และยังไม่ถูกทำลายทิ้ง
- สร้าง State Destroyer เพื่อทำการเคลียร์ State ที่ยังไม่ Exit ก่อนเปลี่ยนฉากหรือปิดโปรแกรม
- สร้าง State Updater เพื่อเป็นศูนย์กลางงาน Update ของทุก State ไม่ต้องเปลือง memory สร้าง MonoBehaviour ของแต่ละ State

## Prologue

ไลบรารี่ชุดนี้เป็นผลพลอยได้มาจากการทำเกมตัวอย่างเพื่อใช้ในการสอนนักศึกษา (ดูได้ที่โฟลเดอร์ Platform-2D) ผนวกกับเป็นคนที่เขียนโปรแกรมแบบไม่ค่อยแบ็คอัพโค้ด ทำให้ผมเองก็อยากลองเขียนอะไรที่เอาไปใช้ซ้ำกับงานอื่นๆ ได้ก็เลยเอาขึ้น Github ซะเลย

แนวคิดของไลบรารี่ตัวนี้มาจากการที่ผมเป็นคนชอบใช้ Play Maker (Unity Plugin) ซึ่งมันเป็น Visual Script ที่อยู่บนพื้นฐานการทำงานโดยใช้ FSM และจากที่ผมเคยเขียนถึงเรื่องการเขียนเอไอแบบเบื้องต้นไป (เมื่อไรจะได้เขียนเบื้องลึกสักทีนะ =_=) ก็ได้มานั่งออกแบบเจ้าไลบรารี่ตัวนี้ จริงๆ ก็คิดไว้หลายแบบแต่ล่าสุดก็ยึดตามที่ใช้แล้วรู้สึกโอเค ตอนเทสก็เอาไปใช้กับเกมนั่นแล ตอนเริ่มต้นเขียนตัวเกมยังไม่ได้ใช้ FSM ตั้งแต่แรก เขียนแบบเอเวอรี่ติงจิงกะเบลไว้ในไฟล์เดียวพอได้ลอจิกโดยรวมแล้วก็ปรับเป็น FSM ทีละนิดแล้วดูว่าแบบไหนดี

สำหรับใครที่ไม่รู้ว่า FSM หน้าตาเป็นไงก็เข้าไปศึกษาตามลิงก์ได้ครับ มีหลากหลายลีลา เอ๊ย หลายสไตล์ให้เลือกชม

http://wiki.unity3d.com/index.php?title=Finite_State_Machine (อันนี้ดีนะ ผมชอบ 555+)

http://devdayo.com/2014/11/02/ai-beginning/

http://devdayo.com/2014/11/02/ai-beginning-2/

http://devdayo.com/2014/11/02/ai-beginning-3/

ตอนพัฒนาก็ปรับนู่นนี่นั่น พิมพ์กันจนเมื่อยแขน ฮ่าๆ มีหลายจุดที่ต้องคิด เช่น เรื่องการเปลี่ยน State จะมีปัญหาอะไรบ้าง ถ้ามีการสั่งเปลี่ยนจาก Update กับ FixedUpdate พร้อมๆ กันจะมีปัญหาไหมนะ หรือสั่งเปลี่ยนรัวๆ จะบั๊กหรือเปล่า ? ตรงนี้ก็นั่งดักนู่นดักนี่ แล้วก็ทำตัวกลางสำหรับเปลี่ยน State ซะเลยเรียงคิวกันจะได้ไม่วุ่นวาย

อีกเรื่องก็เป็นเรื่องของพวก Event อย่าง Collision, Trigger อีก ตอนศึกษา FSM มาเขียนเอไอ ในหนังสือมันมีแค่ Enter, Stay (Update), Exit เองนี่หว่า เอาไงดีฟะ ? ลองให้ MonoBehaviour มันเป็น FSM ไปเลยแล้วกันแล้วก็ประกาศพวกเมธอด Update, FixedUpdate, Collision, Trigger เอาไว้แต่ไปๆ มาๆ ก็เจอเรื่องที่ถึงจะประกาศ Collision ไว้ก็ใช้ไม่ได้เพราะตัวที่ชนมันดันเป็น Child Object ซะงั้น ก็เลยตั้งไปนั่งศึกษา Observer Pattern กับ Delegate เพิ่มเพื่อจะเอามาใช้

สุดท้ายพวกเมธอดต่างๆ ของ MonoBehaviour ก็ทำให้อยู่ในรูปแบบ Observable ซะ State ไหนอยากใช้ก็แค่ Subscribe Event ที่ต้องการแล้วก็ระบุไปว่าอยากให้ Event ไปผูกกับ Object ไหนก็ได้ตามต้องการ (ไว้รอเสร็จแล้วจะทำวีดีโอแนะนำนะ อิอิ) ตัว FSM และ MonoBehaviour หลักก็เลยบางเบาประหยัดพลังงานในระดับนึง

พวกความรู้ต่างๆ ที่เอามาใช้เสริมก็มีพวก

http://devdayo.com/2013/10/23/action-in-csharp/

http://devdayo.com/2014/11/03/action-in-c-multicast/

http://devdayo.com/2013/10/22/generic-type/

http://devdayo.com/2013/10/23/generic-class/

http://devdayo.com/2013/10/23/generic-method/

http://devdayo.com/2013/11/10/reflection-on-csharp/

http://devdayo.com/2014/10/17/c-sharp-dictionary/
