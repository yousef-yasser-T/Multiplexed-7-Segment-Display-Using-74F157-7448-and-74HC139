

# 🔢 BCD Multiplexed 7-Segment Display Driver

عرض رقمين BCD (A و B) على شاشتين Common-Cathode باستخدام تقنية الـ **Multiplexing**، مبني على 74HC157 (Multiplexer) + 7448/7447 (BCD to 7-Segment Decoder) + 74HC139 (Decoder للتحكم في تفعيل كل شاشة).

> مبني على الشكل التوضيحي **Figure 6-49: Simplified 7-Segment Display Multiplexing Logic**.

---

## 📖 فكرة المشروع

بدل ما نستخدم دائرة Decoder منفصلة لكل شاشة (وده مكلف وبياخد Pins كتير)، بنستخدم دائرة واحدة (7448) ونعمل **Multiplexing** بين الشاشتين بسرعة عالية باستخدام موجة مربعة (Data Select)، بحيث:

- لما `Data Select = LOW` → يتم عرض رقم **A** على الشاشة الأولى (LSD).
- لما `Data Select = HIGH` → يتم عرض رقم **B** على الشاشة الثانية (MSD).
- التبديل بيحصل بسرعة كافية (100Hz – 1kHz) بحيث العين البشرية تشوف الشاشتين شغالين في نفس الوقت بدون Flicker.

---

## 🧩 مكونات الدائرة

| المكون | الوظيفة |
|---|---|
| **74HC157** | Multiplexer رباعي (2-line to 1-line) لاختيار بين رقمي BCD |
| **7448 / 7447** | BCD to 7-Segment Decoder (Common Cathode) |
| **74HC139** | 2-line to 4-line Decoder، يستخدم هنا لتفعيل الشاشة المناسبة (Digit Enable) |
| **2× Common Cathode 7-Segment Display** | لعرض الرقمين |
| **Square Wave Generator** (555 Timer / Arduino / MCU) | لتوليد إشارة Data Select |
| **مقاومات 220Ω × 7** | لحماية الـ Segments |

---

## 🔌 التوصيلات

### 1) 74HC157 — Multiplexer

| Pin | الاسم | التوصيل |
|---|---|---|
| 16 | Vcc | +5V |
| 8 | GND | Ground |
| 15 | Select (S) | Data Select (الموجة المربعة) |
| 1 | Enable (G̅) | GND (تفعيل دائم) |

**المداخل:**

| Pin | الإشارة |
|---|---|
| 2 (1A) | A0 |
| 3 (1B) | B0 |
| 5 (2A) | A1 |
| 6 (2B) | B1 |
| 11 (3A) | A2 |
| 10 (3B) | B2 |
| 14 (4A) | A3 |
| 13 (4B) | B3 |

**المخارج → إلى 7448:**

| Pin | يذهب إلى |
|---|---|
| 4 (1Y) | 7448 – Pin A |
| 7 (2Y) | 7448 – Pin B |
| 9 (3Y) | 7448 – Pin C |
| 12 (4Y) | 7448 – Pin D |

---

### 2) 7448 — BCD to 7-Segment Decoder

| Pin | التوصيل |
|---|---|
| 16 | +5V |
| 8 | GND |

**المداخل (من MUX):**

| Pin | جاي من |
|---|---|
| 7 (A) | 74HC157 Pin 4 |
| 1 (B) | 74HC157 Pin 7 |
| 2 (C) | 74HC157 Pin 9 |
| 6 (D) | 74HC157 Pin 12 |

**المخارج (عبر مقاومة 220Ω لكل Segment، متوازية على الشاشتين):**

| Pin | Segment |
|---|---|
| 13 | a |
| 12 | b |
| 11 | c |
| 10 | d |
| 9 | e |
| 15 | f |
| 14 | g |

---

### 3) 74HC139 — Digit Enable Decoder

| Pin | التوصيل |
|---|---|
| 16 | +5V |
| 8 | GND |
| 1 (Enable G̅) | GND (تفعيل) |
| 2 (A) | Data Select (نفس إشارة الـ MUX) |
| 3 (B) | GND |

**المخارج (Active LOW، تتحكم في Common Cathode لكل شاشة):**

| Pin | يتصل بـ |
|---|---|
| 4 (Y0) | Common Cathode لشاشة A (LSD) |
| 5 (Y1) | Common Cathode لشاشة B (MSD) |

> ⚠️ الخرج **LOW** هو اللي بيشغّل الشاشة (لأنها Common Cathode). فلو الإضاءة ضعيفة يُفضّل استخدام ترانزستور Driver على مخارج 74HC139.

---

### 4) شاشات السفن سيجمنت

- كل Segment (a–g) موصول بالتوازي على الشاشتين، خارج من 7448 عبر مقاومة.
- Common Cathode لكل شاشة موصول لمخرج مختلف من 74HC139 (Y0 / Y1).

---

## ⏱️ إشارة التحكم (Data Select)

- موجة مربعة (Square Wave)، يمكن توليدها من:
  - IC 555 Timer (Astable Mode)
  - Arduino / أي Microcontroller (باستخدام PWM أو Toggle Loop)
- **التردد الموصى به:** 100Hz – 1kHz لتفادي وميض (Flicker) العين.

---

## 🖼️ مخطط الدائرة

يرجى إضافة صورة `circuit-diagram.png` هنا (المرفقة أصلاً بالمشروع) لتوضيح التوصيلات بصرياً.

```
[BCD A] ─┐
         ├─► 74HC157 (MUX) ─► 7448 (Decoder) ─► 2× 7-Seg Display
[BCD B] ─┘         ▲                                    ▲
                    │                                    │
             Data Select ────────────────► 74HC139 (Digit Enable)
```

---

## ⚠️ ملاحظات مهمة

- استخدم مقاومة 220Ω لكل Segment لحمايته من التيار الزائد.
- IC 7448 مخصص لـ **Common Cathode** فقط، تأكد من نوع الشاشة قبل التوصيل.
- لو استخدمت أكثر من شاشتين، تقدر توسّع الفكرة باستخدام 74HC139 كامل (4 مخارج) بدل نصفه فقط.
- تأكد إن تردد الـ Data Select ثابت ومناسب؛ تردد منخفض جداً هيسبب وميض واضح.
- ممكن تحتاج Buffer/Driver إضافي على مخارج الـ Decoder لو الشاشات كبيرة أو الاستهلاك عالي.

---

## 📚 المرجع

مبني على: *Figure 6-49 — Simplified 7-Segment Display Multiplexing Logic* (من كتاب Digital Systems: Principles and Applications).

---

## 📄 الترخيص

هذا المشروع تعليمي (Educational Project) — استخدمه وعدّله بحرية.
