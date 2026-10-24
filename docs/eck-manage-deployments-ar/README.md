<div dir="rtl" lang="ar">

# شرح عربي: إدارة النشر على ECK

هذا المجلد شرح **واضح بالعربية** لقسم التوثيق الرسمي:

<span dir="ltr">[Manage deployments in Elastic Cloud on Kubernetes](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/manage-deployments)</span>

الشرح **ليس ترجمة حرفية**؛ الهدف أن تفهم المفاهيم والخطوات. المصطلحات التقنية الإنجليزية تُترك كما هي داخل جمل عربية، والاتجاه من **اليمين لليسار**.

## تنبيه

- المحتوى مبني على توثيق Elastic (حالة 2026). راجع الرابط الرسمي لأي تحديث.
- هذا المجلد **لا يغني** عن تثبيت وتشغيل **ECK Operator** نفسه؛ يفترض أن المشغّل موجود في الكلاستر.

## فهرس الأجزاء

| # | الملف | يغطي |
|---|--------|------|
| 1 | [01-نظرة-عامة.md](./01-نظرة-عامة.md) | معنى القسم، ما هو الـ deployment، خريطة المواضيع |
| 2 | [02-نشر-elasticsearch.md](./02-نشر-elasticsearch.md) | Quickstart، المراقبة، الوصول، الأمان |
| 3 | [03-نشر-kibana.md](./03-نشر-kibana.md) | ربط Kibana بـ Elasticsearch، الدخول من المتصفح |
| 4 | [04-تحديث-النشر.md](./04-تحديث-النشر.md) | تعديل YAML، التوسع، القيود |
| 5 | [05-helm-eck-stack.md](./05-helm-eck-stack.md) | chart `eck-stack`، الترقية، Ingress |
| 6 | [06-الوصول-والخدمات.md](./06-الوصول-والخدمات.md) | Services، LoadBalancer، الأسرار |
| 7 | [07-ضبط-النشر.md](./07-ضبط-النشر.md) | Configure deployments — المواضيع المشتركة |
| 8 | [08-إعدادات-elasticsearch.md](./08-إعدادات-elasticsearch.md) | كل أقسام Elasticsearch configuration |
| 9 | [09-إعدادات-kibana.md](./09-إعدادات-kibana.md) | كل أقسام Kibana configuration |
| 10 | [10-سياسات-stack-config.md](./10-سياسات-stack-config.md) | `StackConfigPolicy` |
| 11 | [11-ترقية-الإصدار.md](./11-ترقية-الإصدار.md) | ترقية Elastic Stack على ECK |
| — | [روابط-رسمية.md](./روابط-رسمية.md) | روابط الصفحات الأصلية |

## كيف تقرأ الكود والأوامر

كتل الأوامر و<span dir="ltr">YAML</span> تُعرض من اليسار لليمين (هذا طبيعي للبرمجة). باقي الشرح عربي من اليمين لليسار.

</div>
