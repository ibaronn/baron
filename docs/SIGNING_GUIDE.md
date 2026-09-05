# دليل الإنتاج والتوقيع اليدوي — GeoQuiz Globe

## 1) إنتاج بملف `App-unsigned.ipa` (بدون Mac محلي)
لا يوجد أي بناء محلي — كل شيء يحدث في GitHub Actions على macOS:

1. ادفع الكود إلى المستودع (`git push`).
2. يشتغل الـ Workflow `build-ios-unsigned` تلقائياً (أو فعّله يدوياً من تبويب Actions → **Run workflow**).
3. بعد نجاح الخطوات (npm ci → cap add ios → cap sync ios → xcodebuild بلا توقيع → التغليف) يظهر Artifact باسم `App-unsigned.ipa` في أعلى صفحة الـ Action.
4. نزّله.

## 2) ما بداخله
حزمة `ipa` = أرشيف يحوي `Payload/App.app` مبني بـ `CODE_SIGNING_ALLOWED=NO`. غير موقَّع — لهذا لا يُثبَّت مباشرة.

## 3) التوقيع والتثبيت اليدوي (خارج نطاق هذا المشروع — بيدك فقط)
لا توجد أي أتمتة توقيع (ممنوع: fastlane match، App Store Connect API، provisioning آلي). التوقيع يتم يدوياً بشهادتك غير الرسمية عبر أحد المسارات الاعتيادية:

- **Sideloadly** (Windows) — حدد الـ ipa، اختر Apple ID/شهادتك، ثم Install عبر iTunes/الثقة المسبقة.
- **AltStore / SideStore** — افتح الـ ipa داخل التطبيق وثبّته بمعرفك.
- بديل على ماك موجود: `codesign --force --sign <شهادتك> --deep Payload/App.app` ثم أعد التغليف إلى Payload/App-unsigned.ipa.

بعد التثبيت أول مرة: **الإعدادات → عام → إدارة الأجهزة → الموافقة** على ملف التعريف الخاص بك.

## ملاحظة
- التثبيت بملف تعريف مجاني يُعاد توقيعه من الجهة الموقّعة عندما تنتهي مدته.
- عند تعديل `www/` فقط أعد الدفع — الـ CI يعيد البناء كاملاً؛ لا حاجة لأي خطوة يدوية.