<div dir="rtl" lang="ar">

# 9 — إعدادات Kibana على ECK

**الرابط الرسمي:**  
<span dir="ltr">https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/kibana-configuration</span>

---

## Kibana encryption keys — مفاتيح التشفير

في الإنتاج يُفضّل ضبط مفاتيح تشفير Kibana (جلسات، إلخ) حسب متطلباتك.

- ECK **يولّد** المفاتيح تلقائياً.
- يمكن **استبدالها** عبر **Secure settings** إذا أردت قيماً ثابتة أو مدارة مركزياً.

---

## Connect to an Elasticsearch cluster — الربط بـ Elasticsearch

### كلاستر مُدار بـ ECK

الطريقة الافتراضية في quickstart:

```yaml
elasticsearchRef:
  name: quickstart
```

يمكن أيضاً الإشارة لكلاستر في **namespace** آخر (حقول إضافية في المرجع).

### كلاستر غير مُدار بـ ECK

Elasticsearch يعمل خارج المشغّل (مثلاً على VMs). تستخدم **Secret** يحتوي عنواناً وبيانات اعتماد، وتحدد المرجع في مواصفة Kibana — نفس فكرة **Connect to external Elastic resources** في [07](./07-ضبط-النشر.md).

---

## Advanced configuration — إعداد متقدم

### Pod Configuration

مثل Elasticsearch: <span dir="ltr">podTemplate</span> لتعديل الموارد، labels، scheduling، sidecars.

### Kibana Configuration

إعدادات <span dir="ltr">kibana.yml</span> تحت <span dir="ltr">config</span> في مواصفة Kibana — سلوك الواجهة، عناوين، ميزات.

---

## Scaling out — توسيع Kibana

زيادة <span dir="ltr">spec.count</span> لعدة pods Kibana. Kibana **stateless** من ناحية التخزين الرئيسي (البيانات في Elasticsearch)، لكن تحتاج إعدادات جلسة/تشفير متسقة عند أكثر من نسخة.

للتوسع التلقائي حسب الحمل: **Autoscaling stateless applications** (HPA).

---

## Secure settings

أسرار Kibana keystore عبر Kubernetes Secrets — مفاتيح API، encryption keys، إلخ.

---

## HTTP Configuration — الشبكة و TLS

### Load balancer settings and TLS SANs

عند التعريض عبر LoadBalancer أو DNS عام، قد تحتاج إضافة أسماء (**SANs**) في الشهادة حتى يعمل TLS بدون تحذيرات.

### Provide your own certificate

رفع شهادتك بدل CA الافتراضي لـ ECK.

### Disable TLS

ممكن تقنياً للتجربة؛ **غير مناسب للإنتاج**.

---

## Install Kibana plugins

تثبيت plugins عبر صورة مخصصة أو آلية التوثيق (init / custom image) — مشابه لفكرة plugins في Elasticsearch.

---

## Autoscaling stateless applications

استخدام **Horizontal Pod Autoscaler** لـ Kibana (وخدمات stateless أخرى على ECK) بناءً على CPU أو مقاييس مخصصة.

---

## روابط سريعة

| المهمة | الملف |
|--------|--------|
| أول نشر Kibana | [03-نشر-kibana.md](./03-نشر-kibana.md) |
| فتح الواجهة من المتصفح | [03](./03-نشر-kibana.md) + [06](./06-الوصول-والخدمات.md) |
| LoadBalancer / Ingress عبر Helm | [05-helm-eck-stack.md](./05-helm-eck-stack.md) |

</div>
