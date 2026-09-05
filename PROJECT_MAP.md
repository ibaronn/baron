# PROJECT_MAP.md — لعبة الجغرافيا التفاعلية (GeoQuiz Globe)

## [PROJECT_SCALE]
single-file / Simple PWA — بدون build tooling، Single-page Web App داخل `www/`.
يُعامل تحت بروتوكولات Single-file: لا طبقات DDD/Shared/Core، وتنظيم الكود بأقسام واضحة داخل ملف واحد، والاختبار بمحاكاة يدوية موصوفة بدل TDD كامل.

## [TECH_STACK]
- **Frontend**: HTML/CSS/JS (Vanilla) — داخل `www/`
- **3D Globe**: globe.gl **2.46.2** (معبّأة محلياً `assets/lib/globe.gl.min.js`) — تشمل Three.js داخلياً؛ **ممنوع** تحميل Three.js منفصلاً (تعارض مؤكد في النموذج الأولي)
- **حدود الدول**: Natural Earth **110m** (GeoJSON) — محلي (`countries-110m.geojson`)
- **الأعلام**: SVG محلي من flag-icons **7.5.0** (171 علماً مطابقاً للـ ISO_A2) — `assets/flags/`
- **بيانات الدول**: مجانية داخل الـ GeoJSON — `name_ar`، `name_en`، `continent` (كفى لنطاق MVP بدون أي `meta.json`)
- **مؤجل للـ Hints المتقدمة**: المساحة التقريبية + الجوار (`meta.json` غير موجود بعد)
- **حفظ محلي**: localStorage (best score + دول مكتشفة)
- **Native**: Capacitor **8.5.1** (`@capacitor/core`, `@capacitor/cli`, `@capacitor/ios`) — بدون كود Swift مخصص
- **التوقيع**: خارج النطاق تماماً (يدوي، بدون أتمتة — ممنوع fastlane/App Store Connect)

## [SYSTEM_FLOW]
1. **Load** → لوحة تشخيص أخطاء مدمجة + globe.gl يرسم الكرة بحدود الدول (أصول محلية 100%، Offline-ready)
2. **Pick** → اختيار دولة عشوائية غير مكررة → Floating Card يعرض العلم
3. **Answer** → لمس الدولة:
   - صحيحة → احتفال بصري + اهتزاز خفيف + نقاط
   - خاطئة → تقريب الكاميرا تلقائياً نحو المنطقة الصحيحة
4. **Hint** → "تلميح القارة" (خصم نقاط، لا حظر) — تلميح واحد فقط في MVP
5. **Score** → نقاط الجولة (سرعة الأساس − خصم التلميحات) → تحديث localStorage
6. **Loop** → جولة = 10 دول → ملخص ← دولة جديدة

## [ARCHITECTURE]
```
GeoQuiz/
├── package.json               (Capacitor 8.5.1)
├── package-lock.json          (مولّد — الـ CI يستخدم npm ci)
├── capacitor.config.json      (appId, webDir: www)
├── .github/workflows/build-ios.yml  (macOS runner — ينفّذ npm و xcodebuild في CI فقط)
├── docs/SIGNING_GUIDE.md      (مسار الإنتاج + التوقيع اليدوي)
├── PROJECT_MAP.md
└── www/
    ├── index.html             (بنية + كود بنفس الملف بأقسام واضحة)
    └── assets/
        ├── lib/globe.gl.min.js    (globe.gl 2.46.2 معبّأة)
        ├── data/countries-110m.geojson
        └── flags/*.svg            (171 علماً)
```

ملاحظة بيئة (v): لا Node/npm على جهاز المطوّر — الأصول عُبّئت بتنزيل مباشر (HTTPS)، وكل `npm install`/`cap` يعمل داخل GitHub Actions حصراً.

تقسيم أقسام `index.html` (فواصل منطقية فقط، لا وحدات):
1. `CONFIG & DIAG` — ثوابت الأداء + لوحة التشخيص المدمجة
2. `ASSETS LOADER` — جلب geojson + meta + الربط مع الأعلام
3. `GLOBE RENDER` — globe.gl، polygons، atmosphere، PixelRatio≤2، Texture 2048×1024
4. `GAME STATE` — اختيار عشوائي، جولة (10)، احتساب نقاط
5. `INPUT` — لمس الدولة، تقريب الكاميرا عند الخطأ
6. `UI RENDER` — Floating Card، التغذية الراجعة، الوضع الليلي/النهاري
7. `HINTS` — تلميح القارة وخصم النقاط
8. `PERSISTENCE` — localStorage

## [MILESTONES]
| # | المحتوى | معيار النجاح |
|---|---------|-------------|
| **M0** ✅ | Scaffold + أصول محلية + CI | `index.html` بلا أخطاء تشخيص، أصول globe.gl/GeoJSON/أعلام متاحة محلياً، Workflow إنتاج `.ipa` جاهز |
| **M1** ✅ | الكرة الأرضية التفاعلية | دوران/تكبير سلس باللمس، حدود دول واضحة، وضع ليلي/نهاري، لا تقطيع |
| **M2** ✅ | حلقة اللعب الأساسية | جولة كاملة تعمل: علم → نقر → تغذية راجعة (صحيح/خطأ) |
| **M3** ✅ | التلميح والنقاط والحفظ | تلميح قارة يخصم نقاطاً، الحفظ يبقى بعد إعادة فتح الصفحة |
| **M4** ⏳ | النشر النهائي | `App-unsigned.ipa` يُنتج كـ Artifact (محقق بناءً على CI) — تأكيد حقيقي يتطلب أول push |

### حالات الملاحظة (log)
- **2026-09-05 — M0 مكتمل**: 171 دولة قابلة للعب، 171 علماً، globe.gl 2.46.2 + GeoJSON 110m محليان؛ boot-check يتأكد من سلامة الأصول ويعرض في لوحة التشخيص. الـ Workflow يبني `App-unsigned.ipa` (Signing=NO) — التحقق منه يتطلب دفعاً فعلياً للـ GitHub.
- **2026-09-05 — M1 مكتمل**: عرض الكرة (globe.gl) بحدود الدول مع hover (رفع + لون + tooltip ع/إ)، دوران ذاتي يتوقف عند أول لمس، موضوع ليلي/نهاري يتبع النظام ويتبدل حياً، إعادة التحجيم عند التدوير. تبين أن المكتبة تفرض `Math.min(2, devicePixelRatio)` داخلياً، وخامة النهار أُعيد تحجيمها من 4096→2048 محلياً. فحص `node --check` للسكربت: ناجح (exit=0).
- **2026-09-05 — M2 مكتمل**: حلقة لعب من 10 دول — بطاقة علم عائمة، نقاط قائمة على السرعة (base 100 − 10/ث، أدنى 10)، خطأ: تقريب بالكاميرا نحو الدولة الصحيحة + اهتزاز، صحيح: احتفال (قصاصات + فلاش أخضر + اهتزاز Haptics/نبذ)، ملخص نهاية الجولة بزر "جولة جديدة". أُضيف `@capacitor/haptics 8.0.2` (يتفعل نطقياً عند التثبيت، وبحث navigate نبذ في المتصفح). وُلد `package-lock.json` (99 حزمة) وهُيئ الـ CI إلى `npm ci` + تحصين خطوة التغليف بـ `find` قبل `cp` (درس Xcode 26). فحص صياغة ناجح.
- **2026-09-05 — M3 مكتمل**: تلميح القارة (تفعيل يدوي فقط، خصم 25/سؤال) مع خريطة أسماء قارات عربية، احتساب النقاط = سرعة − خصم تلميحات (أدنى 10)، حفظ localStorage: `gq_best_score` (أفضل نتيجة) + `gq_discovered` (دول مكتشفة) مع حماية ضد عدم توفر التخزين. أُضيف شريحة "الأفضل" في HUD وسطر الإحصاء في الملخص. **فحص آلي لا-متصفحي (16/16 PASS)**: إنشاء الجولة بلا تكرار، تدفق التلميح، نقاط إجابة صحيحة، نمو وتخزين المكتشفة، تقدم السؤال وإعادة تفعيل الزر، استرجاع أفضل نتيجة.
- **2026-09-05 — M4 (محلياً)**: التحقق الكامل من سلسلة Capacitor على ويندوز عبر Node محمول — `cap add ios` نجح بقراءة `capacitor.config.json` (بلا TypeScript)، و`cap sync ios` نسخ `www/` إلى `ios/App/App/public/`، وSPM يمثل مصدر الحقيقة (لا CocoaPods) مع اكتشاف `@capacitor/haptics`. تحقق YAML للـ Workflow (8 خطوات، دمج `find` قبل `cp`) — سليم. أُنشئ `docs/SIGNING_GUIDE.md` (إنتاج Artifact + توقيع يدوي خارج النطاق). المتبقي الحقيقي: أول push إلى GitHub لرؤية `App-unsigned.ipa` فعلياً.
- **2026-09-05 — ضبط جودة**: استبعاد أنتاركتيكا (AQ) من أسئلة اللعبة (تظل مرسومة على الكرة) — قاع الأسئلة 170 دولة. كل الفحوص الآلية ما زالت 16/16.

## [ORPHANS & PENDING]
- `meta.json` (المساحة التقريبية + الجوار) — مؤجل لمرحلة التلميحات المتقدمة (خارج MVP)
- تأكيد إنتاج `App-unsigned.ipa` من الـ Workflow — يتطلب أول push إلى GitHub
- دروس بيئة سابقة للـ CI مستوعَبة (سجل: capacitor.config.json بدل TS، `-project` بدل `-workspace` في Capacitor 8/SPM، `build` بدل `archive`، `find` بمسار ثابت قبل cp)