# PESS — حالة التنفيذ

## ما تم تنفيذه في هذه الدفعة

اعتمد المشروع Room 2.7.0 كمصدر Android المحلي الوحيد، وهو إصدار أحدث من الحد المطلوب 2.6.1. كما أن `minSdk=24` و`compileSdk=36` و`targetSdk=36` كانت متحققة مسبقًا.

أزيل fallback الهدّام من `AppDatabase`. عند غياب migration مناسبة لن تُحذف الجداول تلقائيًا؛ يجب إضافة migration مختبرة أو عرض مسار استرداد واضح قبل ترقية قاعدة مستخدم موجودة.

تم تحديث تنسيق المبالغ ليكون رقميًا بخانتين عشريتين دون عملة ضمنية، وتحديث التاريخ إلى `yyyy-MM-dd`. التاريخ المفقود يعرض سلسلة فارغة، ولا يستعمل `created_at` أو وقت الإنشاء بديلًا عن تاريخ الدفع.

تم تحديث PDF ليُحفظ فعليًا في `Downloads/PESS`: يستخدم MediaStore في Android 10 فأحدث، ويستخدم مجلد التطبيق `Download/PESS` مع FileProvider في Android 7–9. يتم التحقق من أن الملف غير فارغ ويبدأ بترويسة `%PDF` قبل إرجاع URI. اسم الملف مشتق من اسم العميل ومعرّفه الحقيقي.

أضيف زر فتح WhatsApp بالنص. يُنظّف الرقم من غير الأرقام ولا يضيف رمز دولة افتراضيًا، ثم يفتح `wa.me` مع نص عام يترك الإرسال النهائي للمستخدم. لا يدّعي التطبيق إرفاق PDF أو الإرسال التلقائي إلى رقم محدد.

أضيفت اختبارات وحدات لتنسيق المبالغ وتنسيق التاريخ الفارغ والتاريخ ISO.

## الملفات المعدلة

- `app/src/main/java/com/example/data/local/AppDatabase.kt`: إزالة الحذف التلقائي عند غياب migration.
- `app/src/main/java/com/example/domain/util/InstallmentCalculator.kt`: تنسيق المال والتاريخ.
- `app/src/main/java/com/example/data/pdf/PdfInvoiceGenerator.kt`: حفظ PDF في Downloads/PESS والتحقق واسم الملف.
- `app/src/main/res/xml/file_paths.xml`: السماح بمشاركة ملفات Download/PESS القديمة عبر FileProvider.
- `app/src/main/java/com/example/ui/screens/CustomerDetailScreen.kt`: زر WhatsApp بالنص.
- `app/src/test/java/com/example/InstallmentSystemTest.kt`: اختبارات التنسيق.

## ما لم يُنفذ بعد

لم يُنفذ بعد استيراد Backup v2 الخام إلى نموذج Room الحالي، ولا مولد XLSX فعلي، ولا مكتبة الأصول داخل Room، ولا شاشة كشف الحساب بخيارات الشعار/الختم/التوقيع/QR، ولا مسار PDF منفصل يعتمد WebView وPrintDocumentAdapter. السبب أن هذه الأجزاء تحتاج تصميم مخطط بيانات وعقود واجهة إضافية، ولا يجوز إدخالها ببيانات أو تواريخ افتراضية.

## الاختبار الفعلي

نجح `git diff --check` ولم يحدث أي تعديل في المستودع البعيد. لم يمكن تشغيل Gradle أو اختبارات JVM/Android أو `adb install` لأن نسخة المشروع لا تحتوي `gradlew` التنفيذي وبيئة التنفيذ لا تحتوي JDK compiler أو Android SDK أو `adb` أو `aapt2` أو `apksigner`. لذلك لا يُعد هذا البناء أو المشاركة أو MediaStore مختبرًا فعليًا على جهاز Android حتى يتم توفير بيئة Android.

لقطة الرجوع قبل التعديل محفوظة خارج المستودع في `/home/ubuntu/work/snapshots/PESS_PV-AGENTS-20261119T164054Z.tar.gz` وبصمتها في الملف المجاور `.sha256`.
