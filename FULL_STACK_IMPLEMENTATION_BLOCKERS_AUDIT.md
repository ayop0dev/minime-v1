# تدقيق معوقات جاهزية التنفيذ الشامل — Minime V1

## 1. الحكم التنفيذي

**غير جاهز للتنفيذ.**

تمت قراءة **141 ملفًا كاملة** قبل كتابة هذا التقرير: 139 ملفًا موجودًا في شجرة العمل الحالية، والنسخة التاريخية المحذوفة من `social.accounts.oauth.boundary.v1.md`، وملف `FULL_REPOSITORY_EXPORT.txt` التاريخي من Git حتى سطره الأخير (57,058). لم تُعامل النسخ التاريخية أو الملفات المحلية أو المواصفات المكررة كمواد قابلة للتجاهل.

المستودع غني بالقرارات، لكنه ليس عقد تنفيذ متماسكًا. توجد عقود ممتازة الصياغة منفردة، ثم تتصادم عند حدود التسجيل، والحذف، والروابط الصادرة، والتحليلات، والتخزين، والذكاء الاصطناعي، والتخزين المؤقت. الأخطر أن بعض الوعود الأساسية لا يمكن ضمانها بالتقنيات التي اختارتها المواصفات نفسها: حدث حذف الحساب غير متين، ومع ذلك يعتمد عليه إبطال الجلسات وجدولة الحذف؛ نقرة واحدة يفترض أن تنتج حدثًا واحدًا، بينما إعادة محاولة المهمة تسمح صراحةً بالتكرار؛ وتأخير العرض العام محدد بستين ثانية بينما توجد طبقتا تخزين مؤقت مستقلتان مدة كل منهما ستون ثانية.

البدء بالتنفيذ الآن لن ينتج «نسخة V1 مطابقة للمواصفات». سينتج عدة تفسيرات متنافسة، ثم ترقيعات داخل الكود تتحول فعليًا إلى معمارية غير موثقة.

---

## 2. أهم 20 معوقًا للتنفيذ

### 1. اكتمال التسجيل لا يملك معاملة قابلة للتنفيذ

- **الخطورة:** حرجة.
- **المجال:** الحساب، المصادقة، اسم المستخدم، QR، التخزين.
- **الملفات:** `implementation/03-canonical-data-model.md`، `implementation/04-service-contracts.md`، `implementation/05-api-contracts.md`، `account/minime.account.claim.system.specification.v1.md`، `qr-code/qr-code.generation.specification.v1.md`.
- **ما يحاول المهندس تنفيذه:** تحويل حجز اسم ومصادقة ناجحة إلى حساب نشط، وهوية مصادقة، ومحتوى ملف، ورمز QR صالح.
- **سبب التوقف:** `Account.status` يبدأ افتراضيًا بـ`active` (`03`، سطر 118)، بينما QR شرط اكتمال في العقود الحالية ويحتاج كتابة PostgreSQL ورفع SVG إلى Object Storage. لا توجد حالة `pending` ولا بروتوكول تعويض إذا نجحت المعاملة وفشل الرفع، أو العكس. وفوق ذلك يقول `AccountService.createAccount` إنه ينشئ هوية المصادقة، مع أن `AuthRepository` محظور عليه (`04`، السطران 126 و164 تقريبًا).
- **الخطر:** حساب نشط بلا QR، أصل يتيم، حجز اسم مستهلك بلا حساب صالح، أو تكرار هوية عند إعادة الطلب.
- **الإصلاح المطلوب:** تعريف منسق تسجيل واحد، وحدود معاملة قاعدة البيانات، وحالة ما قبل التفعيل أو تعويض متين، ومفتاح idempotency، ومالك وحيد لإنشاء هوية المصادقة.
- **تصنيف الإصلاح:** معماري.

### 2. حذف الحساب يعتمد على حدث غير متين

- **الخطورة:** حرجة.
- **المجال:** الحساب، المصادقة، الوظائف الخلفية، كل البيانات التابعة.
- **الملفات:** `implementation/06-event-contracts.md`، `04-service-contracts.md`، `10-background-jobs.md`، `account/account.deletion.policy.v1.md`.
- **ما يحاول المهندس تنفيذه:** إبطال كل الجلسات «لحظة التأكيد» وبدء مسح البيانات.
- **سبب التوقف:** بعد تثبيت `status='deleted'` ينشر النظام `account.deleted`. المواصفة نفسها تقر بأن EventEmitter غير متين وقد يُفقد عند إعادة التشغيل (`06`، سطر 40). إبطال الجلسات وجدولة BullMQ كلاهما مستهلكان لذلك الحدث. انهيار العملية بعد commit وقبل handler يترك جلسات حية ولا ينشئ مهمة الحذف.
- **الخطر:** وصول مصادق لحساب محذوف، وبيانات لا تُحذف، مع استجابة API تدعي أن الحذف نهائي.
- **الإصلاح المطلوب:** outbox متين داخل معاملة الحذف، أو إدراج مهمة BullMQ وإبطال الجلسات ضمن مسار مضمون قابل للاستعادة، مع مراقب مصالحة للحسابات العالقة.
- **تصنيف الإصلاح:** معماري.

### 3. ملكية الأحداث متناقضة مع تنفيذ EventEmitter

- **الخطورة:** حرجة.
- **المجال:** الأحداث والتحليلات.
- **الملفات:** `platform/events/event.delivery.specification.v1.md`، `event.lifecycle.specification.v1.md`، `events.architecture.specification.v1.md`، `implementation/06-event-contracts.md`.
- **ما يحاول المهندس تنفيذه:** «سجّل قبل التوصيل» وتاريخ أحداث دائم.
- **سبب التوقف:** المنصة المعمارية تشترط تسجيل الحدث قبل إتاحته، بينما التنفيذ لا يعرّف مخزن أحداث أصلًا ويستخدم EventEmitter غير المتين. لا يوجد schema أو repository أو transaction للحدث التجاري.
- **الخطر:** لا يمكن الوفاء بالتاريخ، ولا إعادة التوصيل، ولا التحقق من اكتمال التحليلات.
- **الإصلاح المطلوب:** تحديد ما إذا كانت أحداث V1 سجلات متينة أم إشعارات داخلية فقط. إن كانت متينة، يلزم Event/Outbox schema وسياسة تسليم وإعادة ومحاذاة المستهلكين.
- **تصنيف الإصلاح:** معماري.

### 4. دقة عدّ النقرات منقوضة صراحةً

- **الخطورة:** حرجة.
- **المجال:** Out Links، Analytics، BullMQ.
- **الملفات:** `out-links/out-link.analytics.event.specification.v1.md`، `out-link.tracking.policy.v1.md`، `implementation/10-background-jobs.md`.
- **ما يحاول المهندس تنفيذه:** Redirect واحد = حدث نقرة واحد بالضبط.
- **سبب التوقف:** مهمة النقر تعاد عند الفشل، والمواصفة تقول إن التكرار «مقبول لدقة V1» (`10`، سطر 117). لا يوجد `event_id` حتمي أو unique constraint مبني على job id. كما أن عبارة «بعد إرسال الرد بنجاح» ليست نقطة commit قابلة للرصد موثوقًا من تطبيق HTTP.
- **الخطر:** أرقام وCTR خاطئة، خصوصًا وقت الأعطال، مع ادعاء أن الأحداث حقائق.
- **الإصلاح المطلوب:** مفتاح idempotency ثابت لكل نقرة، وقيد uniqueness، وتعريف دقيق لنقطة احتساب النقرة وعلاقة enqueue بإرسال 302.
- **تصنيف الإصلاح:** معماري.

### 5. إجمالي نقرات الحساب غير قابل للاستعلام بالعقد الحالي

- **الخطورة:** حرجة.
- **المجال:** Analytics، Out Links، البيانات.
- **الملفات:** `implementation/03-canonical-data-model.md`، `04-service-contracts.md`، `analytics/profile.analytics.specification.v1.md`.
- **ما يحاول المهندس تنفيذه:** `getAnalyticsSummary(account_id)` و«إجمالي النقرات».
- **سبب التوقف:** حدث `link_click` يحتوي `out_link_id` فقط؛ `AnalyticsService` ممنوع من `OutLinkRepository`. بعد purge للرابط لا يبقى طريق لإسناد النقرة إلى الحساب. لا يوجد `account_id` في حدث النقر ولا read contract عابر للمجال.
- **الخطر:** الاستعلام إما مستحيل أو يتطلب كسر الحدود، ويتلف «All Time» بعد حذف الروابط.
- **الإصلاح المطلوب:** إما تضمين `account_id` snapshot في حدث النقرة، أو تعريف read model/عقد استعلام مستدام يحفظ الإسناد بعد purge.
- **تصنيف الإصلاح:** معماري.

### 6. عقد Button/OutLink/Rendering يعرّف وجهتين مختلفتين

- **الخطورة:** حرجة.
- **المجال:** Blocks، Rendering، Out Links.
- **الملفات:** `blocks/button.block.specification.v1.md`، `rendering/block.renderers.specification.v1.md`، `rendering/rendering.architecture.canon.v1.md`، `implementation/03` و`04`.
- **ما يحاول المهندس تنفيذه:** زر عام متتبع.
- **سبب التوقف:** مواصفة الزر تجعله مالكًا لـ`url`، ومواصفة renderer تقول الحفاظ عليه وعدم لفه أو تتبعه (`block.renderers`، سطر 97 تقريبًا)، بينما canonical rendering يقول إن Button Renderer يقرأ Out Link reference (`rendering canon`، سطر 485)، ونموذج Block لا يحتوي ذلك المرجع. التنفيذ يستنتج الرابط بواسطة `(block_id, connected_account_id=null)`.
- **الخطر:** روابط خام تتجاوز القياس، أو أزرار تختفي إذا فشل إنشاء OutLink، أو اقتران خفي غير محمي بقيود قاعدة البيانات.
- **الإصلاح المطلوب:** عقد وحيد: هل الـBlock يخزن `out_link_id` أم يتم إنشاء projection مضمون؟ ثم تعريف المعاملة بين إنشاء/تحديث Block وأرشفة/إنشاء OutLink.
- **تصنيف الإصلاح:** معماري.

### 7. تحديث الكتلة والرابط الصادر بلا ذَرّية

- **الخطورة:** عالية.
- **المجال:** Profile، Out Links.
- **الملفات:** `implementation/04-service-contracts.md`، `03-canonical-data-model.md`.
- **ما يحاول المهندس تنفيذه:** تغيير label أو URL مع حفظ تاريخ الرابط.
- **سبب التوقف:** العملية تكتب Block عبر repository مملوك لـProfile، ثم تؤرشف وتخلق OutLink عبر خدمة أخرى. لا توجد معاملة مشتركة أو saga أو ترتيب تعويض. Last Write Wins لا يحل نصف العملية.
- **الخطر:** Block جديد مع رابط قديم، رابطان active، أو لا رابط active.
- **الإصلاح المطلوب:** عقد transaction coordinator واضح وقيد يمنع أكثر من OutLink active لكل clickable identity.
- **تصنيف الإصلاح:** تخطيط تنفيذ.

### 8. نموذج المظهر يطلب ويمنع تخصيص الكتلة في الوقت نفسه

- **الخطورة:** عالية.
- **المجال:** Appearance، Block Styling، Profile.
- **الملفات:** `appearance/themes/theme.constraints.specification.v1.md`، `theme.definition.specification.v1.md`، جميع ملفات `block-styling/`، `implementation/03` و`04` و`07`.
- **ما يحاول المهندس تنفيذه:** `style_overrides` والتحقق منها.
- **سبب التوقف:** Appearance يقول صراحةً لا تبنوا block-level overrides قبل مراجعة V1 (`theme.constraints`، سطر 241)، بينما ستة ملفات كاملة تجعله جزءًا من V1 والنموذج الحالي يحمله. لا يوجد catalog تنفيذي نهائي للخصائص والأنواع والحدود؛ معالجة override غير الصالح عند تغيير theme تعرض ثلاث استراتيجيات بلا اختيار.
- **الخطر:** واجهات مختلفة، قيم محفوظة لا تُرسم، أو theme switch يكسر الملف.
- **الإصلاح المطلوب:** قرار V1 واحد، ثم JSON schema مغلق لكل block type، وسياسة واحدة لـtheme switch، ورفض unknown keys.
- **تصنيف الإصلاح:** معماري.

### 9. `display_name` يبدأ فارغًا ويجب ألا يكون فارغًا

- **الخطورة:** عالية.
- **المجال:** التسجيل، Profile، SEO.
- **الملفات:** `profile-content/profile.content.lifecycle.v1.md`، `profile.content.specification.v1.md`، `implementation/03` و`07`.
- **ما يحاول المهندس تنفيذه:** إنشاء ProfileContent أثناء التسجيل.
- **سبب التوقف:** lifecycle يحدد `Display Name = Empty` كحالة ابتدائية صحيحة (سطر 104)، بينما specification يجعله required (سطر 232)، وDB/validation يجعلان non-empty. بيانات provider موصوفة في عقد المصادقة بأنها غير سلطوية، ولا يحتوي طلب التسجيل display_name.
- **الخطر:** لا يمكن تنفيذ INSERT قانوني دون اختراع قيمة من username أو provider.
- **الإصلاح المطلوب:** جعل الحقل nullable/empty حتى onboarding، أو إضافته صراحةً لطلب التسجيل مع قاعدة مصدر واضحة.
- **تصنيف الإصلاح:** توثيق.

### 10. طبقتا Cache تكسران حد الستين ثانية

- **الخطورة:** عالية.
- **المجال:** Rendering، Cache، الحساب.
- **الملفات:** `implementation/09-caching-strategy.md`، `public-profile/public-profile.cache.policy.v1.md`، خريطة المنتج.
- **ما يحاول المهندس تنفيذه:** تحديث عام خلال 60 ثانية كحد أقصى.
- **سبب التوقف:** التنفيذ يعرّف طبقتين مستقلتين (`09`، سطر 38)، لكل واحدة TTL=60. قد يعيد edge cache جلب نسخة Redis عمرها 59 ثانية ثم يخزنها 60 ثانية أخرى. كما لا توجد purge للـedge عند الحذف أو التعليق أو تغيير GTM.
- **الخطر:** 120 ثانية تقريبًا من القدم، وقد يُعرض حساب محذوف/معلّق.
- **الإصلاح المطلوب:** ميزانية freshness مشتركة، وage propagation، أو TTL إجمالي، وpurge فوري للحالات الأمنية والحذف.
- **تصنيف الإصلاح:** معماري.

### 11. التخزين يحذف الأصل القديم بالترتيب الخاطئ

- **الخطورة:** عالية.
- **المجال:** Storage، Profile.
- **الملفات:** `implementation/11-platform-services.md`، `platform/storage/asset.lifecycle.specification.v1.md`، `storage.architecture.specification.v1.md`.
- **ما يحاول المهندس تنفيذه:** استبدال avatar دون انقطاع.
- **سبب التوقف:** implementation يطلب حذف `old_key` قبل تثبيت المرجع الجديد، بينما lifecycle المعماري يقول commit المرجع الجديد ثم يصبح القديم orphan. لا يوجد تعويض إذا فشل upload أو DB update.
- **الخطر:** ملف شخصي يشير إلى أصل محذوف أو أصل جديد يتيم.
- **الإصلاح المطلوب:** upload → validate → DB swap transaction → delete old async، مع cleanup للأصول غير المرتبطة.
- **تصنيف الإصلاح:** تخطيط تنفيذ.

### 12. أنواع الصور متناقضة وملف المعالجة مفقود

- **الخطورة:** عالية.
- **المجال:** Storage، uploads.
- **الملفات:** `storage.architecture.specification.v1.md`، `implementation/07-validation-rules.md`، `11-platform-services.md`.
- **ما يحاول المهندس تنفيذه:** قبول وتحويل الصور إلى WebP.
- **سبب التوقف:** Storage يقبل HEIC ولا يقبل animated GIF دائمًا (سطر 576)، validation يقبل GIF ولا يذكر HEIC (`07`، السطران 451 و463). المرجع `platform/storage/asset.processing.specification.v1.md` مذكور (سطر 660) لكنه غير موجود.
- **الخطر:** اختلاف frontend/backend، أخطاء مكتبة التحويل، وقبول ملفات لا يمكن معالجتها بأمان.
- **الإصلاح المطلوب:** allowlist واحد، سياسة animation، limits دقيقة، فحص magic bytes، وعقد processing موجود فعليًا.
- **تصنيف الإصلاح:** توثيق.

### 13. AI يملك ولا يملك كيانًا في الجملة نفسها

- **الخطورة:** عالية.
- **المجال:** AI، Data، API.
- **الملفات:** `platform/ai/ai.architecture.specification.v1.md`، `platform/data/canonical.entities.map.v1.md`، `implementation/03` و`11` و`13`.
- **ما يحاول المهندس تنفيذه:** Analysis Sessions قابلة لإعادة الاستخدام.
- **سبب التوقف:** التنفيذ يعرّف `AnalysisSession` (`03`، سطر 621)، ثم يقول «لا persistence للذكاء الاصطناعي» (`11`، سطر 303)، و«لا يملك أي كيان DB» (سطر 557)، ثم «AnalysisSession هو الكيان الوحيد المملوك للـAI» (سطر 564). الخريطة التاريخية ما زالت تعرف `AI Suggestion` وقرارات مقبولة بشكل مختلف.
- **الخطر:** schema وrepository وسياسة الحذف والخصوصية تختلف حسب الملف الذي قرأه المهندس.
- **الإصلاح المطلوب:** تعريف فئة الكيان ومالكه وعمره وحقوله النهائية، وإزالة الجمل المتعارضة.
- **تصنيف الإصلاح:** معماري.

### 14. لا توجد API تنفيذية لجلسة التحليل

- **الخطورة:** عالية.
- **المجال:** AI، Frontend، Backend.
- **الملفات:** `implementation/03-canonical-data-model.md`، `11-platform-services.md`، `05-api-contracts.md`، `13-implementation-plan.md`.
- **ما يحاول المهندس تنفيذه:** زر «حلّل ملفي»، حالة pending/completed/failed، ثم عرض التقرير.
- **سبب التوقف:** الكيان والخدمة والخطة موجودة، لكن `05-api-contracts` لا يقدم endpoint أو DTO أو polling contract أو error contract أو schema نهائيًا لـ`report/scores/suggestions`. لا توجد دلالة accept/reject/apply.
- **الخطر:** frontend وbackend سيخترعان بروتوكولين مختلفين.
- **الإصلاح المطلوب:** API كاملة وتحويلات الحالة وschema إصدارية وإلغاء/إعادة ومحاذاة rate limits.
- **تصنيف الإصلاح:** توثيق.

### 15. Connected Accounts يرفض حسابًا «غير موجود» رغم حظر فحص الوجود

- **الخطورة:** عالية.
- **المجال:** Connected Accounts، Social Accounts.
- **الملفات:** `account-management/connected.accounts.specification.v1.md`، `minime.account.management.system.specification.v1.md`.
- **ما يحاول المهندس تنفيذه:** إضافة حساب اجتماعي يدويًا.
- **سبب التوقف:** الأول يقول إن النظام يتحقق من التنسيق فقط ولا يفحص الوجود؛ الثاني يقول «If the account cannot be found: Creation Rejected» (سطر 284).
- **الخطر:** أحد الفريقين يبني طلبات خارجية ممنوعة والآخر لا يبنيها.
- **الإصلاح المطلوب:** حذف شرط الوجود وتوحيد معنى validation على أنه format فقط، أو إدخال عقد تحقق خارجي صريح؛ لا يمكن الإبقاء على الاثنين.
- **تصنيف الإصلاح:** توثيق.

### 16. سجل المنصات الاجتماعية غير متسق

- **الخطورة:** عالية.
- **المجال:** Social Accounts، validation، frontend.
- **الملفات:** `social.accounts.platform.rules.v1.md`، مجلد `social-platforms/`، `implementation/03-canonical-data-model.md`.
- **ما يحاول المهندس تنفيذه:** enum واحد وقواعد platform registry.
- **سبب التوقف:** الوثيقة تشير إلى مجلد غير موجود `social-accounts/platform/`، وتضم GitHub وPinterest بلا ملفي قواعد وبلا وجود في enum؛ بينما enum يضم Snapchat/Spotify/Telegram/Twitch. لا توجد آلية إصدار أو اختبار تطابق registry مع enum.
- **الخطر:** خيارات تظهر في UI ولا يقبلها backend أو قواعد لا يمكن تحميلها.
- **الإصلاح المطلوب:** manifest آلي واحد يولّد enum والقواعد والخيارات، مع اختبار اكتمال يمنع build عند الاختلاف.
- **تصنيف الإصلاح:** تخطيط تنفيذ.

### 17. GTM يمثل تنفيذ كود طرف ثالث وليس «Container ID آمنًا»

- **الخطورة:** حرجة.
- **المجال:** Integrations، Security، Rendering، Cache.
- **الملفات:** `integrations/providers/google-tag-manager.specification.v1.md`، `implementation/12-seo-and-integrations.md`، `account/account.model.specification.v1.md`.
- **ما يحاول المهندس تنفيذه:** GTM مملوك لكل حساب في صفحته العامة.
- **سبب التوقف:** المواصفات تدعي أن قبول ID يمنع arbitrary JavaScript؛ عمليًا مالك الحاوية يستطيع نشر tags/scripts تعمل على أصل Minime. لا يوجد CSP/nonce/sandbox/consent/domain isolation أو سياسة إساءة. وتغيير `settings.gtm_container_id` غير مدرج بوضوح في invalidation.
- **الخطر:** XSS وظيفي، سرقة بيانات الصفحة، phishing، وكسر الخصوصية على نطاق المنصة.
- **الإصلاح المطلوب:** قرار أمني صريح: منع GTM لكل مستخدم، أو عزله على origin/sandbox منفصل مع CSP وسياسة مراجعة وموافقة. هذا ليس regex فقط.
- **تصنيف الإصلاح:** معماري.

### 18. قواعد التزامن تسمح بخرق القيود

- **الخطورة:** عالية.
- **المجال:** Data، Blocks، Username، Connected Accounts.
- **الملفات:** `data.architecture.specification.v1.md`، `implementation/03` و`04`.
- **ما يحاول المهندس تنفيذه:** max-one identity block، حد الكتل، حجز username، reorder.
- **سبب التوقف:** «Last Write Wins» (`data.architecture`، سطر 454) لا يمنع عمليتي create متزامنتين. لا يوجد partial unique index لـavatar/name/bio، ولا lock/serializable لحساب العدد، ولا قيد يضمن عدم تضارب Account.username مع جدول الحجز، ولا uniqueness/contiguity لـsort_order.
- **الخطر:** duplicate identity blocks، تجاوز الحد، ترتيب غير حتمي، وحجز اسم مملوك فعليًا.
- **الإصلاح المطلوب:** قيود DB ومعاملات isolation/locks وسياسة retry لكل invariant.
- **تصنيف الإصلاح:** تخطيط تنفيذ.

### 19. Audit Logging مطلوب لكنه غير موجود في التنفيذ

- **الخطورة:** عالية.
- **المجال:** Security، Operations، deletion.
- **الملفات:** `account-management/audit.logging.policy.v1.md`، `account/account.deletion.policy.v1.md`، كل `implementation/`.
- **ما يحاول المهندس تنفيذه:** الاحتفاظ بسجلات تدقيق 12 شهرًا بعد حذف الحساب.
- **سبب التوقف:** السياسة تعرف payload والوصول والاحتفاظ، والحذف يعتمد على بقاء السجل، لكن لا يوجد AuditLog model/repository/service/job/admin authorization. بعض الأحداث المذكورة مثل recovery email وpage lifecycle غير موجودة أصلًا.
- **الخطر:** لا يمكن الوفاء بالاحتفاظ أو التحقيق الأمني، ويظل سبب إبقاء Account record بلا تطبيق داعم.
- **الإصلاح المطلوب:** إما إدخال النظام التنفيذي كاملًا بما في ذلك redaction وaccess وretention، أو إزالة اعتماده من ضمانات V1.
- **تصنيف الإصلاح:** معماري.

### 20. خطة النشر لا تشغّل النظام الموصوف

- **الخطورة:** حرجة.
- **المجال:** Operations، Routing، Workers.
- **الملفات:** `implementation/01-technology-stack.md`، `02-repository-structure.md`، `13-implementation-plan.md`، API QR/public routes.
- **ما يحاول المهندس تنفيذه:** نشر Web وAPI وworkers خلف Nginx.
- **سبب التوقف:** الخطة تبني حاويتين فقط API/Web، ولا تعرف worker process رغم اعتماد الحذف والنقر والاحتفاظ عليه. Nginx يوجه `/api/*` و`/out/*` إلى Nest و`/*` إلى Next (`13`، سطر 302)، لكنه لا يوجه `/qr/*` العام إلى Nest. «current Node.js LTS» غير مثبت (`01`، سطر 39)، ولا توجد إصدارات dependencies أو rollback migration أو topology للتوسع، مع EventEmitter محلي للعملية.
- **الخطر:** الوظائف لا تنفذ، QR يصل للتطبيق الخطأ، والمثيلات المتعددة تفقد handlers محلية.
- **الإصلاح المطلوب:** topology تنفيذية كاملة، worker service، routing table، process ownership، pinning، health/readiness، migration/rollback، وسياسة multi-instance.
- **تصنيف الإصلاح:** تخطيط تنفيذ.

---

## 3. التعارضات العابرة للمجالات

| الحد | التعارض القابل للإثبات | الأثر التنفيذي |
|---|---|---|
| Account ↔ Authentication | `AccountService` ينشئ AuthenticationIdentity لكنه ممنوع من AuthRepository؛ AuthService أيضًا مالك الإنشاء. | لا مالك وحيد ولا transaction واضحة. |
| Account ↔ QR | الحساب يبدأ active بينما QR مطلوب قبل اكتمال التسجيل؛ DB وObject Storage بلا تعويض. | حالات نصف مكتملة. |
| Authentication ↔ Lifecycle | الحذف والتعليق يجب أن يبطلا الجلسات فورًا، لكن المسار حدث غير متين. | جلسات صالحة لحساب غير صالح. |
| Profile ↔ Blocks | Profile Block Reference يعرض `block_ids` ثم يقر بأن `Block.sort_order` هو المصدر؛ ProfileContent الحالي لا يحتوي القائمة. | مفهوم «ملكية المجموعة» بلا تمثيل. |
| Profile ↔ Registration | display_name فارغ وصحيح في lifecycle، required/non-empty في schema. | التسجيل لا يستطيع إنشاء الصف قانونيًا. |
| Blocks ↔ Rendering | Title/Textbox يخزنان `content.text`؛ أمثلة Render Object والـrenderer تستخدم `value`. Image يخزن `image_id` لكن renderer القديم يطلب `url/alt`. | mapping تخميني لكل نوع. |
| Blocks ↔ Out Links | Button يملك URL خام، لكن rendering الحالي يفرض `/out/{public_id}`. | مصدر وجهة مزدوج. |
| Appearance ↔ Block Styling | Appearance يؤجل overrides، Block Styling يجمدها كـV1. | نطاق V1 غير محسوم. |
| Rendering ↔ Storage | بعض العقود تتكلم عن URL مباشر/موقع عام، وأخرى تمنع كشف key وتلزم `buildPublicUrl`. | عقد تسليم وأذونات غير كامل. |
| Connected Accounts ↔ Social Accounts | فحص وجود الحساب ممنوع ومطلوب في وثيقتين معتمدتين. | سلوك backend غير محدد. |
| Analytics ↔ Events | Analytics يفترض أحداثًا صحيحة ودائمة؛ implementation يسمح بالفقد والتكرار. | التقارير ليست حقائق. |
| Analytics ↔ Out Links | link_click لا يحمل account_id؛ OutLink يُحذف بعد 90 يومًا. | All-time attribution ينكسر. |
| QR ↔ Storage | QR كيان مستقل `AccountQRCode` في الحالة الحالية، بينما النسخة التاريخية/عقود سابقة تضع `qr_config.storage_key` داخل Account. | migrations والخدمات القديمة لا تتفق. |
| AI ↔ Data | AI «لا يملك كيانًا» ويمتلك AnalysisSession؛ خريطة الكيانات القديمة تعرف AI Suggestion. | لا نموذج نهائي. |
| Deletion ↔ Retention | الحذف «نهائي فورًا» لكنه cascade غير متزامن وقد يفشل، وAudit Logs بلا تنفيذ. | الاستجابة تكذب على الحالة الفعلية. |
| Cache ↔ Events | بعض invalidation مباشر وبعضه event handler غير متين، والـedge لا يُبطل. | stale output غير محدود بالوعد. |
| SEO ↔ Visibility | SEO يتحدث عن draft/private/restricted/published، بينما V1 ينفي هذه الحالات. | robots/sitemap rules بلا inputs. |
| SEO ↔ Connected Accounts | JSON-LD `sameAs` يضم كل الحسابات حتى غير المختارة في Social Icons. | كشف روابط لم يختر المستخدم عرضها. |
| Settings ↔ Integrations | GTM صار user setting حاليًا، لكن ملفات تاريخية/تنفيذية سابقة تجعله deployment-wide؛ cache triggers لا تغطيه بثبات. | صفحة المستخدم قد تحمل container خاطئًا أو قديمًا. |
| Platform ↔ Deployment | EventEmitter لا يعبر العمليات؛ خطة الإنتاج لا تعرف عدد API instances أو worker. | السلوك يتغير مع scale. |

---

## 4. عقود التنفيذ المفقودة

### واجهات API وDTO

- API كاملة لـAnalysisSession: البدء، الحالة، النتيجة، إعادة الاستخدام، الفشل، وحدود التزامن.
- عقد dashboard جامع أو قرار واضح بأن الواجهة تجمع أي endpoints وبأي ترتيب وفشل جزئي.
- طلبات/استجابات QR الدقيقة، بما فيها SVG/PNG، headers، cache، وroute العام.
- schemas فعلية لـ`Account.settings` و`qr` و`appearance.customizations` بدل `{}`.
- schema إصدارية لـAI report/scores/suggestions.
- error code catalog ثابت لكل endpoint؛ أرقام HTTP وحدها لا تكفي لتفرع الواجهة.
- pagination query/meta حقيقية لكل collection أو حذف الوعد العام بها.
- عقد Google callback/handoff: transport، state، nonce، single-use، redirects، وأخطاء المستخدم.
- عقد provider linking/unlinking الحالي، وتحديد primary provider الذي تشير إليه الوثائق ولا يمثله schema بوضوح.

### الخدمات والمعاملات

- transaction coordinator للتسجيل، QR، Block+OutLink، ConnectedAccount removal، وavatar replacement.
- isolation level، locking، وretry rules لكل invariant متزامن.
- idempotency keys للتسجيل، OTP consume، refresh rotation، click ingestion، AI analysis، uploads.
- compensation matrix لكل عملية تجمع PostgreSQL وRedis وS3 وBullMQ.
- عقد dead-letter/manual recovery للحذف والتحليلات والأصول.

### التحقق

- أطوال display name، title، textbox، button label، contact phone/WhatsApp/location.
- schema مغلق لكل JSONB ورفض unknown properties.
- URL normalization ومنع scheme confusion وcontrol characters وopen-redirect abuse.
- upload limits رقمية، dimensions، decompression bomb، magic-byte check، animation، HEIC.
- GTM security validation أوسع من regex.
- قواعد ترتيب arrays: uniqueness، completeness، duplicates، gaps، والـIDs المحذوفة.

### الأحداث والأخطاء

- Event envelope فعلي ومخزن، أو حذف ادعاء المتانة.
- event payloads كاملة لكل اسم في catalog، مع version.
- delivery semantics: at-most/at-least/exactly once، ordering per aggregate، retry، deduplication.
- mapping موحد لأخطاء `/out`: الوثائق تجمع 404 و410 وcodes مختلفة.
- correlation/request/job/event IDs وعلاقة logging بها.

### الوظائف الخلفية والتخزين المؤقت

- schedules دقيقة لا أمثلة «كل 5/15 دقيقة» فقط.
- attempts/backoff/jitter/timeouts وأقصى عمر للمهمة.
- worker concurrency وgraceful shutdown وstalled-job recovery.
- edge purge contract وcache-control/Vary/ETag وnegative-cache TTL.
- stampede protection عند cache miss.
- orphan asset cleanup job الفعلي، وهو مذكور معماريًا وغير موجود بوضوح في catalog.

### التفويض والحماية من الإساءة

- JWT claims، algorithm، issuer/audience، access/refresh TTL، signing-key rotation.
- session binding للـlogout ومعنى «current session».
- OAuth state/nonce storage وTTL والاستهلاك الذري.
- rate-limit values/windows/keys/storage/failure mode؛ قائمة endpoints ليست policy.
- quotas للحسابات والكتل/uploads/AI والروابط مع قواعد concurrent enforcement.
- admin authorization للتعليق وAudit Logs، وكلاهما مذكور بلا سطح تنفيذي.

---

## 5. مخاطر نموذج البيانات

1. JSONB مستخدم كبديل عن schema في `settings` و`contact` و`appearance_config` و`content` و`source_snapshot` ونتائج AI، دون CHECK constraints تغلق الأشكال.
2. `Account.status DEFAULT active` لا يمثل مرحلة التسجيل/QR.
3. لا يوجد تمثيل واضح لـprimary authentication provider رغم إلزام «واحد بالضبط» في وثائق الحساب.
4. `OutLink.connected_account_id` موثق في نسخ كـTEXT رغم أن المعرّف UUID؛ العلاقات المنطقية بلا FK تسمح بالدوالّ المعلقة.
5. `AnalyticsEvent` union لا يملك CHECK يفرض حقول profile_view مقابل link_click.
6. click event بلا account_id يفقد الإسناد بعد purge.
7. single-instance blocks تعتمد على read-then-write بلا partial unique index.
8. MAX_BLOCKS يعتمد على count متزامن بلا lock.
9. `sort_order` لا يضمن uniqueness أو contiguity أو tie-breaker في Blocks وConnectedAccounts وداخل SocialIcons.
10. soft-delete للكتل يخالف قاعدة Hard Delete الافتراضية في Data lifecycle ولا توجد purge policy للكتل.
11. ImageAsset orphan cleanup لا يغطي بوضوح حذف/استبدال الكتل؛ «يحذف عند حذف الحساب» ليس cleanup.
12. AnalysisSession JSONB غير معرف بنيويًا، وإعادة الاستخدام لا تُبطل تلقائيًا عند تبديل provider/model إلا إذا غُيّر version يدويًا.
13. لا uniqueness تمنع تحليلي AI pending متطابقين.
14. AccountQRCode الحالي يحتاج unique account_id، وحالة active/asset consistency، ومعاملة S3؛ هذه التفاصيل غير مكتملة.
15. AuditLog مطلوب ولا model له.
16. UsernameReservation منفصل عن Account uniqueness؛ قاعدة البيانات لا تستطيع منع اسم في الجدولين دون locking أو exclusion strategy.
17. AuthenticationIdentity uniqueness والتطبيع عبر email/provider subject يحتاج قواعد case/canonicalization وهجرة عند تغير email لدى provider.
18. retained Account بعد الحذف يحتفظ settings وQR metadata ما لم تُحدد redaction دقيقة.

---

## 6. مخاطر الخلفية

- **OTP:** إرسال البريد قبل إنشاء السجل يخلق OTP وصل للمستخدم ولا يمكن التحقق منه إذا فشل insert. عكس الترتيب يخلق سجلًا لبريد لم يصل. يلزم state/attempt أو transactional outbox، لا مجرد اختيار ترتيب.
- **استهلاك الرموز:** verification_token وgoogle_token موصوفان single-use بلا مخزن replay أو claim ذري.
- **Refresh rotation:** طلبان متزامنان للرمز نفسه يحتاجان compare-and-swap وعقوبة reuse؛ غير معرف.
- **Google OAuth:** لا state/nonce/PKCE/callback contract نهائي ولا mapping لأخطاء provider.
- **إنشاء الحساب:** ملكية إنشاء AuthenticationIdentity متداخلة.
- **الحذف:** فجوة commit/event، لا حالة progress، لا recovery endpoint، لا reconciliation.
- **التعليق الإداري:** السلوك موجود بلا admin API/policy/audit.
- **الكتل:** add/update/delete تنشئ آثار OutLink عابرة للمجال بلا transaction.
- **الحسابات المتصلة:** الحذف يعدل عدة Block JSONB ويؤرشف روابط ويحذف السجل؛ لا rollback.
- **إعادة الترتيب:** لا تعريف للائحة ناقصة/مكررة/تحتوي IDs محذوفة، ولا حماية من تعديل متزامن.
- **الرفع:** لا multipart field name، ولا streaming/buffering rule، ولا cleanup عند قطع الاتصال.
- **OutLink:** public_id collision retry غير محدد، و`created` state لا يملك انتقالًا تنفيذيًا واضحًا.
- **Analytics:** timezone وinclusive/exclusive وCTR عند صفر وrounding وretention duration غير محددة.
- **Rendering:** Nest يجمع البيانات، Next ينتج HTML؛ عقد النقل بينهما غير موجود، ومع ذلك كلاهما يوصف بأنه renderer.
- **SEO/GTM:** حقن metadata/script داخل طبقة HTML غير محددة الملكية بين Next وNest.
- **AI:** provider adapter وstructured output validation وprompt/version migration والتزامن غير معرفة.
- **Jobs:** القيم كلها تقريبًا أمثلة؛ لا DLQ ولا alert threshold ولا replay tooling.

---

## 7. مخاطر الواجهة الأمامية

لا يوجد عقد تنفيذ واجهة أمامية يوازي تفصيل الخلفية. مجلد `apps/web` مجرد شكل مستقبلي، والخطة لا تعرّف:

- خريطة routes للـsignup/login/callback/onboarding/dashboard/editor/settings/analytics/AI/QR.
- state machine للتسجيل: reservation expiry أثناء OAuth، إعادة الحجز، email موجود، Google موجود، handoff token منتهي.
- سلوك editor عند save فوري بلا draft: optimistic أو pessimistic، rollback UI، offline/retry، تعارض تعديلات tabs.
- upload progress/cancel/retry، ومعالجة أصل رُفع ولم تُنشأ كتلته.
- block reorder accessibility واللمس والkeyboard، وحالة reorder جزئية.
- اختيار Social Icons مقابل قائمة Connected Accounts والترتيب المزدوج.
- عرض أخطاء field-level بواسطة codes ثابتة؛ المواصفات لا تقدم codes كافية.
- شكل analytics charts/date range/timezone/empty states/archived links.
- QR download format، filename، preview، regeneration failure.
- AI pending/polling/error/reuse/accept flows.
- loading/skeleton/not-found/suspended/deleted states للصفحة العامة.
- استراتيجية auth token storage وتجديده بين Server Components وClient Components.
- contract يمنع تسريب GTM أو بيانات خاصة في prefetch/cache.
- frontend test matrix beyond أربع رحلات عامة في الخطة.

التنفيذ الأمامي سيضطر إلى اختراع معظم سلوك المنتج، وهذا ليس «تفصيل UI»؛ إنه منطق lifecycle وتكامل.

---

## 8. مخاطر التكامل الشامل

1. Next route `/{username}` وNest `RenderingService` بلا بروتوكول داخلي موثق.
2. تحديث Profile يكتب DB ثم يبطل Redis، لكن edge يبقى قديمًا.
3. إنشاء Button يكتب Block ثم OutLink؛ الصفحة قد تقرأ بينهما وتحذف الزر من output.
4. حذف ConnectedAccount يغير JSONB في كتل متعددة، ويرابط OutLinks، ثم يحذف؛ أي failure ينتج UI/DB غير متوافق.
5. Image upload ينتج ImageAsset مستقلًا قبل ربطه بكتلة؛ لا cleanup lifetime.
6. AI يأخذ snapshot من profile/blocks/analytics؛ لا consistency snapshot أو transaction isolation بين هذه القراءات.
7. Analytics click يتأخر عن redirect؛ dashboard لا يملك freshness/SLA.
8. Account deletion لا يبطل edge cache مباشرة.
9. `sameAs` يقرأ كل ConnectedAccounts وليس الاختيار المرئي للكتل.
10. GTM setting يغير HTML العام، لكنه account-settings mutation لا يملك contract متين لإبطال طبقتي cache.
11. QR public route غير موجه في Nginx plan.
12. EventEmitter مع أكثر من API instance لا يضمن تنفيذ handler حيث تتوفر dependencies/worker enqueue.
13. نسخ endpoint في خطة التنفيذ تختلف عن API canon: `/profile/blocks/reorder` مقابل `/profile/blocks/order`، و`/account/connected-accounts` مقابل `/api/v1/connected-accounts`.
14. service names تختلف: `removeBlock` في الخطة مقابل `deleteBlock` في contract، وأسماء analytics summaries مختلفة.

---

## 9. مخاطر الأمن

- **حرج:** GTM لكل مستخدم يمنح كود طرف ثالث قدرة داخل صفحة Minime؛ لا عزل.
- **حرج:** جلسات الحساب المحذوف قد تبقى بسبب فقد EventEmitter.
- **عالٍ:** single-use handoff tokens بلا replay store/atomic consume.
- **عالٍ:** JWT crypto/lifetimes/key rotation غير محددة.
- **عالٍ:** rate limiting بلا أرقام أو window أو fail-open/fail-closed.
- **عالٍ:** logout لا يحدد كيف يُستنتج session_id من access token.
- **عالٍ:** unlink provider لا يحدد أثره على الجلسات المنشأة به.
- **عالٍ:** URL destinations تسمح open redirect عامًا؛ لا abuse policy أو domain blocking أو phishing response.
- **عالٍ:** uploads لا تعرف content sniffing، decompression bombs، SVG sanitization، antivirus، أو image library sandbox.
- **عالٍ:** public asset URL policy لا تميز public/private ولا signed/static.
- **متوسط:** username substring blocklist يسبب false positives كبيرة ولا يملك version/rollout/appeal operational flow.
- **متوسط:** OAuth provider_profile وemail handling يحتاجان minimization/redaction/audit policy.
- **متوسط:** logs مطلوبة للأحداث الأمنية بلا schema/redaction/retention/access.
- **متوسط:** CORS وheaders قائمة فقط؛ CSP غائبة رغم GTM.
- **متوسط:** 404 caching قد يحجب حسابًا أُنشئ حديثًا؛ وقد يسهّل اختلاف timing كشف حالات ما لم تُوحّد المسارات.

---

## 10. عبارات تبدو واضحة لكنها ليست كذلك

| العبارة | لماذا ليست تنفيذية | إعادة الصياغة المطلوبة كقاعدة تنفيذ |
|---|---|---|
| «الحذف فوري ونهائي» | البيانات تمسح async وقد تضيع مهمة البدء. | لا تعاد استجابة نجاح حتى تُثبت حالة deletion_requested وoutbox job ذريًا؛ كل auth check يرفضها فورًا؛ terminal state يُراقب. |
| «الجلسات تبطل لحظة التأكيد» | handler غير متين. | UPDATE sessions وAccount في معاملة واحدة، أو deny كل token عند قراءة account status مع cache آمن. |
| «كل redirect = event واحد» | retry يكرر الحدث. | job_id/event_id فريد وقيد DB؛ إعادة الإدراج no-op. |
| «آخر كتابة تفوز» | لا يحمي invariants ولا يخبر العميل بفقد تعديله. | حدد row version، isolation، وما العمليات المسموح أن تكون LWW وما القيود التي يحميها DB. |
| «التحديث خلال ≤60 ثانية» | طبقتان 60 ثانية. | `edge_age + app_age <= 60` مع headers/TTL محسوبة وpurge للحالات الأمنية. |
| «Rendering stateless» | Redis cache وrender output موجودان. | عملية التحويل لا تكتب canonical data؛ cache مشتق ذو TTL ومفتاح/version محددين. |
| «URL صالح» | scheme/host/Unicode/control chars غير مكتملة. | parser واحد؛ allowlist؛ canonical serialization؛ رفض credentials/control chars؛ limits رقمية. |
| «invalid block is skipped» | يخفي فساد بيانات ويختلف عن رفض persistence. | writes ترفض schema-invalid؛ runtime skips فقط stale/missing references ويسجل code محددًا. |
| «AI failure returns empty array» | empty قد يعني لا اقتراحات أو outage أو quota. | response union يميز success-empty/disabled/rate_limited/provider_error مع UX contract. |
| «GTM ID فقط، فلا كود عشوائي» | الحاوية نفسها تحمل كودًا. | GTM user containers غير مسموحة على origin الرئيسي، أو تعمل في sandbox/origin معزول وفق CSP. |
| «الأصل القديم يصبح orphan» | من يكتشفه ومتى؟ | بعد DB swap تُدرج cleanup job ذريًا؛ scan مصالحة دوري؛ retention وretry محددان. |
| «one active OutLink» | لا قيد يمنعه. | partial unique index على clickable identity حين status=active. |
| «format validation» | وثيقة أخرى تطلب existence. | لا I/O خارجي؛ قبول/رفض حتمي من registry version محدد فقط. |

---

## 11. مخاطر التبسيط المفرط

- **«EventEmitter يكفي V1»** يخفي فجوة durability في أكثر مسار أمني حساسية.
- **«Last Write Wins»** يخفي lost updates والقيود المتزامنة؛ البساطة هنا تُرحّل الخطأ للمستخدم.
- **«JSONB يبقي النموذج مرنًا»** يخفي عدم وجود migration/schema/query contracts.
- **«لا media library»** لا يلغي الحاجة إلى orphan management وupload compensation.
- **«لا publish workflow»** لا يلغي الحاجة إلى atomic edits، preview داخل المحرر، أو cache invalidation.
- **«analytics بسيطة»** لا يلغي idempotency، attribution، timezone، retention، ودقة العد.
- **«AI اختياري»** لا يلغي API/state/schema/security عندما تكون AnalysisSession جزءًا من V1.
- **«QR واحد»** يخفي معاملة DB+S3 والتوليد والتجديد والتوجيه العام.
- **«GTM اختياري»** لا يقلل أثره الأمني عندما يُفعّل.
- **«repository per domain»** لا يحل العمليات العابرة للمجالات؛ بل يجعل المعاملة أصعب إذا لم يوجد coordinator.
- **«worker retries»** لا يعني idempotency؛ يجب أن يكون لها مفتاح وقيد وحالة.
- **«الـfrontend يعيد استخدام Zod»** لا يحدد UX state machines أو server/client auth.

---

## 12. نتائج ملفًا بملف، مجمعة حسب المجلد

### ملفات الجذر والإعداد

- `.claude/settings.local.json` والنسخة تحت `Architecture-Product-Definition/.claude/`: لا تعرفان عقد منتج، لكنهما تؤكدان وجود حالة محلية داخل المستودع؛ لا عائق مباشر.
- `.gitignore`: واضح كملف منصة توثيقية، لكنه لا يعكس monorepo التنفيذ الموعود.
- `FULL_REPOSITORY_EXPORT.txt` التاريخي: سجل مهم لتتبع الانحراف؛ يثبت أن نماذج OAuth وQR وAI وGTM والخدمات تغيرت دون migration/decision log موحد.
- `social.accounts.oauth.boundary.v1.md` التاريخي المحذوف: يفصل OAuth عن Social Accounts ويجعل التخزين/الملكية بصياغة قديمة؛ حذفه لم يُرفق بخريطة استبدال صريحة.

### `account/` — سبعة ملفات

- `account.model.specification.v1.md`: جيد في تثبيت Account root؛ غير واضح في primary provider وsettings JSON، ويتقاطع مع GTM وQR الحالية.
- `authentication.policy.v1.md`: يحدد OTP جيدًا؛ يترك block thresholds وtoken contracts للتكوين بلا قيم.
- `authentication.provider.system.specification.v1.md`: يضيف Google/Apple/provider lifecycle، لكنه يوسع الواقع عن بعض ملفات التنفيذ ويحتاج callback DTOs وstate/nonce/storage.
- `minime.account.claim.system.specification.v1.md`: يثبت username-first، لكنه يتعارض مع خريطة رحلة تقول create/verify ثم choose username.
- `username.policy.v1.md` و`username.reserved.and.blocked.lists.v1.md`: قواعد قابلة للبرمجة نسبيًا، لكن إدارة القائمة substring والتحديث التشغيلي غير معرفة.
- `account.deletion.policy.v1.md`: يعرّف النتيجة جيدًا؛ لا يعرّف آلية تضمنها، ويعتمد على Audit Logs غير منفذة.

### `account-management/` — ثلاثة ملفات

- `connected.accounts.specification.v1.md`: نموذج خفيف جيد؛ يتعارض في فحص الوجود ومعالجة المراجع عند الحذف.
- `minime.account.management.system.specification.v1.md`: يجمع مسؤوليات كثيرة ويحتوي شرط «cannot be found» المناقض.
- `audit.logging.policy.v1.md`: يعرّف الحاجة والاحتفاظ، لكنه بلا أي تنفيذ مقابل.

### `analytics/` — ثمانية ملفات

تعريف metrics والخصوصية متسق عمومًا. العوائق المشتركة: click attribution بلا account_id، CTR عند صفر، timezone وحدود الفترات، rounding، رابط archived/purged في التقارير، واعتماد كامل على أحداث يمكن فقدها أو تكرارها. `analytics.reporting.policy` يريد archived links في التقرير، لكن management API لا يسردها بوضوح.

### `appearance/` و`appearance/themes/` — تسعة ملفات

الفصل بين content/appearance/rendering موضح جيدًا. غير القابل للتنفيذ: schema الداخلي «implementation-specific»، catalog versions/migrations، fallback «last valid أو default» بلا اختيار، theme retirement، global customizations، والتناقض المباشر حول block overrides.

### `block-styling/` — ستة ملفات

تشرح inheritance/resolution نظريًا، لكنها تفترض property catalog وconstraint evaluator غير معرفين. `Reject Save / Reset / Auto-Correct` خيارات لا سياسة. وجود المجلد كله يتعارض مع تحذير Appearance بعدم بناء هذه الميزة في V1.

### `blocks/` — عشرة ملفات

الأنواع والمضاعفة واضحة. المخاطر: لا أطوال نصوص، لا قيود DB للأنواع المفردة، تناقض Button/OutLink، تناقض Image `image_id` مقابل renderer `url/alt`، وتناقض Title/Textbox `text` مقابل `value`. Social Icons يذكر visibility بلا حقل visibility.

### `docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`

مفيد كخريطة، لكنه ليس مرجع تنفيذ: رحلة التسجيل مرتبة خلاف claim flow، تعريف AI القديم لا يطابق AnalysisSession الحالي، وادعاء ≤60 ثانية لا يطابق طبقتي cache.

### `integrations/` — ثلاثة ملفات

العزل والفشل الاختياري موضحان، لكن الفرضية الأمنية عن GTM خاطئة تقنيًا. النماذج القديمة deployment-wide والحالية per-account تكشف انتقالًا لم يُستكمل عبر كل العقود.

### `out-links/` — ثمانية ملفات

النموذج والـroute واضحان نظريًا. العوائق: `created` بلا path، 302 مقابل أمثلة 307، event بعد «send»، duplicate retries، status/error 404/410، public_id collision، وغياب transaction مع block changes.

### `platform/ai/` — أربعة ملفات

المبادئ تحمي سلطة المستخدم، لكنها تصف knowledge/memory/decisions ثم تنفي ملكية persistence. الحالة الحالية أضافت AnalysisSession من دون تنظيف كل الجمل القديمة أو تقديم API/output schema.

### `platform/data/` — أربعة ملفات

قواعد ownership مفيدة. الخريطة التاريخية تسجل كيانات لم تعد canonical، وسياسة Hard Delete الافتراضية لا تطابق soft-deleted Blocks. retention يتجنب عمدًا المدد، بينما التنفيذ يحتاجها ولا يحدد كثيرًا منها.

### `platform/events/` — أربعة ملفات

تصر على record-before-deliver والحدث الدائم، لكن implementation لا ينفذ ذلك. جملة «business correctness أعلى من event persistence» لا تعفي المسارات التي تعتمد وظيفيًا على الحدث مثل الحذف.

### `platform/storage/` — أربعة ملفات

فصل binary/metadata جيد. `asset.processing.specification.v1.md` مفقود، وسياسة الأنواع متعارضة، وعقد delivery يتجنب authorization وsigned/public decisions التي يحتاجها التنفيذ. orphan detection بلا job/algorithm.

### `platform/` — ملفان

`platform.architecture` و`platform.interaction` يقدمان مبادئ، لكن best-effort cross-platform order وغياب atomicity يتركان Product Domains مطالبة بحل غير موثق. AI ownership matrix لا يطابق AnalysisSession الحالي.

### `profile-content/` — ثلاثة ملفات

`profile.block.reference` يعرض نموذج مرجعي ثم يجعل `Block.sort_order` الحقيقة؛ `profile.content.lifecycle` يسمح باسم فارغ؛ `profile.content.specification` يمنعه. كما ينسب Design Tokens/Visual Styling إلى Rendering في موضع، بينما Appearance يملكها في مواضع أخرى.

### `public-profile/` — سبعة ملفات

pipeline واضح، لكن cache ordering متناقض: cache policy يقول routing دائمًا قبل cache، request lifecycle يسمح cache قبل retrieval دون تحديد access recheck. error states تحصر 200/404/500 بينما security يفرض 429. reserved routes لا تتطابق آليًا مع username blocked list.

### `qr-code/` — ثلاثة ملفات

تعرف هدفًا ثابتًا وQR واحدًا، لكنها لا تحسم معنى deterministic bytes، ولا transaction/persistence/route/cache/download. النسخة الحالية من data model تجعل `AccountQRCode` مستقلًا، ما يتطلب تحديث كل العبارات القديمة التي تصف QR كحقل Account.

### `rendering/` — ثمانية ملفات

الفصل النظري منظم. العقود الحقلية غير متطابقة مع Blocks، وLayout يقول لا يتحكم بموضع block ثم Presentation ينسب له Block Positioning. Rendering يوصف بأنه ينتج HTML وفي الوقت نفسه Next مسؤول عن الواجهة؛ لا interface بينهما. بعض الملفات تقول renderer لا يتحقق، وأخرى تجعله يعيد null عند malformed URL.

### `seo/` — أربعة ملفات

metadata الأساسية قابلة للبناء في `implementation/12`. ملفات architecture/indexing لا تزال تعتمد حالات نشر/خصوصية غير موجودة. Structured Data يسمح Person/Organization/WebSite بلا profile type. fallback strategy غير محددة. `sameAs` يكشف كل الحسابات المتصلة.

### `settings/` — ثلاثة ملفات

تنظيم واجهة فقط. Connected Accounts يذكر linked identities بينما Security يملك authentication methods، وهو تداخل. لا توجد setting schemas أو routes تفصيلية تغطي الفئات.

### `social-accounts/` — تسعة ملفات

المعالجة offline والحفاظ على نية المستخدم واضحان. المشاكل: النص القديم يقول Social Accounts «stores» ثم الحالي يقول handoff transient؛ registry path خطأ؛ المنصات غير متطابقة مع enum؛ Smart Mode مع identifier واحد لا يحدد ماذا يحدث إذا كان صالحًا لمنصة وغير صالح لأخرى؛ handoff delivery بلا transaction/idempotency.

### `social-accounts/social-platforms/` — اثنا عشر ملفًا

كل ملف قابل للتحويل إلى validator منفرد، لكن لا توجد source code registry واحدة أو tests. Spotify يخفض platform-assigned opaque identifier بلا دليل أن case غير مهم. القواعد عرضة لتغير المنصات ولا توجد version migration للسجلات القديمة.

### `implementation/` — أربعة عشر ملفًا

- `README.md`: عبارة «جاهز للتنفيذ» غير صحيحة أمام التعارضات المثبتة.
- `01`: التقنية محددة بالاسم لا بالإصدار؛ EventEmitter يناقض event platform؛ AI القديم يقول لا تخزين بينما الحالي يخزن AnalysisSession.
- `02`: بنية مقترحة وليست repository منفذة؛ لا worker app رغم BullMQ.
- `03`: أكثر الملفات فائدة، لكنه يحمل JSONB غير محكوم، نقص قيود التزامن، ونموذج AI/QR حديث غير متسق مع كل المراجع.
- `04`: عقود خدمات واسعة، لكنها تنتهك قاعدة repository ownership عند العمليات العابرة وتترك المعاملات.
- `05`: يغطي معظم REST، لكنه يفتقد AI، وDTOs غامضة، وpagination غير متسقة، ولا frontend-facing error codes.
- `06`: صادق بشأن فقد الأحداث، لكنه يبني عليه الحذف والإبطال.
- `07`: أوضح validation، لكنه يناقض HEIC/GIF ويترك حدودًا أساسية للتكوين أو بلا قيمة.
- `08`: قائمة أمنية جيدة، لكنها لا تحدد crypto/token/rate policies، ولا تدرك خطر GTM كاملًا.
- `09`: يحدد المفاتيح، لكنه لا يحقق SLA بوجود طبقتين ولا يبطل edge.
- `10`: catalog مفيد، لكنه يقبل duplicate clicks ولا يحدد retry numbers/DLQ، وorphan cleanup غامض.
- `11`: Storage interface مفيد؛ ترتيب replacement خطر؛ AI يناقض نفسه في persistence.
- `12`: SEO عملي نسبيًا؛ GTM per-account خطر، وsameAs لا يحترم اختيار الظهور.
- `13`: ليست خطة Frontend كاملة، وأسماء endpoints/services تنحرف عن العقود، وتنسى worker وroute QR.

---

## 13. درجة جاهزية التنفيذ

**الدرجة الإجمالية: 28/100.**

| المحور | الدرجة | السبب المختصر |
|---|---:|---|
| الخلفية | 34/100 | توجد خدمات ونماذج، لكن المعاملات والمتانة والتزامن غير محسومة. |
| الواجهة الأمامية | 14/100 | لا state machines ولا route/UX/error contracts تنفيذية. |
| البيانات | 38/100 | catalog قوي، لكن JSONB والقيود والإسناد والحذف بها فجوات. |
| الواجهات البرمجية | 31/100 | REST واسع، مع غياب AI وDTO/error/pagination الدقيقة. |
| الأمن | 22/100 | GTM والجلسات والحذف والرموز ومعدلات الحد تهدد الإنتاج. |
| الاتساق | 16/100 | تعارضات مباشرة بين canonical/implementation/history. |
| التشغيل | 20/100 | worker/topology/routing/recovery/migrations غير مكتملة. |

هذه ليست درجة جودة الفكرة؛ إنها درجة قدرة فريق حقيقي على تحويل المستودع إلى إنتاج دون سؤال معماري.

---

## 14. خطة الإصلاح المطلوبة

### المرحلة 1 — يجب إصلاحها قبل كتابة أي كود

1. إعلان مرجع أولوية وحيد مع إزالة/وسم العقود المنسوخة والمتعارضة، لا الاكتفاء بكلمة «Canonical» في عدة ملفات.
2. حسم نموذج التسجيل وQR والمعاملة/التعويض وحالة الحساب.
3. استبدال مسار الحذف المعتمد على EventEmitter بمسار متين.
4. حسم Button ↔ OutLink وتحديد المعاملة والقيد.
5. حسم block styling في V1.
6. حسم AI entity/API/output schema.
7. قرار أمني نهائي بشأن GTM لكل مستخدم.
8. توحيد SocialPlatform registry والـenum.
9. حل display_name الابتدائي.
10. تعريف topology الإنتاج: API/Web/Worker/Nginx/Redis/PostgreSQL/S3.

### المرحلة 2 — يجب إصلاحها قبل تنفيذ الخلفية

1. إضافة قيود DB لكل invariant وunion JSON shape الممكنة.
2. تعريف isolation/locks/retry/idempotency.
3. تعريف outbox/event durability أو تخفيض ادعاءات الحدث رسميًا.
4. استكمال token/OAuth/session/rate-limit contracts.
5. استكمال storage processing والأنواع والحدود وcompensation.
6. استكمال jobs: schedules، attempts، backoff، timeout، DLQ، reconciliation.
7. إصلاح analytics attribution ودقة النقر والفترات الزمنية.
8. تنفيذ أو حذف Audit Logging من التزامات V1.
9. تعريف edge cache invalidation وSLA مشتركة.
10. توحيد أسماء الخدمات والمسارات بين `04` و`05` و`13`.

### المرحلة 3 — يجب إصلاحها قبل تنفيذ الواجهة الأمامية

1. نشر route map وstate machines للتسجيل والمحرر وAI وQR.
2. إصدار DTOs وerror codes مشتركة قابلة للتوليد.
3. تعريف optimistic concurrency وconflict UX.
4. تعريف uploads/reorder/accessibility/empty/error/loading states.
5. تعريف token storage/refresh في Next Server وClient Components.
6. تعريف dashboard aggregation وanalytics timezone/chart contracts.
7. تعريف AI polling/reuse/accept semantics.

### المرحلة 4 — يمكن توضيحها أثناء التنفيذ

1. أسماء classes والملفات الداخلية غير العامة.
2. اختيار مكتبة تحويل الصور بعد تثبيت عقد المعالجة.
3. شكل مكونات UI والتفاصيل البصرية التي لا تغير lifecycle.
4. قيم monitoring dashboards بعد تعريف SLOs الأساسية.
5. تحسينات query/index التي لا تغير العقود.

---

## 15. التحذير النهائي

إلى مالك المشروع: إذا بدأ التنفيذ بهذه الحالة، فلن «يملأ الفريق التفاصيل»؛ سيخترع التفاصيل. مهندس المصادقة سيختار معنىً للتسجيل، ومهندس الملفات سيختار ترتيبًا مختلفًا للاستبدال، ومهندس الواجهة سيخترع حالات المنتج، ومهندس التحليلات سيقبل أرقامًا مكررة، ثم تظهر التناقضات عند الدمج لا عند كتابة الوحدة المنفردة.

النتيجة المتوقعة هي حسابات نصف منشأة، حذف غير مكتمل، جلسات لا تُبطل دائمًا، روابط بلا تتبع أو بتتبع مضاعف، تحليلات لا يمكن إسنادها، ملفات عامة قديمة، وواجهة تتعامل مع أخطاء لم يتفق backend على أسمائها. الأسوأ أن اختبارات كل فريق قد تنجح لأن كل فريق اختبر تفسيره الخاص.

لا تبدأوا التنفيذ الإنتاجي قبل إغلاق معوقات المرحلة الأولى وتوقيع عقود المرحلة الثانية. عبارة «جاهز للتنفيذ» في `implementation/README.md` ليست حالة هندسية؛ هي ادعاء يناقض الأدلة الموجودة في المستودع نفسه.
