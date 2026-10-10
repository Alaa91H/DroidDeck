# DroidDeck — خطة التعريب العربي الشامل وRTL

**الحالة:** خطة تنفيذ فقط، ولم يُعدّل كود التطبيق  
**مصدر الفحص:** `main` عند `c875ceda4d100c96e0466e2ab0826856a9da2dd7` بتاريخ 2026-10-10  
**الهدف:** عربية فصحى حديثة مفهومة، تغطية كل نصوص DroidDeck القابلة للترجمة، وتجربة RTL محترفة مع الحفاظ على Steam والجلسات وأزرار التحكم.

> حدود التحقق: تمت مراجعة بنية المستودع وملفات الموارد والتنقل واللغة وعينات واسعة من واجهات Kotlin/Compose واختبارات Steam/Linux. لم يتم بعد تشغيل APK أو تدقيق بصري شامل على جهاز حقيقي. تظل نتائج التنفيذ والـ QA لاحقة، ولا يجوز الادعاء بأنها منجزة مسبقًا.

## 1. نتائج الفحص الموثقة

| المجال | الوضع الحالي | الإجراء |
| --- | --- | --- |
| مصدر النصوص | `app/src/main/res/values/strings.xml` يضم **1880** `string` بينها **12** `translatable=false`، و**21** `plurals`، و**2** `string-array` | 1868 نصًا عاديًا قابلاً للتعريب في خط الأساس، إضافة إلى الجمع والقوائم والنصوص المكتشفة خارج XML |
| اللغات | مجلدات es/fr/ru/ja/ko/zh-CN/zh-TW/zh-HK، بدون `values-ar` | إنشاء `app/src/main/res/values-ar/strings.xml` |
| تكوين اللغة | `res/xml/locales_config.xml` و`core/AppLanguage.kt` لا يعرفان `ar` | تسجيل اللغة في القائمتين والاسم الأصلي «العربية» ودعم locales إقليمية |
| تبديل اللغة | `ui/FrontEndSetup.kt` يحتوي قائمة اللغات؛ `MainActivity.kt` يستخدم `setChosen` ثم `recreate()` | اختبار تبديل سلس يحفظ الصفحة والحالة والجلسات الجارية |
| RTL | `AndroidManifest.xml` يحوي `android:localeConfig` دون `android:supportsRtl="true"` | تفعيل مشروط بالتحقق من الواجهات وأسـطح اللعب |
| القوائم المنبثقة | `ui/SettingsWidgets.kt` يستخدم `anchorBounds.left/right` دون `layoutDirection` في مزودي المواقع | مواضع RTL صحيحة وبقاء القائمة على الشاشة |
| الشريط الجانبي | `ui/FrontEndRail.kt` مصمم كتخطيط جانبي يساري | فحص الاتجاه والمنطق الملاحي دون تحطيم ترتيب D-pad |
| محتوى خارج موارد Android | `input/SteamTouchControls.kt` يرسم `Trackpad` نصًا إنجليزيًا على Canvas | استخراج النص إلى resource وفحص التشكيل |
| Steam | `session/SteamLanguage.kt` يملك مسبقًا `ar -> arabic` | اختبارات تكامل بدلاً من إعادة كتابة التحويل |
| Linux fonts | `tools/linuxfs/overlay/usr/local/bin/droiddeck-session` يقدم fallback محددًا لبعض لغات آسيا لا للعربية صراحةً | اختبار glyph/fontconfig عربي وإصلاح إن لزم |
| الاختبارات | `SteamLanguageTest.kt` و`tools/tests/test_steam_language.py` و`test_droiddeck_fonts.py` موجودة | توسيعها لتغطية العربية وبيئات RTL |

**تنبيه:** تُستثنى أسماء الأزرار القياسية A/B/X/Y وأسماء الملفات والأوامر والعلامات التجارية من «الترجمة الحرفية» متى كان تعريبها يغيّر المعنى أو يضر التشغيل؛ توثَّق الاستثناءات. ملفات اللغات الأخرى ناقصة بعض مفاتيح النسخة الإنجليزية؛ لا تغيّرها مهمة التعريب العربية عشوائيًا.

## 2. مسار التنفيذ المرحلي — ترتيب إلزامي

### AR-00: تدقيق وفهرسة شاملة
- [ ] جرد كل موارد `strings/plurals/arrays` من XML بتمييز عناصر `translatable=false`.
- [ ] مسح كامل للملفات `*.kt/*.java/*.xml` و`Canvas` و`assets` والسكربتات والأوامر التي تعرض نصًا فعليًا للمستخدم، مع فحص Toaster/Notifications/ContentDescription/Dialogs/Errors/FileManager/WebView.
- [ ] تقرير CSV/Markdown آلي يحوي: الملف والسطر، مفتاح المورد، النص الحالي، حالة الترجمة، الـ placeholders، المراجع داخل الشيفرة، والاستثناءات المعللة.
- [ ] فصل السلاسل القابلة للعرض عن السجلات التشخيصية، مفاتيح JSON والبروتوكول، رموز أوامر Linux وSteam، وأسماء الملفات والألعاب.
- [ ] تثبيت baseline SHA وتوفير إحصاءات تغطية تستحدث تلقائيًا في كل PR.
**بوابة AR-00:** لا تبقى واجهة ضمن DroidDeck خارج نطاق الجرد أو لها استثناء موثق.

### AR-01: بنية لغة عربية صحيحة
- [ ] تسجيل `ar` داخل `AppLanguage.supported` و`supportedTag` و`nativeName`.
- [ ] إضافة `<locale android:name="ar" />` في `res/xml/locales_config.xml`.
- [ ] إعداد `values-ar/strings.xml` وتفعيل `android:supportsRtl="true"` مع مراجعة تفاعلات كل Activity/Service/Compose.
- [ ] اختبار `System default` واختيار `ar` و`ar-EG` و`ar-SA` وتغيير لغة النظام وAndroid 13+ per-app language وإعادة الفتح.
- [ ] عدم إعادة تشغيل جلسة Steam أو تنزيل أو تطبيق آخر عند تغيير اللغة؛ التحقق من الاستعادة بعد `MainActivity.recreate()`.
- [ ] اختبار عودة LTR بعد اختيار English وفشل الاسترجاع الآمن لخيارات غير معروفة.
**بوابة AR-01:** العربية تظهر وتثبت وتستعيد الحالة، دون انحدار للجلسات أو اللغات السابقة.

### AR-02: ترجمة محتوى كاملة ودقيقة
- [ ] تعريب **1868 string قابلة للترجمة** من خط الأساس مع أي إضافات حديثة بعد إعادة الجرد.
- [ ] تعريب **21 plurals** بكافة فئات CLDR العربية `zero/one/two/few/many/other` وبالأخص 0/1/2/3/5/11/21/100.
- [ ] تعريب عنصري `string-array` بكافة `item` مع الحفاظ على الترتيب.
- [ ] Glossary موحَّد: Launcher/Session/Library/Downloads/Runtime/Component/Compatibility Layer/Proton/FEX/Drivers/Controls/Overlay/Games/Stores.
- [ ] ترجمة لغوية بشرية وفق سياق الواجهة والاختصار في الأزرار، دون ترجمة حرفية ميكانيكية أو تغيير أسماء المنتجات المعروفة.
- [ ] حفظ placeholders مرتبطة بالمواقع `%1$s`, `%2$d` وأنواعها، escapes `\\'`، الأسطر، HTML، الروابط، المحددات، الوحدات/الاختصارات التقنية.
- [ ] تدقيق لغوي ثانٍ + مراجعة كل نص ضمن شاشة حقيقية.
**بوابة AR-02:** تغطية 100% للموارد العربية القابلة للترجمة وصحة صيغ الجمع والتنسيق مع صفر resource format mismatch.

### AR-03: النصوص الثابتة خارج XML
- [ ] رصد واستخراج `Text("...")`, `Paint.drawText("...")`, `Toast`, `Notification`, `contentDescription`, `Dialog` والنصوص المحسوبة.
- [ ] مثال مؤكد: `SteamTouchControls.kt` يرسم كلمة `Trackpad` مباشرة.
- [ ] فحص النصوص المركبة وحالات رقم/نسبة/تاريخ/حجم ملف في التطبيق وإعادة تنسيق ما يواجه المستخدم فقط.
- [ ] عدم تغيير أسماء متغيرات البيئة أو الأكواد أو الإشارات الأساسية لخدمات Steam/Linux/Proton أو نصوص السجلات الخام.
**بوابة AR-03:** صفر سلاسل إنجليزية واجهية صلبة غير موثقة في كود DroidDeck.

### AR-04: RTL شامل مع عزل وظائف الألعاب
- [ ] مراجعة Launcher/SideRail/Setup/Downloads/Stores/Library/FileManager/Notifications/SessionOverlay/ControllerEditor/TouchControls/Dialogs/Popups والصفحات الضيقة.
- [ ] مواءمة `SettingsWidgets.BelowStartProvider/BelowEndProvider` مع `layoutDirection`، وتجنّب تجاوز الشاشة.
- [ ] استبدال اتجاهات الواجهة البصرية غير اللازمة `left/right` بـ `start/end`، وتدقيق ترتيب الخطوات والتبويبات والـ focus والـ D-pad والتلميحات والأيقونات الاتجاهية.
- [ ] دعم LTR spans وعزل bidi في المسارات وURL وأسماء الألعاب الإنجليزية والعربية والاختصارات، دون تعديل النص الأصلي أو عمليات ملفات النظام.
- [ ] فحص قص النصوص/ellipsis/الرسومات وملاءمة 16:9 و4:3 ومربع وApp Scale 75–150% وFont Scale النظام.
- [ ] **لا يُعكس** Steam/Proton/Surface/PiP، مخطط أزرار Xbox/Steam، المحاور واللمس وإحداثيات اللمس، مؤشرات الأداء والمواقع الفيزيائية. اعزل `LayoutDirection.Ltr` محليًا فقط حيث يلزم، ولا تفرضه على التطبيق كله.
**بوابة AR-04:** صفر عناصر واجهة معكوسة بشكل خاطئ، وصفر خلل في اللمس أو وحدة التحكم، واللغات LTR لا تتراجع.

### AR-05: Steam وLinux والخطوط
- [ ] اختبار تمرير `arabic` من `SteamLanguage.forLocale` إلى سكربتات تشغيل Steam عند أول تشغيل وعند تغيير اللغة وبعد إعادة تشغيل الجلسة.
- [ ] فحص `droiddeck-steam-language` و`test_steam_language.py` على قيمة `arabic` وsettings/registry دون حذف قيم أخرى.
- [ ] فحص Arabic glyphs والاتصال والتشكيل وعلامات الترقيم والاتجاه في Android وLinux desktop وتكامل `droiddeck-fonts` وfontconfig. إضافة قاعدة fallback عربية أو خطوط عند إثبات الحاجة.
- [ ] توثيق أن واجهات Steam والألعاب ومتاجر الأطراف الثالثة قد تتبع دعمها الخاص؛ لا تُوعد بترجمتها أو تغييرها قسرًا.
**بوابة AR-05:** يعمل اختيار العربية عبر الطبقات التي يضبطها DroidDeck ولا يكسر عملية التشغيل.

### AR-06: اختبارات آلية + screenshots + الجهاز
- [ ] توفير `tools/i18n` checker أو Gradle task يقارن المفاتيح وأنواع XML ويعالج plural items والـ placeholders والـ escaping والروابط.
- [ ] Robolectric/JUnit: AppLanguage fallback/persistence/system/per-app، AppLanguage+SteamLanguage(`ar`)، Arabic plurals، bidi helpers.
- [ ] Compose/UI screenshot tests لجميع الشاشات الأساسية، مع `ar` و`en` وpseudo locale `ar-XB` إن أمكن، مع تأكيد قابلية الوصول والـ focus.
- [ ] عرض RTL على هاتف فعلي وأحجام شاشة 16:9/4:3/square، مع 75/100/150% وتضخيم خط النظام.
- [ ] Smoke/regression: استعادة Session، تنزيل خلفية، تحديث التطبيق، إشعارات، إدارة الملفات، المتاجر، وحدة التحكم، واجهة الألعاب، تغيير اللغة أثناء فتح الإعدادات، والعودة للإنجليزية.
- [ ] تشغيل `./gradlew -PskipRust=true :app:testDebugUnitTest` مع Android lint، ثم بناء وتثبيت APK بعد تجهيز مدخلات runtime وفق `tools/build_local.sh`، وتسجيل أدلة ناجحة وفاشلة.
**بوابة AR-06:** جميع الفحوص الإلزامية تنجح ويُرفق تقرير وصور/فيديوهات وقيود الأجهزة.

### AR-07: مراجعة وإصدار
- [ ] تنظيم الـ PRs: البنية، الموارد، النصوص الثابتة، RTL، Steam/Linux، اختبارات CI.
- [ ] مراجعة نتائج كل PR ودمجها بالتسلسل بعد اجتياز بوابتها؛ لا تغيّر `targetSdkVersion` أو هوية الحزمة أو التوقيع أو إعدادات اللعبة لمجرد التعريب.
- [ ] ملف glossary وcoverage report وtest matrix وknown limitations في `docs/i18n/`.
- [ ] إعلان جاهزية التعريب عندما تكون **التغطية 100%** لموارد DroidDeck المؤهلة بما فيها الجمع والقوائم، و**صفر أخطاء placeholders**، و**صفر نص واجهة صلب غير موثق**، و**صفر RTL/blocker errors**، واجتياز الاختبارات الآلية وعلى جهاز حقيقي.
- [ ] لا يُدمج `main` ولا يُطلق APK رسمي دون مراجعة واضحة ونتائج build وQA مؤكدة.

## 3. قاموس مصطلحات مبدئي قابل للمراجعة

| English | المقابل العربي |
| --- | --- |
| Setup | الإعداد |
| Library | المكتبة |
| Games | الألعاب |
| Session | الجلسة |
| Resume | استئناف |
| Download | تنزيل |
| Updates | التحديثات |
| Components | المكوّنات |
| Driver | برنامج التشغيل |
| Performance | الأداء |
| Controller | وحدة التحكم |
| Overlay | الواجهة العائمة |
| Compatibility layer | طبقة التوافق |
| Runtime | بيئة التشغيل |
| Store | المتجر |
| File Manager | مدير الملفات |

يبقى `DroidDeck` و`Steam` و`Proton` و`FEX` و`DXVK` و`LSFG` و`GOG` و`Epic Games` و`Amazon Games` أسماء تقنية أو تجارية كما هي، ويمكن وصفها بالعربية عند الحاجة.

## 4. ملاحظات إدارة المخاطر

- تمكين RTL على مستوى التطبيق قد يؤثر على الواجهة والـ overlays؛ تحقق أولًا من عزل أسطح اللعبة ومسارات الإدخال.
- Android `Locale.setDefault` وتحديث `resources.updateConfiguration` قد يؤثران على التشغيل متعدد الـ Activity/Service؛ اختبر دورة الحياة ولا تفترض نجاحها لمجرد عرض النص.
- دعم Steam للعربية موجود في التحويل، لكن توفر جميع الترجمات هناك خارج ضمان DroidDeck.
- تغيير اللغة لا يجب أن يتلف أسماء ملفات أو ألعاب، مسارات، جداول، بروتوكولات أو معرّفات أخرى.
- هذا المستند خطة قابلة للتنفيذ؛ علامة [x] لا توضع دون diff واختبار حقيقي ودليل.
