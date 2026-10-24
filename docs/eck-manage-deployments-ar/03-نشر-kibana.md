<div dir="rtl" lang="ar">

# 3 — نشر Kibana وربطها بـ Elasticsearch

**الرابط الرسمي:**  
<span dir="ltr">https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/kibana-instance-quickstart</span>

## الفكرة

<span dir="ltr">Kibana</span> واجهة ويب للبحث واللوحات وإدارة البيانات. على ECK تُعرّف كمورد منفصل من نوع <span dir="ltr">Kibana</span> و**تُشير** إلى كلاستر Elasticsearch موجود عبر <span dir="ltr">elasticsearchRef</span>.

## المثال

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 9.5.5
  count: 1
  elasticsearchRef:
    name: quickstart
```

| الحقل | المعنى |
|--------|--------|
| <span dir="ltr">version</span> | يجب أن يكون متوافقاً مع إصدار Elasticsearch (عادة نفس الرقم) |
| <span dir="ltr">count</span> | عدد نسخ Kibana (للتوسع الأفقي لاحقاً) |
| <span dir="ltr">elasticsearchRef.name</span> | اسم مورد <span dir="ltr">Elasticsearch</span> الذي نشرته سابقاً |

## المراقبة

```bash
kubectl get kibana
kubectl get pod --selector='kibana.k8s.elastic.co/name=quickstart'
```

- عندما يظهر <span dir="ltr">green</span> في <span dir="ltr">get kibana</span>، Kibana تعتبر **available**.
- إن تعطل التشغيل: راجع <span dir="ltr">kubectl logs</span> لـ pod Kibana.

## الوصول من المتصفح

### الخدمة

```bash
kubectl get service quickstart-kb-http
```

خدمة <span dir="ltr">ClusterIP</span> على المنفذ **5601**.

### Port-forward (تجربة محلية)

```bash
kubectl port-forward service/quickstart-kb-http 5601
```

افتح: <span dir="ltr">https://localhost:5601</span>

- المتصفح سيحذر من الشهادة (CA ذاتية التوقيع) — مقبول للتجربة فقط.
- في الإنتاج: شهادات موثوقة (**HTTP Configuration** في [09-إعدادات-kibana.md](./09-إعدادات-kibana.md)).

### تسجيل الدخول

- المستخدم: <span dir="ltr">elastic</span>
- كلمة المرور من نفس سر Elasticsearch:

```bash
kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```

## CRD

```bash
kubectl describe crd kibana
```

## الخطوات التالية

- تحديث النشر → [04-تحديث-النشر.md](./04-تحديث-النشر.md)
- إعدادات Kibana → [09-إعدادات-kibana.md](./09-إعدادات-kibana.md)

</div>
