<div dir="rtl" lang="ar">

# 5 — إدارة النشر عبر Helm (`eck-stack`)

**الرابط الرسمي:**  
<span dir="ltr">https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/managing-deployments-using-helm-chart</span>

## متى تستخدم Helm؟

- تريد تثبيت **Elasticsearch + Kibana** (ومكونات أخرى) بأمر واحد.
- لديك بيئات متعددة وتمرّر **values** مختلفة.
- فريقك معتاد على <span dir="ltr">helm install / upgrade</span>.

بديله: ملفات YAML مباشرة + <span dir="ltr">kubectl apply</span> (الفصول 2 و 3).

## الإعداد الأولي

```bash
helm repo add elastic https://helm.elastic.co
helm repo update
```

- أقل إصدار Helm مدعوم: **3.2.0**
- من ECK **2.4.0** فصاعداً يتوفر chart <span dir="ltr">eck-stack</span>.

## البنية

- <span dir="ltr">eck-stack</span> يجمع charts فرعية: <span dir="ltr">eck-elasticsearch</span>، <span dir="ltr">eck-kibana</span>، وغيرها.
- أمثلة التوثيق غالباً في namespace: <span dir="ltr">elastic-stack</span> (غيّرها حسب بيئتك).

## تثبيت سريع (مثل quickstart)

```bash
helm install es-kb-quickstart elastic/eck-stack -n elastic-stack --create-namespace
```

## تخصيص متقدم (مثال hot/warm/cold)

```bash
helm install es-quickstart elastic/eck-stack -n elastic-stack --create-namespace \
  --values https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.5/deploy/eck-stack/examples/elasticsearch/hot-warm-cold.yaml \
  --values https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.5/deploy/eck-stack/examples/kibana/http-configuration.yaml
```

## سيناريوهات إضافية (نفس chart)

| السيناريو | ملف values مثال في مستودع ECK |
|-----------|-------------------------------|
| Fleet + Agents | <span dir="ltr">examples/agent/fleet-agents.yaml</span> |
| Logstash + Beats | <span dir="ltr">examples/logstash/basic-eck.yaml</span> |
| APM Server | <span dir="ltr">examples/apm-server/basic.yaml</span> |
| AutoOps | <span dir="ltr">examples/autoops/basic.yaml</span> |
| Package Registry | <span dir="ltr">examples/package-registry/basic-eck.yaml</span> |

<span dir="ltr">Enterprise Search</span> غير متاح في Stack 9.0+؛ للإصدار 8.x راجع توثيق ECK القديم.

## تثبيت مكون واحد فقط

**عبر `eck-stack` مع تعطيل Kibana:**

```bash
helm install es-quickstart elastic/eck-stack -n elastic-stack --create-namespace \
  --set=eck-kibana.enabled=false
```

**أو chart منفصل:**

```bash
helm install es-quickstart elastic/eck-elasticsearch -n elastic-stack --create-namespace
```

## فرق مهم: إصدار Chart vs إصدار المنتج

| | Chart version | Component version |
|---|---------------|-------------------|
| مثال | <span dir="ltr">eck-stack-0.17.0</span> | Elasticsearch <span dir="ltr">9.5.5</span> |
| تحديده | <span dir="ltr">helm install --version</span> | في values: <span dir="ltr">eck-elasticsearch.version</span> |

ترقية chart **قد** ترقّي إصدارات Stack الافتراضية؛ يمكنك تثبيت إصدار المنتج صراحة.

## `helm upgrade`

```bash
helm repo update
helm upgrade es-kb-quickstart elastic/eck-stack -n elastic-stack
```

**مهم:** مرّر نفس القيم التي استخدمتها في التثبيت + التغييرات الجديدة، حتى لا ترجع إعدادات للافتراضي.

### ترقية Stack لإصدار محدد

```bash
helm upgrade es-kb-quickstart elastic/eck-stack -n elastic-stack \
  --set eck-elasticsearch.version=9.5.5 \
  --set eck-kibana.version=9.5.5
```

### مثال: 3 نودز + Kibana على LoadBalancer

```yaml
# custom-values.yaml
eck-elasticsearch:
  nodeSets:
  - name: default
    count: 3
eck-kibana:
  http:
    service:
      spec:
        type: LoadBalancer
```

```bash
helm upgrade es-kb-quickstart elastic/eck-stack -n elastic-stack -f custom-values.yaml
```

## Ingress

- Ingress مفهوم Kubernetes عام؛ ECK يعطي أمثلة لكن **تحتاج خبرة Ingress controller** في بيئتك.
- إن كان Ingress صعباً: استخدم <span dir="ltr">LoadBalancer</span> ([06-الوصول-والخدمات.md](./06-الوصول-والخدمات.md)).

تفعيل Ingress (مختصر):

```bash
helm install es-kb-quickstart elastic/eck-stack -n elastic-stack --create-namespace \
  --set=eck-elasticsearch.ingress.enabled=true \
  --set=eck-elasticsearch.ingress.hosts[0].host=elasticsearch.example.com \
  --set=eck-elasticsearch.ingress.hosts[0].path="/" \
  --set=eck-kibana.ingress.enabled=true \
  --set=eck-kibana.ingress.hosts[0].host=kibana.example.com \
  --set=eck-kibana.ingress.hosts[0].path="/"
```

## عرض القيم والإصدارات

```bash
helm show values elastic/eck-stack
helm repo search elastic/eck-stack --versions
helm list -n elastic-stack
```

</div>
