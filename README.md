# TIMER — Turn Your Time Into Progress

منصة إنتاجية بأسلوب الألعاب: مؤقت تركيز، نقاط، متجر، مهارات، عجلة حظ. هذا هو **Phase 5** (يتضمن المراحل السابقة؛ المتجر في Phase 4 لم يُنفَّذ بعد).

## حالة المشروع

| الجزء | الحالة |
|---|---|
| هيكل المشروع، قاعدة البيانات (كل الجداول)، تسجيل/دخول/خروج | منفّذ (Phase 1) |
| **المهام**: إضافة، تعديل، حذف، إكمال، إعادة فتح (لكل مستخدم فقط) | منفّذ |
| **مؤقت التركيز** (الخادم هو المرجع: بدء، إيقاف، استئناف، إنهاء، إلغاء) | منفّذ |
| **النقاط والـ XP**: كل فترة تركيز مكتملة تمنح نقاطاً وXP مرة واحدة فقط، مع سجل (Ledger) | منفّذ |
| **المستويات** والألقاب، هدف اليوم، احتفال عند رفع المستوى، أصوات | منفّذ |
| **المكافأة اليومية** مع سلسلة أيام متتالية (Streak) ومسار 7 أيام، حسب توقيت القاهرة | منفّذ |
| **عجلة الحظ**: لفّة مجانية يومياً، الاحتمالات والنتيجة على الخادم فقط | منفّذ |
| المتجر، المهارات، Time Bomb | **لم تُنفَّذ بعد** (Phase 4) |
| الإنجازات، الإحصائيات، الإعدادات | **لم تُنفَّذ بعد** (المراحل 6–7) |

## ما الذي تم التحقق منه فعلاً

- **اختبارات الـ Frontend (16):** نجحت.
- **اختبارات الـ Backend (98):** نجحت في بيئة التطوير التي بنيت فيها، على قاعدة SQLite حقيقية (مع ترحيل حقيقي للجداول) لكن عبر محوّل مختلف عن الذي ستستخدمه أنت.
- **تجربة متصفح كاملة** على عرض الموبايل (تسجيل ← مهمة ← جلسة تركيز ← نقاط ← خروج، ثم استلام المكافأة اليومية ← لفّ العجلة ← منع اللفّة الثانية): نجحت.
- **ما لم يُجرَّب بعد:** تشغيل `npm test --prefix server` على جهازك مع محرك Prisma الأصلي. هذا أول اختبار حقيقي له، فإن فشل شيء أرسل لي النص كاملاً.

## الترقية من Phase 2

لا تحتاج لإعادة إنشاء قاعدة البيانات (لم يتغيّر `schema.prisma`). لإضافة إعدادات اللعبة وشرائح العجلة:

```powershell
npm install --prefix client
npm install --prefix server
npm run db:seed --prefix server
```

الإعدادات المفقودة لها قيم افتراضية في الكود، فلن ينكسر شيء لو لم تُشغّل الـ seed.

## قواعد اللعبة (Phase 2)

تُحفظ في جدول `AppSettings` ويمكن تغييرها بدون تعديل الكود:

| المفتاح | القيمة الحالية |
|---|---|
| `focus.intervalSeconds` | 120 (فترة = دقيقتان) |
| `focus.pointsPerInterval` | 10 نقاط |
| `focus.xpPerInterval` | 5 XP |
| `focus.maxSessionMinutes` | 120 |
| `focus.durationPresets` | 25 / 50 / 90 دقيقة |
| `level.baseXp` / `level.stepXp` | 100 / 50 (المستوى L يحتاج 100 + 50×(L−1) XP) |
| `daily.goalIntervals` | 15 فترة يومياً |
| `daily.loginPoints` / `daily.loginXp` | 20 نقطة / 10 XP في اليوم الأول من السلسلة |
| `daily.streakBonusPoints` | +5 نقاط لكل يوم متتالٍ إضافي |
| `daily.streakBonusCapDays` | 7 (تتوقف الزيادة عند اليوم السابع: 50 نقطة) |

**عجلة الحظ:** 8 شرائح في جدول `LuckyWheelReward` (5، 10، 20، 30، 50، 100 نقطة، و10 / 25 XP) ولكل شريحة `weight` يحدد احتمالها (متوسط الجائزة حوالي 15 نقطة للّفة). الأوزان لا تُرسَل للمتصفح أبداً. اليوم يتغيّر عند منتصف الليل بتوقيت القاهرة، ولا تُحتسب الأيام الفائتة.

قاعدة المؤقت والمكافآت: وقت الإيقاف المؤقت لا يُحتسب، ولا تُمنح مكافأة إلا لفترة اكتملت كاملة. الواجهة لا تستطيع تحديد النقاط أبداً؛ لا يوجد مسار يضيف نقاطاً، والخادم يحسب الوقت بساعته هو.

## التثبيت على Windows

1. ثبّت **Node.js LTS** (الإصدار 20 أو أحدث) من nodejs.org، ثم تحقق في Windows Terminal:
   ```powershell
   node -v
   npm -v
   ```
2. افتح مجلد المشروع في VS Code، ثم من الـ Terminal داخل المجلد الرئيسي `timer`:
   ```powershell
   npm install
   npm run setup
   ```
   الأمر `setup` يثبّت حزم `server` و`client`، وينشئ `server/.env` بمفتاح سري عشوائي، ثم ينشئ قاعدة البيانات ويضيف حساباً تجريبياً.
3. تشغيل المشروع:
   ```powershell
   npm run dev
   ```
   - الواجهة: http://localhost:5173
   - الـ API: http://localhost:4000/api/health

حساب تجريبي (للتطوير فقط): `demo@timer.local` / `Demo12345`

### لو ظهر خطأ

- أرسل لي **نص الخطأ كاملاً** من الـ Terminal. أكثر خطأ متوقع هو في `npm run setup` عند أول تشغيل لـ Prisma.
- لو ظهر `EADDRINUSE`: المنفذ 4000 أو 5173 مستخدم في برنامج آخر، أغلقه أو غيّر `PORT` في `server/.env`.
- لو أردت إعادة قاعدة البيانات من الصفر: `npm run db:reset --prefix server`.

## الاختبارات

```powershell
npm test                          # الاثنان معاً
npm test --prefix server          # الـ Backend فقط
npm test --prefix client          # الـ Frontend فقط
npm run build                     # بناء الواجهة للتأكد من سلامة TypeScript
```

## البنية

```
timer/
├─ client/                 React + Vite + TypeScript + Tailwind (RTL)
│  └─ src/ api/ components/ context/ i18n/ pages/ test/
└─ server/                 Express + TypeScript + Prisma (SQLite)
   ├─ prisma/              schema.prisma, seed.ts
   └─ src/
      ├─ routes/           يربط المسار بالـ controller
      ├─ controllers/      يقرأ الطلب ويرجع الرد فقط
      ├─ services/         منطق العمل (التحقق، التشفير، JWT)
      ├─ repositories/     الجزء الوحيد الذي يعرف قاعدة البيانات
      ├─ middleware/       حماية المسارات ومعالجة الأخطاء
      ├─ validation/       قواعد التحقق (zod)
      └─ tests/
```

**القاعدة الذهبية:** الواجهة تطلب، والـ Backend يقرر. لا يوجد منطق نقاط أو مكافآت داخل React.

## الأمان في هذه المرحلة

- كلمات المرور تُحفظ بـ **bcrypt** ولا تُرجَع في أي رد.
- الجلسة في **JWT داخل cookie من نوع httpOnly** (لا تقرؤها JavaScript)، مع `SameSite=Lax`.
- رسالة موحّدة لخطأ الدخول (لا تكشف هل البريد موجود)، ومقارنة كلمة مرور وهمية لتساوي زمن الاستجابة.
- **Rate limit** على الدخول والتسجيل، و`helmet`، وCORS مقيَّد بعنوان الواجهة، وحد لحجم الطلب.
- التحقق من كل مدخل في الـ Backend بـ zod (التحقق في الواجهة للتجربة فقط).
- في الإنتاج: ضع `COOKIE_SECURE=true` وشغّل الموقع على HTTPS.

## متغيرات البيئة (`server/.env`)

| المتغير | المعنى |
|---|---|
| `DATABASE_URL` | مسار قاعدة SQLite، الافتراضي `file:./dev.db` |
| `JWT_SECRET` | مفتاح توقيع الجلسات (32 حرفاً على الأقل، ينشئه `npm run setup`) |
| `JWT_EXPIRES_DAYS` | مدة الجلسة بالأيام |
| `PORT` | منفذ الـ API |
| `CLIENT_ORIGIN` | عنوان الواجهة المسموح له بالاتصال |
| `COOKIE_SECURE` | `true` عند استخدام HTTPS |
| `BCRYPT_ROUNDS` | قوة التشفير (12 مناسب) |
| `DEFAULT_TIMEZONE` | المنطقة الزمنية الافتراضية للمستخدمين الجدد: `Africa/Cairo` |

## الـ API

| الطريقة | المسار | الوصف |
|---|---|---|
| POST | `/api/auth/register` | `{email, username, displayName, password}` |
| POST | `/api/auth/login` | `{email, password}` |
| POST | `/api/auth/logout` | يمسح الـ cookie |
| GET | `/api/auth/me` | المستخدم الحالي (يحتاج تسجيل دخول) |
| GET | `/api/health` | فحص تشغيل الخادم |
| GET/POST | `/api/tasks` | قائمة المهام / إنشاء مهمة |
| PATCH/DELETE | `/api/tasks/:id` | تعديل / حذف |
| POST | `/api/tasks/:id/complete` ، `/reopen` | إكمال / إعادة فتح |
| GET | `/api/focus/active` | الجلسة الحالية (تُحسب المكافآت المستحقة عند الطلب) |
| POST | `/api/focus/start` ، `/pause` ، `/resume` ، `/finish` ، `/reset` | التحكم بالجلسة |
| GET | `/api/progress` | النقاط، XP، المستوى، هدف اليوم |
| GET | `/api/points/transactions?limit=` | سجل النقاط |
| GET | `/api/config` | قواعد اللعبة للعرض فقط |
| GET | `/api/daily` | حالة اليوم والسلسلة والمسار |
| POST | `/api/daily/claim` | استلام مكافأة اليوم (بدون أي مدخلات) |
| GET | `/api/wheel` | شرائح العجلة (بدون الأوزان) |
| POST | `/api/wheel/spin` | لفّة اليوم؛ الخادم يختار الجائزة |

الأخطاء بصيغة: `{ "error": { "code": "...", "message": "...", "details": [...] } }`

## قاعدة البيانات

الجداول (كلها في `server/prisma/schema.prisma` مع التعليقات): `User`, `UserProfile`, `UserProgress`, `Task`, `FocusSession`, `FocusInterval`, `PointsTransaction`, `Skill`, `UserInventory`, `SkillActivation`, `DailyLogin`, `LuckyWheelReward`, `LuckyWheelSpin`, `Achievement`, `UserAchievement`, `AppSettings`.

أهم القيود التي تمنع الغش:
- `FocusInterval @@unique([sessionId, index])` → مكافأة الفترة تُمنح مرة واحدة فقط.
- `PointsTransaction.refKey @unique` → مفتاح تفرّد لكل حركة نقاط.
- `DailyLogin @@unique([userId, date])` → تسجيل دخول واحد يومياً.
- `LuckyWheelSpin.dailyLoginId @unique` → لفة واحدة لكل تسجيل دخول.
- `SkillActivation.inventoryId @unique` + `UserInventory.consumedAt` → المهارة تُستخدم مرة واحدة.
