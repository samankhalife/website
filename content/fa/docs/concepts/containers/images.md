---
title: تصاویر
weight: 10
reviewers:
- erictune
- thockin
content_type: مفهوم
---

<!-- مرور -->

یک تصویر کانتینر داده‌های دودویی را نمایش می‌دهد که یک برنامه و تمام وابستگی‌های نرم‌افزاری آن را در بر می‌گیرد. تصاویر کانتینر بسته‌های نرم‌افزاری قابل اجرا هستند که به تنهایی قابل اجرا هستند و فرضیات خیلی خوبی درباره محیط اجرایی آن‌ها دارند.

به طور معمول، شما یک تصویر کانتینر از برنامه‌تان ایجاد می‌کنید و آن را به یک ثبتگاه فشار می‌دهید قبل از ارجاع به آن در {{< glossary_tooltip text="پاد" term_id="pod" >}}.

این صفحه مفهوم تصویر کانتینر را ارائه می‌دهد.

{{< note >}}
اگر به دنبال تصاویر کانتینر برای یک انتشار Kubernetes هستید (مانند v{{< skew latestVersion >}}، آخرین انتشار کوچک)، به [دانلود Kubernetes](https://kubernetes.io/releases/download/) مراجعه کنید.
{{< /note >}}

<!-- متن -->

## نام‌های تصویر

تصاویر کانتینر معمولاً نامی مانند `pause`، `example/mycontainer` یا `kube-apiserver` دارند.
تصاویر می‌توانند شامل نام میزبان ثبتگاه باشند؛ به عنوان مثال: `fictional.registry.example/imagename`، و شاید شماره پورت را هم داشته باشند؛ به عنوان مثال: `fictional.registry.example:10443/imagename`.

اگر نام میزبان ثبتگاه را مشخص نکنید، Kubernetes فرض می‌کند که شما منظورتان [ثبتگاه عمومی Docker](https://hub.docker.com/) است.
شما می‌توانید این رفتار را با تنظیم ثبتگاه تصویر پیش‌فرض در پیکربندی [زمان اجرای ظرفیت بالا](/docs/setup/production-environment/container-runtimes/) تغییر دهید.

بعد از قسمت نام تصویر، می‌توانید _برچسب_ یا _هضم_ (به همان روشی که با دستورات مانند `docker` یا `podman` استفاده می‌شود) را اضافه کنید.
برچسب‌ها به شما امکان می‌دهند نسخه‌های مختلف یک سری از تصاویر را شناسایی کنید.
هضم‌ها شناسه‌ی منحصربه‌فردی برای یک نسخه خاص از یک تصویر هستند.
هضم‌ها هش‌های محتوای تصویر هستند و غیرقابل تغییر هستند. برچسب‌ها ممکن است به انتقال به تصاویر مختلف نقل مکان کنند، اما هضم‌ها ثابت هستند.

برچسب‌های تصویر شامل حروف کوچک و بزرگ، ارقام، زیرخطها (`_`)، نقطه‌ها (`.`) و خط‌ها (`-`) هستند.
می‌تواند تا ۱۲۸ نویسه طول داشته باشد. و باید مطابق با الگوی regex زیر باشد:
`[a-zA-Z0-9_][a-zA-Z0-9._-]{0,127}`
می‌توانید بیشتر درباره آن بخوانید و الگوی اعتبارسنجی را در
[مشخصات توزیع OCI](https://github.com/opencontainers/distribution-spec/blob/master/spec.md#workflow-categories)
پیدا کنید.
اگر برچسبی مشخص نکنید، Kubernetes فرض می‌کند که شما منظورتان برچسب `latest` است.

هضم‌های تصویر شامل الگوریتم هش (مانند `sha256`) و مقدار هش هستند. به عنوان مثال:
`sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07`
می‌توانید بیشتر درباره فرمت هضم‌ها در
[مشخصات تصویر OCI](https://github.com/opencontainers/image-spec/blob/master/descriptor.md#digests)
بیابید.

برخی از نمونه‌های نام تصاویر که Kubernetes می‌تواند استفاده کند عبارتند از:

- `busybox` - فقط نام تصویر، بدون برچسب یا هضم. Kubernetes از ثبتگاه عمومی Docker و برچسب latest استفاده می‌کند. (مانند `docker.io/library/busybox:latest`)
- `busybox:1.32.0` - نام تصویر با برچسب. Kubernetes از ثبتگاه عمومی Docker استفاده می‌کند. (مانند `docker.io/library/busybox:1.32.0`)
- `registry.k8s.io/pause:latest` - نام تصویر با یک ثبتگاه سفارشی و برچسب latest.
- `registry.k8s.io/pause:3.5` - نام تصویر با یک ثبتگاه سفارشی و برچسب غیر latest.
- `registry.k8s.io/pause@sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07` - نام تصویر با هضم.
- `registry.k8s.io/pause:3.5@sha256:1ff6c18fbef2045af6b9c16bf034cc421a29027b800e4f9b68ae9b1cb3e9ae07` - نام تصویر با برچسب و هضم. فقط هضم برای کشیدن استفاده می‌شود.

## به‌روزرسانی تصاویر

هنگامی که اولین بار یک {{< glossary_tooltip text="استقرار" term_id="deployment" >}},
{{< glossary_tooltip text="مجموعه‌ای دارای حالت" term_id="statefulset" >}}, پاد، یا سایر
اشیاء که شامل یک الگوی پاد هستند، از پیش‌فرض سیاست جذب همه کانتینرها در آن پاد
به `IfNotPresent` تنظیم می‌شود اگر به صراحت مشخص نشده باشد. این سیاست باعث می‌شود
{{< glossary_tooltip text="kubelet" term_id="kubelet" >}} برای گذر کشیدن از یک
تصویر اقدام نکند اگر در محلی قبلاً وجود داشته باشد.

### سیاست جذب تصویر

`imagePullPolicy` برای یک کانتینر و برچسب تصویر تأثیری بر زمانی دارد
که [kubelet](/docs/reference/command-line-tools-reference/kubelet/) سعی می‌کند (بارگیری) تصویر مشخص را بکشد.

در اینجا لیستی از مقادیری که می‌توانید برای `imagePullPolicy` تنظیم کنید و تأثیرات
این مقادیر را مشاهده می‌کنید:

`IfNotPresent`
: تصویر فقط در صورت عدم وجود محلی جلب می‌شود.

`Always`
: هر بار که kubelet یک کانتینر راه‌اندازی می‌کند، kubelet به ثبت تصویر کانتینر می‌پرسد
تا نام را به یک [هضم](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier)
تصویر تبدیل کند. اگر kubelet تصویر کانتینر با آن هضم دقیق را در محلی کش کرده باشد، kubelet از
تصویر کشیده شده استفاده می‌کند؛ در غیر این صورت، kubelet تصویر را با هضم حل شده جذب می‌کند و از آن تصویر
برای راه‌اندازی کانتینر استفاده می‌کند.

`Never`
: kubelet سعی نمی‌کند تصویر را بکشد. اگر تصویر به هر دلیلی قبلاً در محلی وجود داشته باشد، kubelet
سعی می‌کند کانتینر راه‌اندازی کند؛ در غیر این صورت، راه‌اندازی ناموفق است.
برای اطلاعات بیشتر می‌توانید به [تصاویر پیش‌کشیده](#pre-pulled-images) مراجعه کنید.

در معنی‌شناسی کشش ارائه دهنده تصویر زیرین، حتی `imagePullPolicy: Always` نیز کارآمد است،
تا زمانی که ثبت مورد نظر قابل اعتماد قابل دسترس باشد.
زمان اجرای ظرفیت می‌تواند متوجه شود که لایه‌های تصویر در اینگونه اتچه می‌مانند
تا اینکه دوباره دانلود شوند.

{{< note >}}
باید از استفاده از برچسب `:latest` هنگام استقرار کانتینرها در محیط تولیدی خودداری کنید زیرا
سخت‌تر است که ردیابی کنید کدام نسخه از تصویر در حال اجرا است و سخت‌تر به بازگشت مناسب برمی‌گردد.

به جای اینکه برچسب بی‌معنی مانند `v1.42.0` را مشخص کنید و/یا هضم.
{{< /note >}}

برای اطمینان از اینکه Pod همیشه از نسخه یکسانی از تصویر کانتینر استفاده می‌کند، می‌توانید
هضم تصویر را مشخص کنید؛
`<image-name>:<tag>` را با `<image-name>@<digest>`
(برای مثال، `image@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2`) جایگزین کنید.

در استفاده از برچسب‌های تصویر، اگر ثبتگاه تصویر کد تغییر کند که برچسب برای آن تصویر
را نمایند، شما ممکن است با مخلوطی از پادهایی که کد قدیمی و جدید را اجرا می‌کنند. یک هضم تصویر
به طور منحصربه‌فرد یک نسخه خاص از تصویر را شناسایی می‌کند، بنابراین Kubernetes هر بار
که یک کانتینر با نام و هضم تعیین شده از آن تصویر راه‌اندازی می‌کند، همان کد را اجرا می‌کند.
مشخص کردن یک تصویر با هضم، کد را که شما اجرا می‌کنید ثابت می‌کند تا یک تغییر در ثبتگاه نتواند
به آن مخلوط از نسخه‌ها منجر شود.

وجود [کنترل‌کنندگان پذیرش شخص ثالث](/docs/reference/access-authn-authz/admission-controllers/)
اونها Pods (و پودهای الگو) را تغییر می‌دهد، بنابراین تحمل
بارکنندگی که به تصویر هضم مبتنی بر تصاویر هضم برمی‌گردد.
اگر می‌خواهید مطمئن شوید که همه بار کاری شما
اجرا کنید همان کد است که تغییرات برچسبی در ثبتگاه اتفاق می‌افتد.

#### سیاست پیش‌فرض جلب تصویر {#imagepullpolicy-defaulting}

زمانی که شما (یا یک کنترل‌کننده) یک Pod جدید را به سرور API ارسال می‌کنید، خوشه شما فیلد `imagePullPolicy` را زمانی تنظیم می‌کند که شرایط خاصی برقرار شود:

- اگر فیلد `imagePullPolicy` را حذف کنید و برچسب معینی برای تصویر کانتینر مشخص کنید، `imagePullPolicy` به طور خودکار به `IfNotPresent` تنظیم می‌شود.
- اگر فیلد `imagePullPolicy` را حذف کنید و برچسب تصویر کانتینر `:latest` باشد، `imagePullPolicy` به طور خودکار به `Always` تنظیم می‌شود.
- اگر فیلد `imagePullPolicy` را حذف کنید و برچسب تصویر کانتینر مشخص نباشد، `imagePullPolicy` به طور خودکار به `Always` تنظیم می‌شود.
- اگر فیلد `imagePullPolicy` را حذف کنید و برچسب معینی برای تصویر کانتینر مشخص کنید که `:latest` نباشد، `imagePullPolicy` به طور خودکار به `IfNotPresent` تنظیم می‌شود.

{{< note >}}
مقدار `imagePullPolicy` کانتینر همیشه زمانی که شیء اولیه ایجاد می‌شود، تنظیم می‌شود و اگر برچسب یا هضم تصویر بعداً تغییر کند، به‌روزرسانی نمی‌شود.
به عنوان مثال، اگر یک Deployment با تصویری که برچسب آن `:latest` نیست ایجاد کنید و بعداً تصویر این Deployment را به برچسب `:latest` به‌روز کنید، فیلد `imagePullPolicy` به `Always` تغییر نمی‌کند. شما باید به‌طور دستی سیاست جذب هر شیء را بعد از ایجاد اولیه آن تغییر دهید.
{{< /note >}}

#### جذب تصویر ضروری

اگر می‌خواهید همیشه جذب کنید، می‌توانید یکی از موارد زیر را انجام دهید:

- فیلد `imagePullPolicy` کانتینر را به `Always` تنظیم کنید.
- فیلد `imagePullPolicy` را حذف کنید و `:latest` را به عنوان برچسب تصویر استفاده کنید؛ Kubernetes هنگام ارسال Pod، سیاست را به `Always` تنظیم می‌کند.
- فیلد `imagePullPolicy` را حذف کنید و برچسب تصویر را برای استفاده مشخص نکنید؛ Kubernetes هنگام ارسال Pod، سیاست را به `Always` تنظیم می‌کند.
- کنترل‌کننده پذیرش [AlwaysPullImages](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) را فعال کنید.

### جلب تصویر بازگشت

زمانی که یک kubelet شروع به ایجاد کانتینرها برای یک Pod با استفاده از یک زمان اجرای کانتینر می‌کند، ممکن است کانتینر به دلیل `ImagePullBackOff` در حالت انتظار باشد.

وضعیت `ImagePullBackOff` به این معنی است که یک کانتینر نتوانسته است شروع کند زیرا Kubernetes نتوانسته است تصویر کانتینر را جذب کند (به دلایلی مانند نام تصویر نامعتبر یا جذب از یک ثبتگاه خصوصی بدون `imagePullSecret`). قسمت `BackOff` نشان می‌دهد که Kubernetes تلاش خواهد کرد تا تصویر را جذب کند، با تأخیر افزایشی.

Kubernetes این تأخیر را بین هر تلاش افزایش می‌دهد تا به حد محدود داخلی برسد، که 300 ثانیه (5 دقیقه) است.

### جذب تصویر برای کلاس اجرایی چندگانه

{{< feature-state feature_gate_name="RuntimeClassInImageCriApi" >}}
Kubernetes شامل پشتیبانی alpha برای انجام جذب تصاویر بر اساس RuntimeClass یک Pod است.

اگر شما به [دروازه ویژگی](/docs/reference/command-line-tools-reference/feature-gates/) `RuntimeClassInImageCriApi` را فعال کنید، kubelet تصاویر کانتینر را با استفاده از تاپل (نام تصویر، دستگاه اجرایی انجام) به جای فقط نام تصویر یا هضم مرجع می‌کند. می‌تواند عملکرد خود را بر اساس دستگاه اجرایی انتخابی تنظیم کند. جذب تصاویر بر اساس کلاس اجرایی برای کانتینرهای مبتنی بر VM مانند کانتینرهای windows hyperV مفید خواهد بود.

## جذب تصاویر موازی و سریال

به طور پیش‌فرض، kubelet تصاویر را به صورت سریالی جذب می‌کند. به عبارت دیگر، kubelet فقط یک درخواست جذب تصویر را به خدمات تصویر ارسال می‌کند. درخواست‌های جذب تصویر دیگر باید منتظر اتمام درخواست در حال پردازش باشند.

گره‌ها تصمیمات جذب تصویر را در انزوا می‌گیرند. حتی هنگام استفاده از جذب تصاویر سریالی، دو گره مختلف می‌توانند همان تصویر را به صورت موازی جذب کنند.

اگر می‌خواهید جذب تصاویر موازی را فعال کنید، می‌توانید فیلد `serializeImagePulls` را در [پیکربندی kubelet](/docs/reference/config-api/kubelet-config.v1beta1/) به `false` تنظیم کنید. با تنظیم `serializeImagePulls` به `false`، درخواست‌های جذب تصویر به صورت فوری به خدمات تصویر ارسال می‌شوند و چندین تصویر به صورت همزمان جذب می‌شوند.

هنگام فعال کردن جذب تصاویر موازی، لطفاً مطمئن شوید که خدمات تصویر اجرای کانتینر شما می‌تواند جذب تصاویر موازی را اداره کند.

Kubelet هرگز چندین تصویر را به صورت همزمان به نمایندگی یک Pod جذب نمی‌کند. به عنوان مثال، اگر یک Pod دارای یک کانتینر init و یک کانتینر برنامه باشد، جذب تصاویر برای دو کانتینر به صورت موازی نخواهد بود. با این حال، اگر دو Pod دارای تصاویر مختلف باشند، kubelet تصاویر را به صورت موازی به نمایندگی از دو Pod مختلف جذب می‌کند، هنگام فعال بودن جذب تصاویر موازی.

### حداکثر جذب تصاویر موازی

{{< feature-state for_k8s_version="v1.27" state="alpha" >}}

زمانی که `serializeImagePulls` به `false` تنظیم شود، kubelet به پیش‌فرض هیچ محدودیتی بر روی حداکثر تعداد تصاویری که همزمان جذب می‌شوند ندارد. اگر می‌خواهید تعداد جذب تصاویر موازی را محدود کنید، می‌توانید فیلد `maxParallelImagePulls` را در پیکربندی kubelet تنظیم کنید. با تنظیم `maxParallelImagePulls` به _n_, تنها _n_ تصویر می‌توانند به صورت همزمان جذب شوند و هر جذب تصویر بیشتر از _n_ باید حداقل یک جذب تصویر در حال انجام تکمیل شود.

محدود کردن تعداد جذب تصاویر موازی می‌تواند جلوگیری کند از اینکه جذب تصاویر باعث مصرف زیاد پهنای باند یا ورودی/خروجی دیسک شود، هنگام فعال بودن جذب تصاویر موازی.

می‌توانید `maxParallelImagePulls` را به یک عدد مثبت بزرگتر یا مساوی ۱ تنظیم کنید. اگر `maxParallelImagePulls` را به یک عدد بزرگتر یا مساوی ۲ تنظیم کنید، باید `serializeImagePulls` را به `false` تنظیم کنید. kubelet با تنظیمات نامعتبر `maxParallelImagePulls` فرار می‌دهد.

## تصاویر چندمعماری با نمایه‌های تصویر

همچنین به جای ارائه تصاویر دودویی، یک ثبتگاه خصوصی می‌تواند یک [فهرست تصویر کانتینر](https://github.com/opencontainers/image-spec/blob/master/image-index.md) را ارائه دهد. یک فهرست تصویر می‌تواند به چندین [مانیفست تصویر](https://github.com/opencontainers/image-spec/blob/master/manifest.md) برای نسخه‌های خاص ساختار معماری اشاره کند. ایده این است که شما می‌توانید نامی برای یک تصویر (به عنوان مثال: `pause`, `example/mycontainer`, `kube-apiserver`) داشته باشید و به سیستم‌های مختلف اجازه دهید تصویر دودویی مناسب را برای معماری دستگاهی که در حال استفاده است، بردارد.

به طور کلی، Kubernetes نام‌گذاری تصاویر کانتینر را با پسوند `-$(ARCH)` انجام می‌دهد. برای سازگاری به عقب، لطفاً تصاویر قدیمی را با پسوندهای تولید کنید. ایده این است که به عنوان مثال تصویر `pause` داشته باشید که مانیفست برای همه پوشش‌ها (es) را داشته باشد و بگویید `pause-amd64` که برای تنظیمات قدیمی‌تر یا فایل‌های YAML ای که توابع خودکار تصویرها را با پسوندها روغن کرده‌اند.

## استفاده از ثبتگاه خصوصی

نمایه های خصوصی می‌توانند نیازمند کلیدها به منظور خواندن تصاویر از آنها باشند.  
اعتبارات می‌توانند به چندین شیوه فراهم شوند:

- تنظیم گره‌ها برای احراز هویت به یک ثبتگاه خصوصی
  - تمام Podها می‌توانند هر ثبتگاه خصوصی تنظیم شده را بخوانند
  - نیاز به پیکربندی گره توسط مدیر خوشه
- ارائه دهنده اعتبار Kubelet برای جذب داینامیک اعتبارهای ثبتگاه خصوصی
  - kubelet می‌تواند پیکربندی شود تا از افزونه اجرایی ارائه دهنده اعتبار برای ثبتگاه خصوصی مربوط استفاده کند.
- تصاویر پیش‌کش شده
  - همه Podها می‌توانند از هر تصویری که در یک گره ذخیره شده باشد، استفاده کنند
  - نیاز به دسترسی روت به تمام گره‌ها برای تنظیم
- مشخص کردن ImagePullSecrets در یک Pod
  - تنها Podهایی که از کلیدهای خود استفاده می‌کنند، می‌توانند به ثبتگاه خصوصی دسترسی داشته باشند
- افزایش افزوده‌های مختصات و محلی
### گزینه‌های موجود توضیح داده شده است.

### پیکربندی گره‌ها برای احراز هویت به یک ثبتگاه خصوصی

دستورات خاص برای تنظیم اعتبارها به وابستگی به زمان اجرای کانتینر و ثبتگاه مورد استفاده است. شما باید به مستندات راه حل خود مراجعه کنید تا بهترین اطلاعات را در اختیار داشته باشید.

برای مثال برای پیکربندی یک ثبتگاه تصویر کانتینر خصوصی، به تسک [Pull an Image from a Private Registry](/docs/tasks/configure-pod-container/pull-image-private-registry) مراجعه کنید که از یک ثبتگاه خصوصی در Docker Hub استفاده می‌کند.

### ارائه دهنده اعتبار Kubelet برای جذب تصاویر معتبر {#kubelet-credential-provider}

{{< note >}}
این رویکرد بیشترین سازگاری را در مواقعی که Kubelet نیاز به جذب اعتبارهای ثبتگاه داینامیک دارد، ارائه می‌دهد. اغلب برای ثبتگاه‌های ارائه شده توسط ارائه دهندگان ابر که توکن‌های احراز هویت آنها کوتاه‌مدت هستند، استفاده می‌شود.
{{< /note >}}

می‌توانید Kubelet را به گونه‌ای تنظیم کنید که یک افزونه باینری را فراخوانی کند تا به طور دینامیک اعتبارهای ثبتگاه را برای یک تصویر کانتینر جذب کند. این روش قوی‌ترین و چندمنظوره‌ترین روش برای جذب اعتبارها برای ثبتگاه‌های خصوصی است، اما نیاز به پیکربندی سطح Kubelet برای فعال‌سازی دارد.

برای اطلاعات بیشتر به [پیکربندی یک ارائه دهنده اعتبار تصویر Kubelet](/docs/tasks/administer-cluster/kubelet-credential-provider/) مراجعه کنید.

### تفسیر `config.json` {#config-json}

تفسیر `config.json` بین اجرای اصلی Docker و تفسیر Kubernetes متفاوت است. در Docker، کلیدهای `auths` فقط می‌توانند URLهای ریشه را مشخص کنند، در حالی که Kubernetes به URLهای glob نیز اجازه می‌دهد همچنین به مسیرهای پیش‌مطابق برخورد کند. تنها محدودیت این است که الگوهای glob (`*`) باید شامل نقطه (`.`) برای هر زیردامنه باشند. تعداد زیردامنه‌های تطابقی باید برابر با تعداد الگوهای glob (`*.`) باشد، به عنوان مثال:

- `*.kubernetes.io` *با* `kubernetes.io` تطابق *نخواهد* داشت، اما `abc.kubernetes.io`
- `*.*.kubernetes.io` *با* `abc.kubernetes.io` تطابق *نخواهد* داشت، اما `abc.def.kubernetes.io`
- `prefix.*.io` با `prefix.kubernetes.io` تطابق *خواهد* داشت
- `*-good.kubernetes.io` با `prefix-good.kubernetes.io` تطابق *خواهد* داشت

این بدان معناست که یک `config.json` مانند این معتبر است:

```json
{
    "auths": {
        "my-registry.io/images": { "auth": "…" },
        "*.my-registry.io/images": { "auth": "…" }
    }
}
```

عملیات جذب تصویر حالا برای هر الگوی معتبر شناسه‌ها را به CRI container runtime ارسال می‌کند. برای مثال نام‌های تصویر کانتینر زیر با موفقیت تطابق خواهند داشت:

- `my-registry.io/images`
- `my-registry.io/images/my-image`
- `my-registry.io/images/another-image`
- `sub.my-registry.io/images/my-image`

اما نه:

- `a.sub.my-registry.io/images/my-image`
- `a.b.sub.my-registry.io/images/my-image`

kubelet عملیات جذب تصاویر را به ترتیب برای هر اعتبار یافته انجام می‌دهد. این به این معنی است که ورودی‌های چندگانه در `config.json` برای مسیرهای مختلف نیز امکان‌پذیر است.

```json
{
    "auths": {
        "my-registry.io/images": {
            "auth": "…"
        },
        "my-registry.io/images/subpath": {
            "auth": "…"
        }
    }
}
```

اگر حالا یک کانتینر نام‌های تصویر `my-registry.io/images/subpath/my-image` را مشخص کند، آنگاه kubelet تلاش خواهد کرد تا آنها را از هر منبع احراز هویتی که یکی از آنها شکست می‌خورد، دانلود کند.

### تصاویر پیش‌کش شده

{{< note >}}
این رویکرد مناسب است اگر شما بتوانید پیکربندی گره را کنترل کنید. این به طور قابل اعتماد کار نخواهد کرد اگر ارائه دهنده ابر شما گره‌ها را مدیریت کرده و به طور خودکار آنها را جایگزین کند.
{{< /note >}}

به طور پیش‌فرض، kubelet سعی می‌کند که هر تصویر را از ثبتگاه مشخص شده جذب کند. با این حال، اگر ویژگی `imagePullPolicy` کانتینر به `IfNotPresent` یا `Never` تنظیم شود، آنگاه از تصویر محلی استفاده می‌شود (اولویتاً یا به طور انحصاری، به ترتیب).

اگر می‌خواهید که بر روی تصاویر پ

یش‌کش شده به عنوان جایگزین برای احراز هویت به ثبتگاه خصوصی اعتماد کنید، باید اطمینان حاصل کنید که تمام گره‌ها در خوشه دارای همان تصاویر پیش‌کش شده هستند.

تمامی Podها به همه تصاویر پیش‌کش شده دسترسی خواهند داشت.

### مشخص کردن imagePullSecrets در یک Pod

{{< note >}}
این رویکرد توصیه شده برای اجرای کانتینرها بر اساس تصاویر در ثبتگاه‌های خصوصی است.
{{< /note >}}

Kubernetes پشتیبانی از مشخص کردن کلیدهای ثبت تصویر کانتینر در یک Pod را دارد. تمامی Secretهای مرجع باید از نوع `kubernetes.io/dockercfg` یا `kubernetes.io/dockerconfigjson` باشند.
### ایجاد یک Secret با یک پیکربندی Docker

شما باید نام کاربری، رمز عبور ثبتگاه و آدرس ایمیل مشتری را برای احراز هویت به ثبتگاه، همچنین نام میزبان آن بدانید. دستور زیر را اجرا کنید و مقادیر مناسب را به جای مقادیر بزرگ‌نویسی شده جایگزین کنید:

```shell
kubectl create secret docker-registry <name> \
  --docker-server=DOCKER_REGISTRY_SERVER \
  --docker-username=DOCKER_USER \
  --docker-password=DOCKER_PASSWORD \
  --docker-email=DOCKER_EMAIL
```

اگر قبلاً فایل اعتبار Docker را دارید، به جای استفاده از دستور فوق، می‌توانید فایل اعتبارها را به عنوان یک {{< glossary_tooltip text="Secret" term_id="secret" >}} Kubernetes وارد کنید.  
[ایجاد Secret براساس اعتبارهای موجود Docker](/docs/tasks/configure-pod-container/pull-image-private-registry/#registry-secret-existing-credentials)
توضیح می‌دهد که چگونه این کار را انجام دهید.

این بسیار مفید است به ویژه اگر از چندین ثبتگاه تصویر خصوصی استفاده می‌کنید، زیرا `kubectl create secret docker-registry` یک Secret ایجاد می‌کند که تنها با یک ثبتگاه خصوصی کار می‌کند.

{{< note >}}
Podها فقط می‌توانند به اسرار جذب تصویر در فضای نام خود ارجاع دهند، بنابراین این فرآیند باید یک بار در هر فضای نام انجام شود.
{{< /note >}}

### اشاره به imagePullSecrets در یک Pod

حالا می‌توانید Podها را که به آن Secret ارجاع داده می‌شوند، با افزودن بخش `imagePullSecrets` به تعریف Pod ایجاد کنید. هر مورد در آرایه `imagePullSecrets` تنها می‌تواند به یک Secret در همان فضای نام ارجاع دهد.

به عنوان مثال:

```shell
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
EOF

cat <<EOF >> ./kustomization.yaml
resources:
- pod.yaml
EOF
```

این باید برای هر Pod که از یک ثبتگاه خصوصی استفاده می‌کند، انجام شود.

با این حال، تنظیم این فیلد می‌تواند با تنظیم imagePullSecrets در یک منبع [ServiceAccount](/docs/tasks/configure-pod-container/configure-service-account/) به صورت خودکار انجام شود.

برای دستورات دقیق، [افزودن ImagePullSecrets به یک حساب سرویس](/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account) را بررسی کنید.

شما می‌توانید این را با یک `.docker/config.json` برای هر گره‌ی کنترلی ترکیب کنید. اعتبارها ترکیب می‌شوند.

## موارد استفاده

برای پیکربندی ثبتگاه‌های خصوصی، تعدادی راه‌حل وجود دارد. اینجا چندین مورد استفاده رایج و راه‌حل‌های پیشنهادی آنها آمده است.

1. خوشه‌ای که فقط تصاویر غیرمتعلق به شرکت (مانند متن‌باز) را اجرا می‌کند. نیازی به پنهان کردن تصاویر نیست.
   - استفاده از تصاویر عمومی از یک ثبتگاه عمومی
     - نیازی به پیکربندی نیست.
     - برخی ارائه‌دهندگان ابر به طور خودکار تصاویر عمومی را نگهداری و یا آنها را آینه‌بری می‌کنند که توسعه پذیری را بهبود می‌دهد و زمان لازم برای جذب تصاویر را کاهش می‌دهد.
1. خوشه‌ای که برخی از تصاویر متعلق به شرکت را اجرا می‌کند که باید برای افراد خارج از شرکت مخفی باشد، اما برای همه کاربران خوشه قابل مشاهده باشد.
   - استفاده از یک ثبتگاه تصویر خصوصی میزبان
     - ممکن است نیاز به پیکربندی دستی در گره‌هایی باشد که نیاز به دسترسی به ثبتگاه خصوصی دارند
   - یا اجرای یک ثبتگاه تصویر خصوصی داخلی پشت دیوار آتش با دسترسی خواندن باز
     - نیازی به پیکربندی Kubernetes نیست.
   - استفاده از خدمات میزبانی شده ثبتگاه تصویر کانتینر که کنترل دسترسی تصویر را انجام می‌دهد
     - با تنظیمات خودکار خوشه بهتر کار می‌کند تا پیکربندی دستی گره‌ها.
   - یا در یک خوشه که تغییر پیکربندی گره ناخوشایند است، از `imagePullSecrets` استفاده کنید.
1. خوشه با تصاویر خصوصی، چندین تصویر از آنها نیاز به کنترل دسترسی دقیق‌تر دارند.
   - مطمئن شوید که [کنترل‌کننده تأیید همواره تصاویر را همیشه بکشید](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) فعال است. در غیر این صورت، همه Podها احتمالاً به تمام ت

صاویر دسترسی دارند.
   - داده‌های حساس را به جای بسته‌بندی آن در یک تصویر، در یک منبع "Secret" انتقال دهید.
1. یک خوشه چند-کاربره که هر کاربر نیاز به ثبتگاه تصویر خصوصی خود دارد.
   - مطمئن شوید که [کنترل‌کننده تأیید همواره تصاویر را همیشه بکشید](/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages) فعال است. در غیر این صورت، همه Podهای همه کاربران احتمالاً به تمام تصاویر دسترسی دارند.
   - یک ثبتگاه تصویر خصوصی با نیاز به احراز هویت را اجرا کنید.
   - برای هر کاربر، اعتبار ثبتگاه را تولید کرده، در Secret قرار دهید و Secret را به هر فضای نام کاربر اضافه کنید.
   - کاربر Secret را به imagePullSecrets هر فضای نام اضافه کند.

اگر نیاز به دسترسی به چند ثبتگاه دارید، می‌توانید برای هر ثبتگاه یک Secret ایجاد کنید.

## ارائه‌دهنده اعتبار قدیمی داخلی kubelet

در نسخه‌های قدیمی‌تر Kubernetes، kubelet ارتباط مستقیمی با اعتبارهای ارائه‌دهنده ابر داشت.
این به او امکان می‌داد که به طور پویا اعتبارها را برای ثبتگاه‌های تصویر بگیرد.

سه پیاده‌سازی داخلی از اینتگراسیون اعتبارهای kubelet وجود داشت:
ACR (Azure Container Registry)، ECR (Elastic Container Registry)، و GCR (Google Container Registry).

برای کسب اطلاعات بیشتر درباره مکانیزم قدیمی، مستندات نسخه Kubernetes که استفاده می‌کنید را بخوانید. Kubernetes v1.26 تا v{{< skew latestVersion >}} شامل مکانیزم قدیمی نیست، بنابراین شما باید یا:
- یک ارائه‌دهنده اعتبار تصویر kubelet را در هر گره پیکربندی کنید
- اعتبارهای پیکربندی تصویر را با استفاده از `imagePullSecrets` و حداقل یک Secret مشخص کنید

## {{% heading "whatsnext" %}}

* مشخصات مانیفست تصویر OCI را از [مشخصه مانیفست تصویر OCI](https://github.com/opencontainers/image-spec/blob/master/manifest.md) بخوانید.
* درباره [جمع‌آوری زباله تصویر کانتینر](/docs/concepts/architecture/garbage-collection/#container-image-garbage-collection) بیشتر بدانید.
* بیشتر درباره [جذب تصویر از یک ثبتگاه خصوصی](/docs/tasks/configure-pod-container/pull-image-private-registry) بیاموزید.