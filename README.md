# صفحات مشاركة تطبيق My Phone

موقع ثابت صغير يُستضاف مجاناً على GitHub Pages. عند مشاركة إعلان أو متجر
يصبح الرابط المُرسَل صفحةً تعرض المحتوى نفسه (صورة، سعر، مواصفات) مع شريط
علوي «فتح بالتطبيق» — تماماً كما تفعل تطبيقات مثل iQ Cars.

عند الضغط على «فتح بالتطبيق»:

- **التطبيق منصّب** → يُفتح على الإعلان المقصود مباشرة.
- **غير منصّب** → تُحوَّل الصفحة تلقائياً إلى Google Play أو App Store حسب الجهاز.

يعتمد ذلك على السكيم الخاص `myphone://` المضاف أصلاً في
`AndroidManifest.xml` و `ios/Runner/Info.plist`، فلا يحتاج أي تحقق من
جوجل أو آبل ولا أي بيانات من حسابات النشر.

## النشر

1. أنشئ منظمة (Organization) مجانية على GitHub باسم قصير، مثل `myphone-iq`.
   اسم المنظمة هو ما يحدد الدومين: `https://myphone-iq.github.io`
2. أنشئ داخلها مستودعاً **عاماً** باسم مطابق تماماً لـ `myphone-iq.github.io`.
3. ارفع محتويات هذا المجلد إلى جذر المستودع (لا داخل مجلد فرعي).
4. `Settings` ← `Pages` ← Deploy from a branch ← `main` ← `/ (root)`.

## قيم يجب تعبئتها قبل الرفع

| المكان | القيمة |
|---|---|
| `post/index.html` و `store/index.html` | `SUPABASE_URL` و `SUPABASE_KEY` من ملف `.env` |
| `lib/config/app_links.dart` ← `kWebBaseUrl` | الدومين بعد إنشائه، مثال `https://myphone-iq.github.io` |

المفتاح المقصود هو المفتاح العام (anon) وهو مُصمَّم ليكون علنياً في الواجهات؛
سياسات RLS في `supabase/schema_stage_9_security_hardening.sql` تسمح لغير
المسجَّلين بقراءة الإعلانات المعتمدة فقط (`status = 'approved'`)، ولا تسمح
بأي كتابة. لا تضع أبداً مفتاح `service_role` هنا.

ما دام `kWebBaseUrl` فارغاً يعود التطبيق تلقائياً إلى إرسال روابط المتاجر
المباشرة في نص المشاركة، فلا شيء ينكسر قبل نشر الموقع.

## ترقية اختيارية لاحقاً: App Links

ما سبق يفتح التطبيق بعد ضغطة واحدة من المستخدم (ومع سؤال تأكيد على iOS).
لجعل الرابط يفتح التطبيق **دون أي وسيط ولا صفحة ويب**، تُضاف طبقة
App Links / Universal Links، وتحتاج:

| الملف | القيمة | مصدرها |
|---|---|---|
| `.well-known/assetlinks.json` | بصمة SHA-256 | Play Console ← Test and release ← Setup ← App integrity ← App signing key certificate |
| `.well-known/apple-app-site-association` | Team ID | developer.apple.com ← Membership details |

الملفان موجودان هنا جاهزين بانتظار القيمتين، ويجب أن يبقى ملف `.nojekyll`
موجوداً وإلا تجاهَل GitHub Pages مجلد `.well-known` كلياً (لأن اسمه يبدأ بنقطة).

بعد تعبئتهما يُضاف `intent-filter` بخاصية `autoVerify` على أندرويد،
وقدرة Associated Domains على iOS. الملف `lib/services/deep_link_service.dart`
يتعامل مع الشكلين مسبقاً (`myphone://post/<id>` و `https://…/post/?id=<id>`)
فلا يحتاج تعديلاً حينها.

> ملاحظة: لا تدعم الاستضافة الثابتة الروابط المؤجَّلة (Deferred Deep Linking)،
> أي أن من ينصّب التطبيق **بعد** ضغط الرابط سيصل إلى الشاشة الرئيسية لا إلى
> الإعلان. هذا يتطلب خدمة وسيطة مثل Branch.io.
