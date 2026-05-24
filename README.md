# 📱 سيرة — تطبيق بناء السيرة الذاتية

> تطبيق Android احترافي لبناء السيرة الذاتية، مخصص للسوق السوداني.  
> **30 قالب** | **بدون إنترنت** | **تصدير PDF & Word** | **شهادات** | **حقول سودانية**

---

## 🗂️ هيكل المشروع

```
sira-app/
├── www/
│   └── index.html          ← تطبيق الويب الكامل (1100+ سطر)
├── .github/
│   └── workflows/
│       └── build.yml       ← GitHub Actions (APK + AAB تلقائي)
├── package.json            ← تبعيات Capacitor
├── capacitor.config.json   ← إعدادات التطبيق
├── .gitignore
└── README.md
```

---

## 🚀 الإعداد المحلي (من Termux)

### 1. تثبيت المتطلبات
```bash
# في Termux
pkg install nodejs git
npm install -g @capacitor/cli
```

### 2. استنساخ المشروع
```bash
git clone https://github.com/USERNAME/sira-app.git
cd sira-app
npm install
```

### 3. إضافة منصة Android
```bash
npx cap add android
npx cap sync android
```

### 4. فتح في Android Studio (اختياري)
```bash
npx cap open android
```

---

## 🏗️ GitHub Actions — البناء التلقائي

### كيف يعمل؟

| حدث | ما يحدث |
|-----|---------|
| `push` إلى `main` | بناء Debug APK تلقائياً |
| `push` إلى `develop` | بناء Debug APK |
| إنشاء Tag `v*` | بناء Debug APK + Release AAB + APK وإنشاء GitHub Release |
| تشغيل يدوي | اختيار نوع البناء |

### تحميل الـ APK
بعد اكتمال كل بناء:
1. افتح **Actions** في GitHub
2. اختر آخر workflow run
3. ارحل للـ **Artifacts** في الأسفل
4. حمّل `سيرة-debug-XX`

---

## 🔑 إعداد التوقيع للإصدار الرسمي

### الخطوة 1 — إنشاء Keystore
```bash
# في الكمبيوتر أو Termux (يحتاج Java)
keytool -genkeypair \
  -v \
  -keystore sira-release.keystore \
  -alias sira \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -dname "CN=Sira App, OU=Dev, O=Sira, L=Khartoum, S=Khartoum, C=SD"
```

سيطلب منك:
- `Enter keystore password:` — اختر كلمة مرور قوية
- `Enter key password:` — يمكن نفس الكلمة

### الخطوة 2 — تحويل Keystore إلى Base64
```bash
# Linux / Mac / Termux
base64 -w 0 sira-release.keystore > keystore.b64
cat keystore.b64
# انسخ المحتوى كاملاً
```

### الخطوة 3 — إضافة GitHub Secrets

اذهب إلى: `GitHub Repo → Settings → Secrets and variables → Actions`

أضف هذه الـ Secrets:

| Secret Name | القيمة |
|-------------|--------|
| `KEYSTORE_BASE64` | محتوى ملف `keystore.b64` |
| `STORE_PASSWORD` | كلمة مرور الـ keystore |
| `KEY_ALIAS` | `sira` (أو الاسم الذي اخترته) |
| `KEY_PASSWORD` | كلمة مرور المفتاح |

### الخطوة 4 — بناء إصدار رسمي
```bash
# إنشاء tag جديد يُطلق البناء الرسمي تلقائياً
git tag v1.0.0
git push origin v1.0.0
```

سيجد GitHub Actions الـ Secrets ويبني APK + AAB موقّعَين.

---

## 📦 رفع على Google Play

1. ابنِ الـ AAB عبر tag push (انظر أعلاه)
2. حمّل `app-release.aab` من GitHub Artifacts
3. في [Google Play Console](https://play.google.com/console):
   - إنشاء تطبيق جديد
   - App ID: `sd.sira.app`
   - ارفع الـ AAB في Production أو Internal Testing

---

## ⚙️ تخصيص إعدادات التطبيق

### تغيير App ID
في `capacitor.config.json`:
```json
{
  "appId": "sd.sira.app",
  "appName": "سيرة"
}
```

ثم بعد التغيير:
```bash
npx cap sync android
```

### تغيير اسم التطبيق والأيقونة
```bash
# بعد إضافة android
# الأيقونة: android/app/src/main/res/mipmap-*/ic_launcher.png
# الاسم: android/app/src/main/res/values/strings.xml
```

### تغيير رقم الإصدار
في `android/app/build.gradle`:
```groovy
versionCode 1       // رقم داخلي (يزيد مع كل إصدار)
versionName "1.0.0" // يظهر للمستخدم
```

---

## 🧪 اختبار التطبيق

### Debug APK (بدون توقيع)
```bash
# بناء محلي
cd android
./gradlew assembleDebug

# الملف في:
# android/app/build/outputs/apk/debug/app-debug.apk

# تثبيت مباشر على هاتف متصل
adb install android/app/build/outputs/apk/debug/app-debug.apk
```

### Live Reload (للتطوير)
```bash
npx cap run android --livereload --external
```

---

## 📋 GitHub Secrets المطلوبة

| Secret | مطلوب؟ | الوصف |
|--------|---------|-------|
| `KEYSTORE_BASE64` | للإصدار الرسمي | Keystore مشفر بـ Base64 |
| `STORE_PASSWORD` | للإصدار الرسمي | كلمة مرور الـ keystore |
| `KEY_ALIAS` | للإصدار الرسمي | اسم المفتاح |
| `KEY_PASSWORD` | للإصدار الرسمي | كلمة مرور المفتاح |
| `GITHUB_TOKEN` | تلقائي | GitHub يضيفه تلقائياً |

> ⚠️ **تنبيه:** لا تضع الـ keystore أو كلمات المرور في ملفات المشروع أبداً.

---

## 🔧 استكشاف الأخطاء

### مشكلة: `Could not find com.android.tools.build:gradle`
```bash
# في android/build.gradle، تأكد من classpath:
classpath 'com.android.tools.build:gradle:8.0.0'
```

### مشكلة: `SDK location not found`
```bash
# أنشئ ملف local.properties في android/
echo "sdk.dir=$ANDROID_HOME" > android/local.properties
```

### مشكلة: بطء البناء في GitHub Actions
يمكن تقليل وقت البناء بتفعيل Gradle cache (موجود بالفعل في الـ workflow).

---

## 🌟 ميزات التطبيق

- ✅ 30 قالب احترافي (5 مجانية + 25 بمشاهدة إعلان)
- ✅ حقول الاستمارة السودانية (رقم وطني، ديانة، جنسية...)
- ✅ صورة شخصية في السيرة
- ✅ نظام الشهادات (خبرة / تقدير / دورات / حضور)
- ✅ تصدير PDF + Word
- ✅ صفحة المستندات مع بحث وفلاتر
- ✅ يعمل بدون إنترنت — localStorage فقط
- ✅ دعم Android Back Button

---

## 📄 الرخصة

© 2024 سيرة. جميع الحقوق محفوظة.

---

*بُني بـ Capacitor 5 + Vanilla HTML/JS/CSS — بدون frameworks*
