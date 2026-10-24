<div dir="rtl" lang="ar">

# 2 — نشر كلاستر Elasticsearch (Quickstart)

**الرابط الرسمي:**  
<span dir="ltr">https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-deployment-quickstart</span>

## الفكرة في سطر

تطبّق ملفاً يعرّف مورد Kubernetes من نوع <span dir="ltr">Elasticsearch</span>؛ المشغّل ينشئ <span dir="ltr">Pods</span>، تخزين، خدمة HTTP، مستخدمين، وشهادات TLS.

## المثال الأساسي

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 9.5.5
  nodeSets:
  - name: default
    count: 1
    config:
      node.store.allow_mmap: false
```

| الحقل | المعنى |
|--------|--------|
| <span dir="ltr">name: quickstart</span> | اسم الكلاستر داخل الكلاستر؛ يُستخدم في أسماء الخدمات والأسرار |
| <span dir="ltr">version</span> | إصدار Elastic Stack لـ Elasticsearch |
| <span dir="ltr">nodeSets</span> | مجموعات نودز (يمكن فصل الأدوار لاحقاً: master / data …) |
| <span dir="ltr">count: 1</span> | عدد النودز في هذه المجموعة |
| <span dir="ltr">node.store.allow_mmap: false</span> | تعطيل <span dir="ltr">mmap</span> للتخزين — مناسب للتجربة؛ في الإنتاج راجع **Virtual memory** |

## ماذا يحدث بعد `kubectl apply`؟

- المشغّل يخلق الموارد تلقائياً.
- قد يستغرق **دقائق** حتى يصبح الكلاستر جاهزاً.
- في الـ quickstart التخزين الافتراضي حوالي **1 GiB** عبر <span dir="ltr">storage class</span> الافتراضي — غير مناسب للإنتاج بدون تعديل (**Volume claim templates**).

## متطلبات الذاكرة

إذا لم يوجد على أي نود Kubernetes **2 GiB RAM فاضية** على الأقل، الـ <span dir="ltr">Pod</span> يبقى **Pending**. راجع **Manage compute resources** في [07-ضبط-النشر.md](./07-ضبط-النشر.md).

## مراقبة الصحة والتقدم

```bash
kubectl get elasticsearch
```

| العمود | المعنى |
|--------|--------|
| <span dir="ltr">HEALTH</span> | من API صحة الكلاستر في Elasticsearch (<span dir="ltr">green</span> = جيد) |
| <span dir="ltr">PHASE</span> | مرحلة ECK؛ <span dir="ltr">Ready</span> = جاهز من منظور المشغّل |
| <span dir="ltr">NODES</span> | عدد النودز |

متابعة الـ pods:

```bash
kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'
kubectl logs -f quickstart-es-default-0
```

## الوصول للكلاستر

### الخدمة

ECK ينشئ خدمة <span dir="ltr">ClusterIP</span> اسمها عادة:  
<span dir="ltr">quickstart-es-http</span> على المنفذ **9200**.

### المستخدم الافتراضي

- اسم المستخدم: <span dir="ltr">elastic</span>
- كلمة المرور في Secret: <span dir="ltr">quickstart-es-elastic-user</span>

```bash
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
```

### شهادة CA

الاتصال **HTTPS** افتراضياً. شهادة CA في:  
<span dir="ltr">quickstart-es-http-certs-public</span>

```bash
kubectl get secret quickstart-es-http-certs-public -o go-template='{{index .data "ca.crt" | base64decode }}' > quickstart-es-ca.crt
```

### من داخل الكلاستر

استخدم اسم الخدمة الداخلي، مثلاً:  
<span dir="ltr">https://quickstart-es-http.&lt;namespace&gt;.svc:9200</span>

### من جهازك (للتجربة)

```bash
kubectl port-forward service/quickstart-es-http 9200
curl --cacert quickstart-es-ca.crt -u "elastic:$PASSWORD" "https://localhost:9200"
```

ملاحظة: الشهادة قد لا تتطابق مع اسم <span dir="ltr">localhost</span>؛ للتحقق الكامل استخدم <span dir="ltr">--resolve</span> كما في التوثيق، أو في الإنتاج استخدم <span dir="ltr">LoadBalancer</span> / Ingress.

**`-k` / `--insecure`:** للاختبار فقط؛ لا تستخدمها في الإنتاج.

## CRD والمرجع

لتفاصيل حقول المورد:

```bash
kubectl describe crd elasticsearch
```

أو **API Reference** في مستودع <span dir="ltr">cloud-on-k8s</span>.

## الخطوات التالية حسب Elastic

- نشر Kibana → [03-نشر-kibana.md](./03-نشر-kibana.md)
- تطبيق التحديثات → [04-تحديث-النشر.md](./04-تحديث-النشر.md)
- إعدادات متقدمة → [08-إعدادات-elasticsearch.md](./08-إعدادات-elasticsearch.md)

</div>
