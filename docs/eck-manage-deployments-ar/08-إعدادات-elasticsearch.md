<div dir="rtl" lang="ar">

# 8 — إعدادات Elasticsearch على ECK

**الرابط الرسمي:**  
<span dir="ltr">https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-configuration</span>

قبل الإنتاج: راجع هذا القسم كاملاً. كل فقرة أدناه = قسم في التوثيق؛ الشرح **ماذا يعني ومتى تحتاجه**.

---

## Key concepts — المفاهيم الأساسية

### Nodes orchestration — تنسيق النودز

كيف ECK يضيف/يحذف نودز، يطبّق ترقيات، ويمرّر التغييرات **بالترتيب الصحيح** (rolling updates). مهم عند التوسع وترقية الإصدار؛ يمكن ضبط السلوك (**Update strategy**).

### Storage recommendations — التخزين

اختيار <span dir="ltr">StorageClass</span>، أداء الأقراص، وحجم volumes لبيئات البحث. مرتبط بـ **Volume claim templates**.

---

## Basic Elasticsearch settings — إعدادات أساسية

### Node configuration

محتوى ملف <span dir="ltr">elasticsearch.yml</span> يُمرّر تحت <span dir="ltr">nodeSets[].config</span> — إعدادات النود (أدوار، مسارات، ميزات).

### Volume claim templates

تعريف **أقراص دائمة** لكل نود: الحجم، <span dir="ltr">accessModes</span>، <span dir="ltr">storageClassName</span>. بدونها تفقد البيانات عند إعادة تشغيل Pod في كثير من الإعدادات.

### Virtual memory

Elasticsearch يحتاج <span dir="ltr">vm.max_map_count</span> على العقد. في K8s غالباً **initContainer** يشغّل <span dir="ltr">sysctl</span> (كما في أمثلة الترقية). الإصدارات 8.16+ توصي بقيمة أعلى من 8.15 وأقدم.

### Secure settings

أسرار في **keystore** (مفاتيح S3 للنسخ الاحتياطي، إلخ) عبر Secrets.

### Settings managed by ECK

قائمة إعدادات **لا يجب** تغييرها يدوياً لأن المشغّل يديرها (شبكة، أسماء، شهادات داخلية…). تجاوزها يسبب صراعاً.

### Plugins and custom configuration files

- تثبيت plugins: <span dir="ltr">initContainers</span> أو صورة Docker مخصصة.
- ملفات إضافية: <span dir="ltr">ConfigMap</span> / <span dir="ltr">Secret</span> وربطها بالـ Pod.

---

## Scheduling and lifecycle — الجدولة ودورة الحياة

### Advanced Elasticsearch node scheduling

ربط Elasticsearch بجدولة Kubernetes: <span dir="ltr">nodeSelector</span>، <span dir="ltr">affinity</span>، توزيع النودز على مناطق availability.

### Update strategy

التحكم في **كيف** يُطبَّق التحديث (كم pod يتوقف، الترتيب) لتقليل مخاطر فقدان quorum.

### Pod disruption budget (PDB)

يحدّ من عدد pods Elasticsearch المعطّلة **في نفس الوقت** أثناء صيانة الكلاستر أو drain للنودز.

### Security Context

صلاحيات الحاوية و<strong>capabilities</strong> على Linux — توازن بين الأمان ومتطلبات Elasticsearch.

### Readiness probe

متى يعتبر Kubernetes أن النود **جاهز** لاستقبال traffic. تخصيصه لسيناريوهات خاصة (كلاستر كبير، بطء startup).

### Pod PreStop hook

إجراء قبل إيقاف Pod لتقليل اضطراب الكلاستر (إعادة توزيع shards بسلاسة).

---

## TLS/SSL Certificates

### Secure HTTP communications

شهادات **HTTP** (API 9200): CA افتراضي، أو شهاداتك، أو تكامل **cert-manager**.

### Transport settings

شهادات **التواصل بين نودز Elasticsearch** (transport layer) — منفصل عن HTTP.

---

## Traffic handling — التعامل مع الطلبات

### Requests routing to Elasticsearch nodes

عند عدة <span dir="ltr">nodeSets</span> بأدوار مختلفة (coordinating فقط، ingest، …) يمكن توجيه Service لمجموعة نودز معينة بدل كل النودز.

---

## أقسام أخرى مرتبطة (خارج نفس الصفحة لكن ضمن «إدارة النشر»)

| الموضوع | الغرض |
|---------|--------|
| Users and roles | حسابات وصلاحيات |
| Snapshots | نسخ احتياطي واستعادة |
| Remote clusters | ربط كلاسترات |
| Autoscaling | توسع تلقائي (حيث مدعوم) |
| Stack monitoring | مراقبة الكلاستر عبر ECK |
| Troubleshoot | تشخيص المشاكل |

---

## علاقة هذا الفصل ببقية الشرح

| المهمة | أين |
|--------|-----|
| أول نشر | [02-نشر-elasticsearch.md](./02-نشر-elasticsearch.md) |
| تغيير count أو YAML | [04-تحديث-النشر.md](./04-تحديث-النشر.md) |
| ترقية رقم الإصدار | [11-ترقية-الإصدار.md](./11-ترقية-الإصدار.md) |
| سياسات موحدة لعدة كلاسترات | [10-سياسات-stack-config.md](./10-سياسات-stack-config.md) |

</div>
