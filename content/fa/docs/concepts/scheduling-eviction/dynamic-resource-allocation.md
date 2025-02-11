```markdown
---
reviewers:
- klueska
- pohly
title: تخصیص پویای منابع
content_type: concept
weight: 65
---

<!-- overview -->

{{< feature-state feature_gate_name="DynamicResourceAllocation" >}}

تخصیص پویای منابع یک API برای درخواست و به اشتراک گذاری منابع بین پادها و ظروف درون یک پاد است. این یک تعمیم از API حجم‌های ماندگار برای منابع عمومی است. درایورهای منابع شخص ثالث مسئول ردیابی و تخصیص منابع هستند و با پشتیبانی اضافی از Kubernetes از طریق _پارامترهای ساختاری_ (معرفی شده در Kubernetes 1.30) انجام می‌شود. زمانی که یک درایور از پارامترهای ساختاری استفاده می‌کند، Kubernetes بدون نیاز به ارتباط با درایور، برنامه‌ریزی و تخصیص منابع را انجام می‌دهد. انواع مختلف منابع از پارامترهای خود دلخواه برای تعیین نیازها و مقداردهی اولیه پشتیبانی می‌کنند.

## {{% heading "prerequisites" %}}

Kubernetes v{{< skew currentVersion >}} شامل پشتیبانی API سطح خوشه برای تخصیص پویای منابع است، اما [باید به صورت
صریح فعال شود](#enabling-dynamic-resource-allocation). همچنین باید یک درایور منابع برای منابع خاصی که قرار است با استفاده از این API مدیریت شوند، نصب کنید. اگر شما از Kubernetes v{{< skew currentVersion >}} استفاده نمی‌کنید، مستندات آن نسخه Kubernetes را بررسی کنید.

<!-- body -->

## API

گروه `resource.k8s.io/v1alpha2` {{< glossary_tooltip text="گروه API"
term_id="api-group" >}} این انواع را فراهم می‌کند:

ResourceClass
: تعیین می‌کند که کدام درایور منابع به مدیریت یک نوع خاص منابع می‌پردازد و پارامترهای مشترک برای آن فراهم می‌کند. ResourceClasses توسط یک مدیر خوشه ایجاد می‌شوند هنگام نصب یک درایور منابع.

ResourceClaim
: تعریف یک نمونه خاص از منابع که توسط یک بارگذاری مورد نیاز است. توسط یک کاربر (مدیریت چرخه به صورت دستی، قابل به اشتراک‌گذاری بین پادهای مختلف) یا برای پادهای فردی توسط سیستم کنترلی بر اساس یک ResourceClaimTemplate (چرخه عمر خودکار، معمولاً توسط یک پاد استفاده می‌شود).

ResourceClaimTemplate
: تعریف مشخصات و متادیتا برای ایجاد ResourceClaims. توسط یک کاربر هنگام نصب یک بارگذاری ایجاد می‌شود.

PodSchedulingContext
: به طور داخلی توسط سیستم کنترلی و درایورهای منابع برای هماهنگی برنامه‌ریزی پاد استفاده می‌شود زمانی که نیاز به اختصاص ResourceClaims برای یک پاد وجود دارد.

ResourceSlice
: با استفاده از پارامترهای ساختاری برای انتشار اطلاعات درباره منابعی که در خوشه در دسترس هستند.

ResourceClaimParameters
: شامل پارامترهایی برای یک ResourceClaim که برنامه‌ریزی را تحت تأثیر قرار می‌دهد، در یک فرمتی که توسط Kubernetes درک می‌شود (مدل پارامتر ساختاری). پارامترهای اضافی ممکن است در یک توسعه ماتم برای استفاده توسط درایور تهیه شود.

ResourceClassParameters
: مشابه با ResourceClaimParameters، ResourceClassParameters یک نوع برای پارامترهای ResourceClass فراهم می‌کند که توسط Kubernetes درک می‌شود.

پارامترهای ResourceClass و ResourceClaim در اشیاء جداگانه ذخیره می‌شوند، معمولاً با استفاده از نوع تعریف منابع سفارشی که هنگام نصب یک درایور منابع ایجاد شده است.

توسعه‌دهنده یک درایور منابع تصمیم می‌گیرد که آیا می‌خواهد این پارامترها را در کنترل‌کننده خارجی خود مدیریت کند یا به جای آن بر انجام توسط Kubernetes تکیه کند. یک کنترل‌کننده سفارشی انعطاف بیشتری را فراهم می‌کند، اما اتوماسیون مقیاس‌پذیری خوشه برای منابع محلی گره به طور قابل اعتماد کار نمی‌کند. پارامترهای ساختاری اجازه می‌دهند مقیاس‌پذیری خوشه را فعال کنند، اما ممکن است تمام مورد استفاده‌ها را پاسخ ندهند.

زمانی که یک درایور از پارامترهای ساختاری استفاده می‌کند، هنوز هم امکان دارد تا به کاربران اجازه دهد پارامترهای خاص با CRDهای وندور-مخصوص مشخص کنند. در این صورت، درایور باید این پارامترهای سفارشی را به انواع درخت ترجمه کند. به عنوان جایگزین، یک درایور می‌تواند نیز مستند کند که چگونه برای استفاده از انواع درخت مستقیم استفاده شود.

مشخصات `PodSpec` از `core/v1` منابع

ی را تعریف می‌کند که برای یک پاد در `resourceClaims` نیاز است. موارد در آن فهرست اشاره به یک ResourceClaim یا ResourceClaimTemplate می‌کنند. هنگامی که به یک ResourceClaim اشاره شود، همه پادهایی که از این PodSpec استفاده می‌کنند (به عنوان مثال، داخل یک Deployment یا StatefulSet) همه نمونه‌های ResourceClaim را به اشتراک می‌گذارند. هنگامی که به یک ResourceClaimTemplate اشاره شود، هر پاد نمونه خود را دریافت می‌کند.

لیست `resources.claims` برای منابع ظروف تعریف می‌کند که یک ظرف به منابع این نمونه دسترسی دارد، که این امکان را فراهم می‌کند که منابع را بین یک یا چند ظرف به اشتراک بگذارید.

اینجا یک مثال برای یک درایور منابع خیالی است. دو شیء ResourceClaim برای این پاد ایجاد خواهد شد و هر کدام از ظروف به یکی از آنها دسترسی خواهد داشت.

```yaml
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceClass
name: resource.example.com
driverName: resource-driver.example.com
---
apiVersion: cats.resource.example.com/v1
kind: ClaimParameters
name: large-black-cat-claim-parameters
spec:
  color: black
  size: large
---
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceClaimTemplate
metadata:
  name: large-black-cat-claim-template
spec:
  spec:
    resourceClassName: resource.example.com
    parametersRef:
      apiGroup: cats.resource.example.com
      kind: ClaimParameters
      name: large-black-cat-claim-parameters
–--
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-cats
spec:
  containers:
  - name: container0
    image: ubuntu:20.04
    command: ["sleep", "9999"]
    resources:
      claims:
      - name: cat-0
  - name: container1
    image: ubuntu:20.04
    command: ["sleep", "9999"]
    resources:
      claims:
      - name: cat-1
  resourceClaims:
  - name: cat-0
    source:
      resourceClaimTemplateName: large-black-cat-claim-template
  - name: cat-1
    source:
      resourceClaimTemplateName: large-black-cat-claim-template
```

## زمان‌بندی

### بدون پارامترهای ساختاری

بر خلاف منابع نیتیو (مانند CPU، RAM) و منابع گسترده (مدیریت شده توسط یک پلاگین دستگاه، تبلیغ شده توسط kubelet)، بدون پارامترهای ساختاری، برنامه‌ریز ندارد که منابع پویایی در خوشه موجود هستند یا چگونه آن‌ها را می‌توان به‌طور مناسب تقسیم کرد تا نیازهای یک ResourceClaim خاص را برآورده کند. درایورهای منابع مسئول آن هستند. آن‌ها ResourceClaims را با علامت "تخصیص داده شده" مشخص می‌کنند هنگامی که منابع برای آن رزرو شده باشد. این همچنین به برنامه‌ریز می‌گوید که ResourceClaim در خوشه کجا در دسترس است.

ResourceClaims ممکن است همان لحظه که ایجاد می‌شوند، تخصیص داده شوند ("تخصیص فوری") بدون در نظر گرفتن آن که کدام پادها از آن استفاده می‌کنند. پیش‌فرض این است که تخصیص را تا زمانی که یک پاد برای استفاده از ResourceClaim برنامه‌ریزی شود به تأخیر می‌اندازد (به عبارت دیگر "منتظر اولین مصرف‌کننده").

در این حالت، برنامه‌ریز تمام ResourceClaims مورد نیاز یک پاد را بررسی می‌کند و یک شیء PodScheduling ایجاد می‌کند که در آن به درایورهای منابع که مسئول آن ResourceClaims هستند، در مورد گره‌هایی که برنامه‌ریز مناسب برای پاد در نظر گرفته است اطلاع می‌دهد. درایورهای منابع با حذف گره‌هایی که منابع کافی از منابع خود را باقی نگه داشته نمی‌کنند، پاسخ می‌دهند. به محض اینکه برنامه‌ریز این اطلاعات را دریافت کند، یک گره را انتخاب می‌کند و این انتخاب را در شیء PodScheduling ذخیره می‌کند. سپس درایورهای منابع ResourceClaims خود را به گونه‌ای تخصیص می‌دهند که منابع بر روی آن گره در دسترس باشند. پس از انجام این فرآیند، پاد برنامه‌ریزی می‌شود.

در این فرآیند، ResourceClaims همچنین برای پاد رزرو می‌شوند. در حال حاضر ResourceClaims می‌توانند تنها توسط یک پاد منحصر به فرد یا تعداد نامحدودی از پادها استفاده شوند.

یک ویژگی کلیدی این است که پادها تا زمانی که منابع آن‌ها تخصیص داده و رزرو نشوند، به یک گره تخصیص داده نمی‌شوند. این از وضعیت جلوگیری می‌کند که یک پاد به یک گره تخصیص داده شود و سپس نتواند در آنجا اجرا شود که بد است زیرا یک پاد در انتظار نیز سایر منابع مانند RAM یا CPU را که برای آن تعیین شده بود مسدود می‌کند.

{{< note >}}

برنامه‌ریزی پادهایی که از ResourceClaims استفاده می‌کنند به دلیل ارتباطات اضافی که لازم است، کندتر خواهد بود. مراقب باشید که این ممکن است بر پادهایی که از ResourceClaims استفاده نمی‌کنند نیز تأثیر بگذارد زیرا فقط یک پاد در هر زمان برنامه‌ریزی می‌شود، فراخوانی‌های API برای مدیریت یک پاد با ResourceClaims انجام می‌شود و بنابراین برنامه‌ریزی پاد بعدی به تأخیر می‌افتد.

{{< /note >}}

### با پارامترهای ساختاری

زمانی که یک درایور از پارامترهای ساختاری استفاده می‌کند، برنامه‌ریز مسئول تخصیص منابع به یک ResourceClaim هر زمانی که یک پاد به آن‌ها نیاز داشته باشد، می‌شود. این کار را با بازیابی لیست کاملی از منابع موجود از اشیاء ResourceSlice، پیگیری کردن آن‌هایی که قبلاً به ResourceClaims موجود تخصیص داده شده‌اند و سپس انتخاب از آن منابعی که باقی‌مانده‌اند، انجام می‌دهد. منابع دقیق که انتخاب می‌شوند، مشمول محدودیت‌های ارائه شده در هر ResourceClaimParameters یا ResourceClassParameters مرتبط با ResourceClaim هستند.

منبع انتخاب شده در وضعیت ResourceClaim ثبت می‌شود به همراه هر پارامتر ویژه تامین‌کننده، بنابراین زمانی که یک پاد آماده شروع به اجرا روی یک گره می‌شود، درایور منبع در گره اطلاعات لازم را برای آماده‌سازی منبع دارد.

با استفاده از پارامترهای ساختاری، برنامه‌ریز بدون ارتباط با هیچ درایور منابع DRA قادر به اتخاذ تصمیم است. همچنین قادر است با نگه‌داشتن اطلا

عات درباره تخصیص‌های ResourceClaim در حافظه و نوشتن این اطلاعات به اشیاء ResourceClaim به زمینه در حالی که به طور همزمان پیوستگی پاد به یک گره انجام می‌شود، به سرعت چندین پاد را برنامه‌ریزی کند.

## نظارت بر منابع

kubelet خدمات gRPC را فراهم می‌کند تا کشف منابع پویایی پادهای در حال اجرا را فعال کند. برای اطلاعات بیشتر در مورد نقاط پایانی gRPC، [گزارش تخصیص منابع](/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/#monitoring-device-plugin-resources) را ببینید.

## پادهای پیش‌زمانده

وقتی که شما - یا مشتری API دیگری - یک پاد با `spec.nodeName` قبلاً تنظیم شده ایجاد می‌کنید، برنامه‌ریز گذرا می‌شود.
اگر برخی از ResourceClaim مورد نیاز توسط آن پاد هنوز موجود نباشد، برای پردازش پاد ناموفق خواهد شد و به صورت دوره‌ای بازرسی می‌کند زیرا این نیازها ممکن است بعداً تأمین شود.

این موقعیت همچنین می‌تواند پیش آید زمانی که پشتیبانی از تخصیص منابع پویا در زمان برنامه‌ریزی پاد انجام نشده باشد (عدم تطابق نسخه، پیکربندی، دروازه ویژگی و غیره). kube-controller-manager این را تشخیص می‌دهد و سعی می‌کند با ایجاد تخصیص و/یا رزرو ResourceClaim مورد نیاز پاد، پاد قابل اجرا شود.

{{< note >}}

این تنها با درایورهای منابع است که از پارامترهای ساختاری استفاده نمی‌کنند کار می‌کند.

{{< /note >}}

بهتر است از گذراندن برنامه‌ریز خودداری کنید زیرا پادی که به یک گره اختصاص داده شده است، منابع عادی (مانند RAM، CPU) را که در حالت مسدود بودن وقتی که پاد گیر افتاده است، نمی‌توان برای پادهای دیگر استفاده کرد. برای اجرای یک پاد روی یک گره خاص در حالی که همچنان به جریان نرمال برنامه‌ریزی می‌روید، پاد را با یک انتخاب‌کننده گره ایجاد کنید که دقیقاً با گره مورد نظر همخوانی دارد:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-cats
spec:
  nodeSelector:
    kubernetes.io/hostname: name-of-the-intended-node
  ...
```

همچنین ممکن است در زمان پذیرش، پاد ورودی را متغیر کنید، تا فیلد `.spec.nodeName` را حذف و به جای آن از یک انتخاب‌کننده گره استفاده کنید.

## فعال‌سازی تخصیص منابع پویا

تخصیص منابع پویا یک *ویژگی آلفا* است و فقط زمانی فعال می‌شود که دروازه ویژگی `DynamicResourceAllocation` [راهنمای راهنمایی راهنمایی /docs/reference/command-line-tools-reference/feature-gates/] و  `resource.k8s.io/v1alpha2` {{< glossary_tooltip text="API group" term_id="api-group" >}} فعال باشد. برای جزئیات بیشتر در مورد آن، به پارامترهای `--feature-gates` و `--runtime-config` [پارامترهای kube-apiserver](/docs/reference/command-line-tools-reference/kube-apiserver/) مراجعه کنید. kube-scheduler، kube-controller-manager و kubelet نیز به دروازه ویژگی نیاز دارند.

برای بررسی سریع اینکه یک خوشه Kubernetes ویژگی را پشتیبانی می‌کند یا خیر، با استفاده از دستور زیر، اشیاء ResourceClass را لیست کنید:

```shell
kubectl get resourceclasses
```

اگر خوشه شما تخصیص منابع پویا را پشتیبانی می‌کند، پاسخ ممکن است یک لیست از اشیاء ResourceClass یا:

```
No resources found
```

باشد. اگر پشتیبانی نشود، به جای آن این خطا نمایش داده می‌شود:

```
error: the server doesn't have a resource type "resourceclasses"
```

پیکربندی پیش‌فرض kube-scheduler افزونه "DynamicResources" را فعال می‌کند فقط اگر دروازه ویژگی فعال باشد و هنگام استفاده از API پیکربندی v1 باشد. پیکربندی‌های سفارشی ممکن است باید تغییر داده شوند تا آن را شامل کنند.

علاوه بر فعال کردن ویژگی در خوشه، یک درایور منبع هم باید نصب شود. لطفاً به مستندات درایور برای جزئیات مراجعه کنید.

## {{% heading "بعدی چیست" %}}

- برای اطلاعات بیشتر در مورد طراحی، به [KEP تخصیص منابع پویا](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/3063-dynamic-resource-allocation/README.md) و [KEP پارامترهای ساختاری](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/4381-dra-structured-parameters) مراجعه کنید.
