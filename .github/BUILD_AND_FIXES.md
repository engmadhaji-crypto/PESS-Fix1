# PESS — الإصلاحات وطريقة بناء APK

## ما أُصلح
1. app/build.gradle.kts: كان البناء المحلي يفشل لأن debug.keystore غير موجود (مستثنى في .gitignore)
   وكلمات مرور release تأتي من متغيرات بيئة فارغة. صار التوقيع يعمل بدونها (debug الافتراضي)،
   ويستخدم مفتاح release فقط عند توفر KEYSTORE_PATH وSTORE_PASSWORD وKEY_PASSWORD.
2. PdfInvoiceGenerator.kt: إزالة `!!` عند MediaStore واستبدالها برسالة خطأ واضحة.

3. استيراد Backup v2 (القيم الخام بوحدات صغرى تُقسم على 100 مرة واحدة فقط) (ملف PESS-backup-*.json): ملف جديد BackupV2Converter.kt يحوّل
   العملاء/العقود/الأرقام/دفتر الحركات إلى نموذج Room، ويُستدعى تلقائياً من parseAndValidate،
   فيعمل مع شاشة الاستيراد الحالية. على النسخة المرفقة: 67 عميلاً، 120 عقداً، 325 قسطاً
   (252 مدفوع، 43 جزئي، 30 قادم). القواعد: دفعة المقدّم لا تُحسب مرتين، الدفعات تُوزَّع على
   الأقساط بالترتيب، والمدفوع الزائد (4 عقود) يُسجَّل في ملاحظات العقد. التواريخ المجهولة تبقى
   0 ولا تُخترع. 24 عميلاً بلا رقم هاتف يُستوردون بهاتف فارغ (مزامنة Firestore تتطلب هاتفاً).
   لا يُستورد: الشعار/الختم/التوقيع، قرارات الإدارة، سجل التدقيق، بيانات الشركة (لا نموذج لها بعد).
4. InstallmentCalculator: تاريخ الاستحقاق 0 (مجهول) لا يُعدّ "متأخراً" أبداً.
5. اختبار جديد: app/src/test/.../BackupV2ConverterTest.kt (لم يُشغَّل هنا).

6. مكتبة الأصول (شعار/ختم/توقيع): جدول pess_assets (Room v3 مع Migration 2→3 غير هدّامة)،
   الصور تُحفظ ملفات PNG في filesDir (حد 2 ميجابايت و2400 بكسل) والبيانات الوصفية في Room.
   زر "تصدير PDF" يفتح نافذة اختيار الشعار/الختم/التوقيع لكل كشف على حدة (لا شيء مختار افتراضياً).
   استيراد Backup v2 يستورد الصور الستة أيضاً (المؤرشفة تبقى مؤرشفة). الملفات الجديدة:
   AssetEntity.kt, AssetDao.kt, AssetRepository.kt, StatementOptionsDialog.kt.
   لم تُختبر على جهاز: ترتيب الشعار في ترويسة PDF العربية يحتاج معاينة بصرية.

## بناء APK تلقائياً على GitHub
ارفع المشروع إلى المستودع، ثم Actions > Build PESS APK > Run workflow. بعد انتهائه نزّل
الملف PESS-debug-apk من قسم Artifacts. (الملف: .github/workflows/build-apk.yml، لم يُجرَّب هنا.)

## بناء APK
1. ثبّت Android Studio (أحدث إصدار يدعم AGP 9.1) مع Android SDK 36 وJDK 17+.
2. افتح مجلد PESS_PV-AGENTS ودعه يزامن (يحمّل Gradle 9.3.1 تلقائياً).
3. Build > Build APK(s)  — أو من الطرفية بعد `gradle wrapper`:  ./gradlew assembleDebug
4. الناتج: app/build/outputs/apk/debug/app-debug.apk

## ملاحظات لم تُعدَّل (تحتاج قرارك)
- AppDatabase إصدار 2 بلا Migration (والـ fallback المدمّر أُزيل سابقاً). ترقية قاعدة v1 موجودة ستنهار.
- مفاتيح Firebase في firebase-applet-config.json: قيّدها في Google Cloud Console.
- allowBackup=true وقواعد النسخ السحابي فارغة: بيانات العملاء المالية تُنسخ لحساب Google.
- المبالغ بنوع Double؛ يفضَّل Long (بالفلس/السنت) للدقة المالية.
- لم يُشغَّل أي بناء أو اختبار هنا (لا Android SDK ولا شبكة).
