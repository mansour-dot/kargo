<div dir="rtl" lang="ar">

# 10 — سياسات Elastic Stack (`StackConfigPolicy`)

**الرابط الرسمي:**  
<span dir="ltr">https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elastic-stack-configuration-policies</span>

## لماذا توجد؟

عندك **عدة** كلاسترات Elasticsearch أو عدة Kibana، تكرار نفس الإعدادات في كل ملف YAML يزعج ويصعّب الحوكمة.

**StackConfigPolicy** = مورد Kubernetes يعرّف إعدادات **مركزية** يطبّقها ECK على كلاسترات/واجهات **محددة** بـ labels.

مثال: كل ما عليه label <span dir="ltr">env: production</span> يأخذ نفس حدود recovery ونفس سياسة snapshots.

## كيف تعمل؟

1. تنشئ مورد <span dir="ltr">StackConfigPolicy</span>.
2. المشغّل يطابق السياسة مع الموارد المستهدفة ويُطبّق الإعدادات باستمرار (**reconcile**).
3. الإعدادات المُدارة بالسياسة **لا تُعدَّل** عبر REST API لـ Elasticsearch — المصدر هو Kubernetes.

## متطلبات وتحذيرات

| نقطة | التفاصيل |
|------|-----------|
| الترخيص | يحتاج **Enterprise** أو trial Enterprise |
| Elasticsearch 8.15.1 / 8.15.2 | مشكلة في role mappings عبر السياسات — تجنب أو ترقّ لـ 8.16+ |
| component templates من السياسة | لا يُشار إليها حالياً من index templates عبر API/Kibana UI |
| حجم الإعدادات | حد ~**1MB** لكل كلاستر/Kibana (قيود حجم Secret) |
| عدد السياسات | أكثر من ~100 سياسة على مورد واحد يبطئ المزامنة — اجمع الإعدادات في سياسات أقل |

## تعدد السياسات والأولوية (ECK 3.3+)

عدة سياسات يمكن أن تستهدف نفس الكلاستر. إذا تعارض إعداد:

- الأعلى **weight** يفوز.
- نفس <span dir="ltr">weight</span> → المشغّل يبلغ **تعارض**.

## الحقول الإلزامية في `spec`

- **name** — معرّف السياسة (في الأمثلة داخل metadata أيضاً).
- قسم **elasticsearch** أو **kibana** (واحد على الأقل) وفيه حقل إعداد واحد على الأقل.

## الحقول الاختيارية

| الحقل | الوظيفة |
|--------|---------|
| <span dir="ltr">weight</span> | أولوية عند التعارض |
| <span dir="ltr">namespace</span> | نطاق تطبيق السياسة على namespaces |
| <span dir="ltr">resourceSelector</span> | اختيار موارد بـ labels؛ بدونها = كل ES/Kibana في النطاق |
| <span dir="ltr">variablesFrom</span> (3.5+) | ConfigMap/Secret لاستبدال <span dir="ltr">${VAR}</span> في المواصفة |

## مثال minimal

```yaml
apiVersion: stackconfigpolicy.k8s.elastic.co/v1alpha1
kind: StackConfigPolicy
metadata:
  name: production-settings-all-clusters
  namespace: elastic-system
spec:
  resourceSelector:
    matchLabels:
      env: production
  elasticsearch:
    clusterSettings:
      indices.recovery.max_bytes_per_sec: "100mb"
```

## ما الذي يمكن ضبطه في Elasticsearch؟ (ملخص الجدول الرسمي)

| حقل السياسة | ماذا يفعل |
|-------------|-----------|
| <span dir="ltr">config</span> | إعدادات <span dir="ltr">elasticsearch.yml</span> |
| <span dir="ltr">clusterSettings</span> | إعدادات ديناميكية على مستوى الكلاستر |
| <span dir="ltr">secureSettings</span> | keystore |
| <span dir="ltr">secretMounts</span> | تركيب Secrets داخل pods (مثلاً ملفات JWKS) |
| <span dir="ltr">snapshotRepositories</span> | مستودعات snapshot |
| <span dir="ltr">snapshotLifecyclePolicies</span> | SLM — جدولة نسخ احتياطي |
| <span dir="ltr">ingestPipelines</span> | خطوط ingest |
| <span dir="ltr">indexLifecyclePolicies</span> | ILM |
| <span dir="ltr">indexTemplates.*</span> | قوالب فهارس ومكونات |
| <span dir="ltr">securityRoleMappings</span> | ربط أدوار بمستخدمين |
| <span dir="ltr">securityRoles</span> (3.5+) | أدوار مخصصة في <span dir="ltr">roles.yml</span> |

### snapshot repositories — ملاحظة ECK

لتجنب تعارض كلاسترات على نفس المسار، ECK يعدّل تلقائياً:

- S3/GCS/Azure: <span dir="ltr">base_path</span> إن لم يُحدد
- FS/HDFS: يضيف معرّف الكلاستر للمسار

### security roles

تُدمج في <span dir="ltr">roles.yml</span> على pods؛ Elasticsearch يعيد تحميل الملف بدون إعادة تشغيل Pod بالضرورة.

## إعدادات Kibana في السياسة

| الحقل | الوظيفة |
|--------|---------|
| <span dir="ltr">config</span> | <span dir="ltr">kibana.yml</span> |
| <span dir="ltr">secureSettings</span> | keystore Kibana |

## أمثلة شائعة في التوثيق

- مصادقة لعدة كلاسترات
- repository + SLM + cluster settings معاً
- role mappings + pipelines + ILM + index templates في سياسة واحدة

انسخ الأمثلة من الصفحة الرسمية وعدّل labels والأسماء.

## متى **لا** تستخدم السياسات؟

- بيئة تطوير بكلاستر واحد — YAML للنشر أبسط.
- بدون ترخيص Enterprise.
- إعدادات يجب أن يغيّرها فريق التطبيق من Kibana UI بحرية (السياسة تثبتها من Kubernetes).

</div>
