# راهنمای نصب آرچ

<div dir='rtl' align='right'>

این مطلب در راستای کمک به نصب آرچ لینوکس تهیه شده

در این روش آرچ را به صورت `efi` و با `systemd-boot` نصب و از فایل سیستم `btrfs` استفاده می‌کنیم

 - مدنظر داشته باشید که `systemd-boot` هرچند برای روش نصب انتخاب شده ولی این روش بهتر است برای وقتی که تنها یک سیستم‌عامل روی سیستم دارید استفاده بشود و برای دوآل بوت یا غیره 
 از گراب استفاده کنید. چرا که سیستم‌دی-بوت کل درایوی که برای efi انتخاب می‌کنید را اورراید می‌کند و مشکلاتی به همراه دارد. من چون فقط یک سیستم‌عامل دارم این را انتخاب کردم.
---

## قدم اول تهیه فایل ISO و ساختن یک فلش bootable از آن

ابتدا به [صفحه‌ی دانلود آرچ](https://archlinux.org/download/) مراجعه کنید و فایل ایزو را به همراه `signature` مربوطه دانلود کنید

سپس با استفاده از [GnuPG](https://wiki.archlinux.org/title/GnuPG) فایل ایزوی دانلود شده رو با این روش `تائید امضا` بررسی کنید

<div dir='ltr' align='left'>

```bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```  

</div>

<details>
  <summary>تائید امضا</summary>

> مطابق اطلاعات [این لینک](https://www2.cs.arizona.edu/stork/packagemanagersecurity/attacks-on-package-managers.html)
  در صورتی که پکیج‌ها بروز رسانی نشوند امکان آلوده شدن آنها به بدافزارها افزایش پیدا می‌کند
  بدین جهت پکیج‌مینجرها طوری طراحی شدند تا پکیج‌ها(بسته‌ها) را به صورت خودکار بروزرسانی کنند
  اما در صورتی که خود این پکیج‌منیجرها از قبل آلوده باشند امکان اینکه مجددا بسته‌های آلوده به سیستم اضافه کنند یا بسته‌های آلوده‌ی قدیمی را درست بروزرسانی نکنند وجود خواهد داشت.
  پکیج‌منیجرها معمولا با دسترسی‌های بالا و بدون محدودیت اجرا می‌شوند تا قادر به اعمال تغییرات و بروزرسانی در سطوح مختلف سیستم باشند؛
  بنابراین، اقدامات مدیر بسته بر کل سیستم تأثیر می گذارد  
  تکنیک‌های مثل man in the middle یا سایر روش‌ها ممکنه استفاده شده باشن و برای مثال وب سایت توزیع مینت در سالهای دورتر مورد چنین حمله‌های قرار گرفته بود و فایل ایزو که روی سایت
  بود و ملت دانلود می‌کردن درواقع آلوده بود و خب مدتی بعد گندش در اومد. خلاصه اینکه این مرحله رو یکم جدی بگیرید به نفع خودتون هست

</details>

### بوت‌ایبل کرن فلش  

برای اینکار روشهای زیادی وجود داره و برنامه‌های گرافیکی هم هستن که کار رو ساده‌تر کردن اما روش ترمینالی اینکار استفاده از دستور زیر هست  

<div dir='ltr' align='left'>

```bash
dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress
```

</div>

برای اطلاعات بیشتر می‌تونید [این لینک](https://wiki.archlinux.org/title/USB_flash_installation_medium#Using_the_ISO_as_is_(BIOS_and_UEFI)) رو مطالعه کنید
  
<details>
  <summary>خطاها‌ی زمان بوت شدن</summary>  

  ۱- ممکنه سیستم شما به Secure Boot حساس باشه و در نتیجه نتونه از روش فلش بوت بشه پس باید این وضعیت رو از منوی بایوس خاموش کنید
  برای اینکار سیستم رو ری‌استارت کنید و دکمه‌ی f12 یا f8 یا f2 رو بزنید تا وارد منوی بایوس بشین بین تب‌های موجود حرکت کنید و این وضعیت رو خاموش کنید

  ۲- ممکنه سیستم شما روی حالت uefi قرار نداشته باشه و درنتیجه بازم نمیتونه از روی فلش بوت بشه پس از منوی بایوس این حالت رو فعال کنید

  ۳- خطای *No Bootable Device*  
  بعد از نصب سیستم و تموم شدن کار اگر ری‌استارت کردین و خطایی دیدن که سیستم نتونسته یک فایل بوت پیدا کنه بازم باید وارد منوی بایوس بشین و
  از روی گزینه‌ای که برای انتخاب Trusted bootloaders هست وارد بشین و فایل بگردین فایل `EFI/systemd/systemd-bootx64.efi` رو انتخاب کنید براش یک اسم دلخواه بذارین و تنظمیات رو ذخیره کنید و ری‌استارت کنید

</details>

برای اینکه مطمئن بشیم فلش ما به صورت efi بوت شده دستور زیر رو میزنیم

<div dir='ltr' align='left'>

```bash
ls /sys/firmware/efi/efivars
```

</div>

اگر این دستور خروجی نداشته باشه یعنی درحال حاضر این فلش به صورت efi بوت‌ایبل و درنتیجه بوت نشده پس باید دوباره مراحل قبلی رو طی کنید

---

## اتصال به اینترنت

برای اتصال به اینرنت یا از کابل استفاده می‌کنید که خب نیاز به تنظیمات یا کارخاصی ندارید؛ یا از وای‌فای میخواین وصل بشین که روشی که توضیح میدم از ویکی آرچ اقتباس شده و همین رو پیش میریم

راه ساده‌ش میتونید یکجا و در یک دستور بزنید:

<div dir='ltr' align='left'>

```bash
iwctl --passphrase <wifi password> station <device e.g wlan0> connect <SSID means wifi-name>
```

</div>


<details>
 <summary>راه طولانی:</summary>
دستور `ip link` رو بزنید تا ببینید دیوایس‌های سیستم شناخته شدن و فعال هستن

<div dir='ltr' align='left'>

```bash
ip link
```

</div>

دستور ‍`iwctl` رو بزنید تا وارد محیط اینتراکتیوش بشید. بعد از اینکه وارد محیط اینتراکتیوش بشین یک تغییر کوچیک توی ترمینال می‌بینید به این شکل

<div dir='ltr' align='left'>

```bash
$ iwctl

[iwd]$
```

</div>

میتونید دستور `help` رو بزنید تا راهنمای این ابزار براتون بیاد و مطالعه کنید

برای اینکه دیوایس وایرلس سیستم رو پیدا کنید دستور `device list` رو بزنید

برای اینکه این دیوایس بتونه وای‌فای‌های اطراف پیدا کنه دستور `station device scan` که در اینجا کلمه‌ی *device* رو باید اسم دیوایس وایرلس سیستم خودتون عوض کنید

با دستور `station device get-networks` که دوباره کلمه‌ی دیواس رو باید اصلاح کنید، می‌بینید که چه وای‌فای‌هایی رو پیدا کرده

حالا با دستور `station device connect SSID` می‌تونید به وای‌فای مدنظرتون وصل بشین.(SSID = اسم وای‌فای، برای اسم وای‌فای چند کاراکتر اول رو بنویسید و دکمه تب کیبورد رو بزنید.)

<div dir='ltr' align='left'>

```bash
[iwd]$ help
[iwd]$ device list
[iwd]$ station device scan
[iwd]$ station device get-networks
[iwd]$ station device connect SSID
```

</div>

اگر نیاز به پسورد برای وای‌فای باشه یک پرامپت باز میشه تا پسورد رو بگیره
</details>

برای اطلاعات بیشتر به [این لینک](https://wiki.archlinux.org/title/Iwd#iwctl) از ویکی آرچ مراجعه کنید

با دستور پینگ چک کنید که به اینترنت وصل شدین یا نه

<div dir='ltr' align='left'>

```bash
ping -c 4 archlinux.org
```

</div>

ساعت سیستم رو تنظیم کنید

<div dir='ltr' align='left'>

```bash
timedatectl set-ntp true
```

</div>

## بروزرسانی میرورهای مدیر بسته

برای اینکه در ادامه و برای نصب بسته‌های لازم کمی سرعت دانلود از اینترنت بهتری داشته باشم لازم است تا لیست میرورهای مخازن را بروزرسانی کنیم. برای این کار از
بسته‌ی reflector استفاده می‌کنیم. ابتدا باید آن را نصب کنیم و سپس لیست میرورها را آپدیت کنیم. قبل از آپدیت کردن از لیست فعلی یک بک‌آپ تهیه می‌کنیم برای احتیاط

<div dir='ltr' align='left'>

```bash
cp /etc/pacman.d/mirrorlist  /etc/pacman.d/mirrorlist.bac
pacman -Sy reflector
reflector --country Germany,France,England,Nederland --protocol https --age 24 --sort rate --latest 20 --save /etc/pacman.d/mirrorlist
```

</div>

---

## پارتیشن بندی دیسک

این قسمت شاید یکم با بقیه‌ی راهنماهای نصب آرچ فرق کنه و البته پارتیشن بندی یکم حواس جمع میخواد، که یهو نزنید دیسک رو فرمت کنید و فایلهاتون رو پاک کنید!  

دستور `fdisk -l‍` یا `lsblk` برای شناسایی درایوهای سیستم بزنید

حداقل دو تا پارتیشن لازم داریم:
- یکی برای efi و بوت شدن سیستم. حجم ۵۱۲ مگ رو میشه برای حالت تک کرنل امن درنظر گرفت ولی برای امنیت بیشتر ۱ گیگ بدیم. 
- و یکی هم برای خود سیستم‌عامل و درایوهای هوم و غیره،‌ از ابزار `cfdisk‍` یا `fdsik` برای پارتیشن بندی استفاده کنید. طرز کار ساده‌ای دارن، ابزارهای دیگه هم فرقی نداره استفاده بشن و مهم خروجی کار هست. نهایتا در خروجی کار همچین چیزی لازمه:

| partition  |       size        | filesystem-type|  mount point  |         partition         |
|   :---:    |       :---:       |     :---:      |     :---:     |           :---:           |
| boot       |       1 GiB       |   EFI system   | /mnt/boot/efi | /dev/efi_system_partition |
| root       |  remaining space  | Linux file sys |     /mnt      | /dev/root_partition       |

<details>
  <summary>اگر چندتا کرنل میخواین نصب کنید یا گراب استفاده می‌کنید</summary>

کرنل‌هایی که نصب می‌کنید و مثلا گراب و متعلقات گراب(مثل تم و این چیزای اضافیش - البته ما که گراب نمی‌زنیم) هم توی همین پارتیشن EFI قرار می‌گیرن پس حواستون باشه اگر قرار چندتا کرنل همزمان نصب کنید این پارتیشن رو باید بزرگتر درنظر بگیرید.
البته یه راه دیگه‌ای هم وجود داره می‌تونید این پارتیشن رو کم حجم بسازید و فقط فایل‌های .efi که لودر هستن رو داخلش قرار بدین ولی باید یک پارتیشن /boot جداگانه درست کنید تا extended boot partition یا به اختصار XBOOTLDR باشه و اون رو جداگانه مانت کنید. در این صورت یعنی ما باید ۳ تا پارتیشن درست کنیم:

| partition  |       size        | filesystem-type         | mount point   | partition                   |
|   :---:    |       :---:       |     :---:               |     :---:     |           :---:             |
| efi        |      512 MiB      |   EFI system            |   /mnt/efi    | /dev/efi_system_partition   |
| boot       |       2 GiB       |   Linux extended boot   |   /mnt/boot   | /dev/efi_system_partition   |
| root       |  remaining space  |   Linux file sys        |   /mnt        | /dev/root_partition         |

اون آخر کار هم باید از این دستور برای ایجاد سیستم‌دی بوت استفاده کنید:

 ‍`bootctl --esp-path=/mnt/efi --boot-path=/mnt/boot install`

</details>
ساختن پارتیشن `swap` الزامی نیست بعدا هم می‌تونید اضافه کنید! تصمیم گیری با خودتون

<details>
  <summary>پارتیشن swap چیست</summary>

  در توضیح اینکه پارتیشن swap چی هست و چرا هست جواب‌های زیاد و طولانی‌ای هست که با شرح و تفصیل توضیحش میدن. ولی من سعی میکنم مختصر بنویسم.  
  پارتیشن swap به همراه مقدار رم فیزیکی و سخت‌افزاری روی سیستم شما، یک virtual memory یا رم مجازی تشکیل میدن، همچنین این پارتیشن برای بحث hibernation استفاده می‌شه.  
  وقتی حجم رم فیزیکی روی سخت‌افزار کم باشه و برنامه‌های زیادی رم زیادی مصرف کنن مشکلاتی پیش‌میاد که منجر به کرش کردن سیستم و اشکال در اجرای برنامه‌ها میشه،
  لینوکس یک ماژول از کرنل به اسم OOM killer mechanism رو اجرا می‌کنه تا به صورت اتوماتیک پروسس‌ها رو کیل(kill) کنه تا رم آزاد کنه.
  در لینوکس سیستم‌عامل میاد و از رم به صورت تکه‌های به اسم page یا صفحه استفاده می‌کنه، یعنی برنامه‌ها در واقع رم رو در این صفحات اشغال می‌کنن و عملیات swapping که سیستم‌عامل
  لینوکس درنظر گرفته به این منظور هست که سیستم‌عامل بتونه بعضی از این صفحات رم که برنامه‌ها استفاده می‌کنن رو به یک بخشی از هارد دیسک که از قبل برای همین منظور آماده شده
  منتقل کنه؛ خب پس پارتیشن swap رو آماده می‌کنیم برای این منظور. لینوکس با استفاده از اینکار دو نتیجه‌ی مهم به دست میاره فرض کنیم یه وقتی برنامه‌هایی که اجرا می‌کنیم درنهایت از سیستم
  رمی بیشتر از آنچه که واقعا به صورت فیزیکی و سخت‌افزاری داریم بخوان، سیستم‌عامل میاد و اون صفحاتی از رم که کمتر درحال استفاده هستند رو منتقل می‌کنه به پارتیشن swap و
  رم واقعی سیستم رو خالی می‌کنه اینطوری اون نیاز به رم بیشتر برای برنامه‌ای که خواسته بوده رو پوشش میده. مورد دوم اینکه بعضی برنامه‌ها رم زیادی رو موقع بالا اومدن مصرف می‌کنن ولی
  این حجم اولیه‌ی مصرف شده در طول اجرای برنامه‌ مورد استفاده نیست درنتیجه این صفحاتی که اشغال شدن هم گزینه‌ی خوبی برای انتقال به پارتیشن swap هستن و سیستم‌عامل با اینکار بازم
  صفحات رم بیشتری رو خالی می‌کنه تا برای کارهای مهم‌تر بعدی در دسترس باشن.  
  اما فقط خوبی رو نگیم، استفاده از همچین رم مجازی‌ای بدی هم داره و اون اینه که سرعت خواندن و نوشتن توی رم فیزیکی خیلی خیلی بیشتر از هارد فیزیکی هست و
  پارتیشن swap یا swap-file که ما داریم درموردش حرف می‌زنیم روی هارد قرار داره! پس درمجموع فعال کردن و استفاده از این پارتیشن باعث میشه سرعت کلی سیستم افت کنه
  ولی با قرار دادن این پارتیشن روی یک SSD سرعت بهتری نسبت به HDD بدست میاریم.  
  فرق زیادی بین پارتیشن یا فایل بودن swap پیدا نکردم به جز اینکه تو [ویکی آرچ](https://wiki.archlinux.org/title/swap#:~:text=memory%20is%20exhausted.-,Note%3A,-The%20factual%20accuracy)
   دیدم سر این موضوع بین علما اختلاف افتاده و به نظر آقای لینوس توروالدز استفاده از پارتیشن ارحجیت داره از طرفی مطابق مستندات BTRFS استفاده از swap-file دردسرهایی داره روی این فایل سیستم و تنظیم کردن دقیق آپشن‌هایی که برای fstab مثلا هست سخته و خلاصه شر میشه از خیرش بگذریم

بنظرم [اینجا](https://forum.endeavouros.com/t/swapfile-vs-swap-partition-vs-no-swap-at-all/10932/6) خوب خلاصمون کرد

یک نکته‌ای هم دوستانی که می‌خوان از هایبرنیت استفاده کنن مدنظر داشته باشن که این عمل هرچند سرعت بوت شدن رو بیشتر می‌کنه(روی ssd که زیاد محسوس نیست) و ممکنه
    از فوایدش خوشتون بیاد ولی در عمل بسیار بیشتر از بوت شدن معمولی به سیستم فشار میاد مخصوصا به SSD ها. حالا اینکه یکی میگه ssd خریدم که ازش استفاده کنم حالش رو ببرم
    پس هایبرنیت هم می‌کنم. یا یکی میگه خب این هایبرنیت که برام مهم نیست پس الکی سخت‌افزارم رو خراب نکنم نظراتی هست که هرکدوم محترم هست  

نکته‌ای که بنظرم حائز اهمیت هست و بهتره درموردش تحقیق کنید اینه که هاردهای SSD برخلاف هاردهای HDD عمر مفیدشون به عملکرد نوشتن و پاک‌کردن وابسته هست. بدین صورت که در
    هاردهای قدیمی‌تر و نوری یا HDD خراب شدن اینها به خاطر خراب شدن هدشون بود و عملا نوشتن یا پاک‌کردن تاثیری رو اون دیسک نوری نمی‌ذاشت اما در SSD ها تعداد دفعات نوشتن و پاک‌کردن
    اثر مستقیم روی عمر قطعه داره و در همین راستا هرچی بتونیم عملیات‌های نوشتن و پاک‌کردن کمتری روی SSD تحمیل کنیم عمر مفیدش بیشتر میشه. این قضیه برای نسل‌های اولیه SSD ها
    حیاتی بود چون واقعا و در عمل خیلی زود پر می‌شدن و می‌سوختن، اما نسل‌های جدیدتر خیلی بهتر شدن و از طرفی سیستم‌عامل‌ها و حتی سخت‌افزارها هم برای کار با SSD ها ارتقا پیدا کردن
    و شاید این ترس امروز خیلی کمتر حس بشه، اما بد نبود که درموردش بنویسم

البته در مراحل پارتیشن بندی از یک سری فلگ برای fstab استفاده می‌کنم که در راستای بهینه‌تر مصرف کردن SSD باشه و عمر مفید بیشتری براش باقی بمونه
  
</details>

<details>
  <summary>برای پارتیشن بندی من از fdisk استفاده می‌کنم </summary>  

اول دستور `fdisk /dev/sdX` رو میزنیم تا وارد محیط این ابزار بشیم(اگر از ssd های nvme استفاده میکنید بجای sdX باید nvme0n1 یا همچین چیزی رو بزنید[تب بزنید خودش کامل میکنه]
   و در غیر این صورت هم باید `X` رو با حرفی که برای پارتیشن مدنظرتون هست عوض کنید) استفاده از `-w` برای اینه که اگر روی دیسک پارتیشن بندی داشتیم ارور و وارنینگ نداشته باشیم و از اون پارتیشن بندی هم صرف نظر کنیم

با نوشتن `m` و زدن اینتر (نوشتن دستور فلان و زدن اینتر رو خلاصه کنیم، با زدن دستور فلان) لیست راهنمای این ابزار باز میشه که میتونید در هر مرحله بهش روجوع کنید  

با زدن دستور`g` جدول بندی دیسک رو به حالت GPT تغییر میده که برای نصب ما این حالت لازمه

با دستور `n` یک پارتیشن جدید معرفی می‌کنیم که اولین پارتیشن رو برای بوت منیجر درنظر می‌گیریم و حجم یک گیگ و فایل سیستم efi رو براش درنظر میگیریم
   درواقع اول n رو می‌زنیم، برای سوال بعدیش دیافلتش که ۱ هست کافیه و فقط اینتر رو می‌زنیم، برای سوال بعدیش هم فقط اینتر رو میزنیم، برای سوال بعدی علامت + رو بذارید و 512M رو
   بدون فاصله بعدش بنویسد، که یعنی یک گیگ براش حجم ببره(میشه +1G هم نوشت.) و اینتر رو میزنیم تا پارتیشن اول ایجاد بشه

با دستور `p` میتونیم چک کنیم که پارتیشن مدنظر ایجاد شده و سایزش هم درسته، می‌تونیم با دستور `d` هم پارتیشنی رو حذف کنیم

مجدد روال رو طی می‌کنیم تا پارتیشن‌های بیشتری که لازم هستن رو بسازیم مطابق جدول بالا. دستور `n` رو میزنیم. سوال اول و دوم رو فقط اینتر می‌زنیم،
   و سوال چالشی اینجاست! برای swap چقدر حجم ببریم؟ این واقعا جواب‌های مختلفی داره ولی برای هایبرنیت حجم رم + ۲ گیگ برای اطمینان از از دست نرفتن دیتا و برای سایر سیستم‌ها
   اگر رم سیستم کمتر از ۲ گیگ هست یک گیگ و اگر بین ۲ تا ۴ گیگ هست ۲ گیگ و برای ۴ گیگ و بیشتر هم ۴ گیگ مقادیر مرسومی هستن، ولی همونطور که گفته شد اگر ویرچوآل مموری
   بیشتری لازم دارین مقدارش رو بیشتر درنظر بگیرید. ضمن اینکه می‌تونید کلا این مرحله رو رد بشین و swap نسازید
   حتما علامت + رو قبل از عددی که برای حجم میدین بذارید.  

برای آخرین پارتیشن که قراره روت و هوم و سایر پارتیشن‌ها رو داخلش و با استفاده از subvolume بسازیم. مجدد دستور `n` رو می‌زنیم، دو سوال اول رو فقط اینتر می‌زنیم و این دفعه
   سوال سوم هم اینتر می‌زنیم چون همه‌ی حجم باقی‌مونده رو میخوایم بدیم به این پارتیشن  

دستور `p` رو یکبار دیگه بزنیم تا مطمئن بشیم همه چیز همونطور که لازمه درست شده  

دستور `t` رو می‌زنیم تا تایپ پارتیشن‌ها رو هم مشخص کنیم، در جواب سوال اول عدد ۱ رو می‌زنیم و برای سوال دوم هم مجدد عدد ۱ رو می‌دیم تا پارتیشن از تایپ efi باشه
   مجددا دستور `t` رو میزنیم و این‌بار عدد ۲ رو می‌دیم تا تایپ پارتیشن دومی رو مشخص کنیم، به سوال بعدیش عدد ۱۹ رو میدیم و تایپ linux swap رو انتخاب می‌کنیم
   و برای پارتیشن بعدی هم پیش‌فرض خودش درسته و نیازی به تغییر نداره

دستور `w` رو می‌زنیم تا تغییرات روی دیسک رایت بشه و از fdisk خارج بشیم

</details>

<div dir='ltr' align='left'>

```bash
$ fdisk -l
$ fdisk -w /dev/nvme0n1

# Welcome to fdisk (util-linux 2.38.1).
# Changes will remain in memory only, until you decide to write them.
# Be careful before using the write command.

Command (m for help): g
#Created a new GPT disklabel (GUID: 8B0BD377-21F7-5242-9338-AD62411BA6FE).

Command (m for help):  n
Partition number (1-128, default 1): # hit enter
First sector (2048-15441886, default 2048): # hit enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15441886, default 15439871): +512M
#Created a new partition 1 of type 'Linux filesystem' and of size 512 MiB

Command (m for help): n
Partition number (3-128, default 3): # hit enter
First sector (9439232-15441886, default 9439232): # hit enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (9439232-15441886, default 15439871): # hit enter
#Created a new partition 3 of type 'Linux filesystem' and of size 2.9 GiB.

Command (m for help): p
# ...
# Device          Start     End    Sectors  Size  Type
# /dev/nvme0n1p1     2048  1050623 1048576  512M  Linux filesystem
# /dev/nvme0n1p2  9439232 15439871 6000640  2.9G  Linux filesystem

Command (m for help): t
Partition number (1-3, default 3): 1
Partition type or alias (type L to list all): 1
#Changed type of partition 'Linux filesystem' to 'EFI System'.

Command (m for help): p
# ...
# Device           Start      End  Sectors  Size  Type
# /dev/nvme0n1p1     2048  1050623 1048576  512M  EFI System
# /dev/nvme0n1p2  9439232 15439871 6000640  2.9G  Linux filesystem
 
Command (m for help): w
# The partition table has been altered.
$
```

</div>

## ساختن فایل سیستم

اول پارتیشن بوت منیجر رو فرمت می‌کنیم، دستور زیر رو بزنید

<div dir='ltr' align='left'>

```bash
mkfs.fat -F 32 -n EFI /dev/efi_system_partition   # efi_system_partition = /dev/sda1 or /dev/nvme0n1p1
```

</div>

برای پارتیشن اصلی هم دستور زیر رو می‌زنیم. اما اگر یک پارتیشن ویندوزی دارین که میخواین به همون فرمت ویندوزی باقی بمونه و فرمتش نمی‌کنید یا فرمت می‌کنید ولی
بازم میخواین که توی ویندوز قابل استفاده باشه نیازه پکیج ntfs-3g نصب کنید همچنین برای احتیاط یک پکیج برای btrfs هم چک می‌کنیم که نصب هست یا نه

<div dir='ltr' align='left'>

```bash
pacman -Sy ntfs-3g btrfs-progs
mkfs.btrfs -f -L ROOT /dev/your_drive3 # your_drive3 = /dev/sda2 or /dev/nvme0n1p2
```  

</div>

از اینجا به بعد من از `nvme0n1p2` استفاده می‌کنم، حواستون باشه که `nvme0n1p2` رو باید با اسم درایو مربوطه روی سیستم خودتون جایگزین کنید

<details>
  <summary>درباره‌ی فایل سیستم btrfs بخونیم</summary>

 درواقع این فایل سیستم زیاد قدیمی نیست و حتی بعضیام میگن که استیبل نشده، ولی خب توزیع‌های زیادی رفتن سراغش و امکانات جالبی هم داره. برای مثال میتونه یک جور LVM بسازه که volume رو میتونه روی چند دیوایس یا هارددیسک پیاده سازی و یا subvolume هایی که روی یک volume میسازیم هم درواقع پوزیکس فایلهایی میشن که هرچند واقعا درایو یا پارتیشن نیستن ولی در ادامه می‌بینیم که ما اونها رو mount می‌کنیم. از مزایای این ساب‌ولیوم‌ها اینه که از فضای ولیومی که روشن قرار دارن اشتراکی استفاده می‌کنن و درواقع دیگه نیازی نیست نگران این باشیم که وای درایو فلانم داره پر میشه از یکی دیگه حجم جدا کنم به این اضافه کنم. از دیگر مزایای فایل سیستم btrfs میشه به قابلیت فشرده سازی و compress داده‌ها اشاره کرد که اینم در ادامه می‌بینیم چجوریه و مزیتش اینه که عملیات نوشتن و پاک کردن روی هارد کمتر میشه و روی defragmentaion هم تاثیر میذاره ولی خوب یا بد بودن این اثرگذاری رو باید بررسی کنیم.  
یک مورد خوب دیگه هم استفاده از اسنپ‌شات‌هایی بسیار کم حجم هست به طوری که اسنپ‌شات‌ها یجوری شبیه گیت فقط تغییرات رو ثبت می‌کنن و کلی ایمیج سنگین از کل پارتیشن‌های ما نمیسازن. btrfs این امکان رو میده تا تنظیم کنیم برای چه پارتیشن‌ها و حالا که بلدیم ساب‌ولیوم‌هایی اتوماتیک اسنپ‌شات تولید بشه یا اصلا تو اسنپ‌شات اونارو لحاظ نکنه و [امکانات بیشتر](https://btrfs.readthedocs.io/en/latest/index.html)  

</details>

<details>
  <summary>درباره‌ی LVM بخونیم</summary>

  مفهومی به اسم LVM یا Logical Volume Management در نیای کامپیوتر وجود دارد، قبل از این مفهوم سیستم عامل دیسک‌های حافظه را به صورت یک بسته‌ی مستقل که روی آن پارتیشن‌هایی وجود داشت یا خود سیستم‌عامل روی آن ایجاد می‌کرد می‌شناخت؛ در این صورت ما در آخرین لایه‌ی سیستمی در فضای پارتیشن‌های به ظرفیت هاردیسک فیزیکی محدود بودیم و حداکثر فضایی که برای یک پارتیشن میشد متصور بود برابر با حداکثر ظریفیت یک هارد دیسک بود. اما LVM این محدودیت را از بین می‌برد، به این صورت که هارددیسک‌های فیزیکی که به صورت خلاصه PV(Physical Volume) نامیده می‌شوند در یک گروه قرار می‌گیرند و تشکیل یک Volume Group یا به اختصار VG می‌دهند. هر VG به صورت یک دیسک درنظر گرفته می‌شود درحالیکه خود ممکن است از یک یا چند PV یعنی دیسک‌های فیزیکی تشکیل شده باشد. بر روی یک VG که از نظر سیستم‌عامل حالا یک دیسک حافظه هست(`/dev/sda`) گروه‌هایی از Logical Volume ها ایجاد می‌شود. به Logical Volume اختصارا LV گفته می‌شود و می‌توان آن را مشابه با `/dev/sda1` فرض کرد که ما می‌توانیم فایل‌سیستم‌های خود مثل ext4, ntfs و... را روی آن‌ها ایجاد کنیم و داده‌هایمان را روی آنها بریزیم. LVM درواقع مثل یک پرده‌ی پوشاننده روی هارددیسک‌های فیزیکی قرار می‌گیرد و آنها را از دید سیستم‌عامل مخفی می‌کند و VG را به عنوان هارددیسک جا می‌زند درنتیجه می‌توان تعداد هاردیسک‌های فیزیکی سیستم را کم یا زیاد کرد ولی سیستم‌عامل از این تغییر آگاه نباشد، یا حافظه‌های اختصاص یافته به VG ها یا LV ها را به صورت داینامیک و پویاتری تغییر داد و در مدیریت حافظه بسیار دست بازی‌تری به ما می‌دهد.

</details>

<details>
  <summary>درباره‌ی Compression بخونیم</summary>

  در فایل سیستم BtrFS یا ‌B-Tree File System می‌توان بدون فشرده‌سازی یا با یکی از سه الگوریتم مختلف برای فشرده‌سازی دیتا استفاده کرد، ZLIB، LZO و ZSTD که مطابق با بنچمارک‌ها ZLIB پرفورمنس و سرعت عمل کمتری داشته و ZSTD در چند مورد از LZO بهتر بوده است. اما برای مثال توزیع مانجارو در نصب به صورت BTRFS از الگوریتم LZO استفاده کرده است(ممکن است در نسخه‌های دیگرش این الگوریتم را عوض کند). اما من تصمیم گرفتم درمورد الگوریتم ZSTD بنویسم. این الگوریتم دارای چندین سطح فشرده سازی هست که مطابق با مستنداتش از سطح ۱-۳ فشرده‌سازی سبک‌تر و کمتری انجام می‌دهد و به اصطلاح real-time هست، این یعنی سرعت عمل بیشتر و همین طور فشار کمتری به پردازنده(cpu) و از سطح ۴-۹ که فشرده‌سازی جدی‌تری انجام می‌دهد و طبعا کمی سرعت کمتر و همچنین سربار بیشتر برای پردازنده و از سطح ۱۰-۱۵ هم جهت تلاش بر نهایت توان فشرده‌سازی که البته شاید تاثیری به‌سزایی در خروجی نداشته باشد ولی قطعا افت سرعت و همچین لود سنگین‌تری روی پردازنده خواهد داشت. به صورت پیش‌فرض درصورتی که خاصیت فشرده‌سازی zstd برای این فایل سیستم فعال کنیم در سطح ۳ از آن استفاده می‌کند. ولی خب من ترجیح می‌دهم سطح ۸ را انتخاب کنم، همانطور که پیشتر اشاره کردم فشرده کردن دیتا در عمل نوشتن و پاک‌کردن از روی هارددیسک تاثیر مفیدی دارد و برداشت من از مستندات این الگوریتم این است که انتخاب سطح ۹ حد وسط هست. همچنین مطابق اطلاعاتی که از ویکی آرچ بدست آوردم اعمال فشرده سازی اجباری هرچند قدری پردازنده باز اضافی تحمیل می‌کند اما در نهایت برای فشرده‌سازی بهینه‌تر خواهد بود چرا که درحالت عادی ممکن است برای بعضی از داده‌ها فشرده‌سازی صرف‌نظر شود و سیستم با تشخصی خود از اینها بگذرد. به صورت پیش فرض و فقط برای فعال کردن فشرده سازی کافی‌است پاریتشن مدنظر را با دستوری مشابه دستوری زیر مانت کنیم  

<div dir='ltr' align='left'>

```bash
mount -o compress=zstd /dev/sdx /mnt
```

</div>

اطلاعات بیشتر و لینک‌های بنچمارک‌ها و سایر اطلاعات رو از [ویکی آرچ](https://wiki.archlinux.org/title/Btrfs#Compression) بخونید  
 مطابق تصویر زیر من یک تست گرفتم روی سیستم مانجارو که دارم از روش این مطلب رو می‌نویسم و می‌بینید سرعت فشرده‌سازی و از فشرده‌سازی درآوردن برای سطوح تست شده چقدر بوده

![Screenshot_20220822_003822](https://user-images.githubusercontent.com/44068054/185809006-1012ca80-f0ef-4661-b073-da02fac4d513.png)
  
</details>

بریم سراغ ساختن ساب‌ولیوم‌ها، اول باید پارتیشن روت رو مانت کنیم و بعد ساب‌ولیوم‌ها رو روش بسازیم

<div dir='ltr' align='left'>

```bash
mount /dev/nvme0n1p2 /mnt
```

</div>

به این ساب‌ولیوم‌ها نیاز داریم

> @, @home, @root

این ها هم دستوراتی که استفاده می‌کنیم و اختصارشون

<div dir='ltr' align='left'>

> su = subvolume
> cr = create
> li = list

</div>

ساختن ساب‌ولیوم‌ها  

<div dir='ltr' align='left'>

```bash
btrfs su cr /mnt/@  
btrfs su cr /mnt/@home  
btrfs su cr /mnt/@root  
btrfs su li /mnt  
umount /mnt  
```

</div>

#### حالا مانت کردن ساب‌ولیوم‌ها

هرچند توی راهنمای نصب و پارتیشن بندی برای uefi سیستم گفته که پارتیشن efi رو توی /boot/efi مانت کنیم
اما چون ما قصد داریم از systemd-boot استفاده کنیم پارتیشن مدنظر را در /boot مانت می‌کنیم

<div dir='ltr' align='left'>

```bash
  mount -o ssd,discard,noatime,compress=zstd:8,commit=30,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@ /dev/nvme0n1p2 /mnt  

  mkdir -p /mnt/{boot/efi,hdd,home,root}

  mount -o ssd,discard,noatime,compress=zstd:8,commit=30,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@home   /dev/sda3 /mnt/home
  mount -o ssd,discard,noatime,compress=zstd:8,commit=30,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@root   /dev/sda3 /mnt/root
  
  mount /dev/nvme0n1p1 /mnt/boot
  mount -t ntfs-3g -o rw,uid=0,gid=0,umask=0000,nls=utf8,noatime,windows_names /dev/sda1 /mnt/hdd
```

</div>

<details>
  <summary> درمورد آپشن‌هایی که در مانت نوشتم </summary>

  ۱- آپشن noatime – No access time. برای پرفورمنس بهتر سیستم هست این رو قرار بدیم چرا که درغیر این صورت به دیتای تایم هم نیاز داره که ضرورتی نداره
  
  ۲- آپشن commit – فاصله‌های زمانی که دیتا روی حافظه‌ی سخت نگاشته می‌شود، مقدار پیش‌فرض ۳۰ ثانیه هست درصورتی که این گپ رو خیلی زیاد بگیریم دیتایی زیادی رو تو صف قرار میدیم برای رایت شدن رو هارددیسک که ممکنه در اثر کرش شدن سیستم دیتا از دست بره
  
  ۳- آپشن compress – انتخاب یک الگوریتم برای فشرده سازی که من zstd رو انتخاب کردم و سطح ۸ از ۱۵ رو گرفتم و البته مطابق مستندات ویکی آرچ فورس کردنش باعث میشه قدری مصرف سی‌پی‌یو بیشتر بشه اما داده‌های بیشتری رو سعی می‌کنه فشرده کنه و برای ssd ها بودنش بهتر هست همچنین هرچه قدر لول فشرده سازی رو بیشتر بگیریم برای ssd بهتر هست چون دیتای کمتری نوشته و بعدا شاید پاکسازی بشه اما از ۸ به بعد فقط فشار زیادی به سی‌پی‌یو میاد و سرعت نوشتن کمتر میشه ولی مقدار فشردگی تغییر خیلی به سزایی نخواهد داشت
  
  ۴- آپشن subvol – این گزینه هم برای اینه که مشخص کنیم روی کدوم ساب‌ولیوم‌ها میخوایم مانت بشه

  ۵- آپشن noautodefrag/autodefrag - این گزینه بسته به اینکه روی ssd هستین یا نه انتخاب میشه، به این صورت که برای ssd عمل fragmentation نه تنها خوب نیست بلکه باعث میشه یک عملیات خواندن و پاک‌کردن و نوشتن درجایی دیگر انجام بشه که خب برای ssd اینکار تاثیری روی سرعت خواندن یا بهینه‌سازی نداره که هیچ! تازه عملیات نوشتن و پاک‌کردن بی‌موردی انجام میده و برای ssd ضرر داره اینکار اما همین حالت برای hdd خوبه و یکپارچه‌سازی داده رو به همراه داره

  ۶- آپشن discard=async - این گزینه هم بیشتر برای ssd ها مناسب هست. آزاد کردن بلاک‌هایی که تحت استفاده‌ی فایل سیستمی نیستند

</details>

</div>

---

## نصب پکیج‌های پایه

در این مرحله حداقل‌هایی رو نصب می‌کنیم که میشه گفت روی سیستم مدنظر یک آرچ خیلی سبک و پایه رو داشته باشیم
در ادامه پکیج‌های بیشتری رو نصب می‌کنیم تا یک سیستم قابل استفاده بدست بیاریم و بتونیم از روی همون سیستم و نه از روی این لایو مدیا کار رو ادامه بدیم
بعد از نصب اینها حالا یک آرچ روی سیستم مدنظر داریم وارد محیط آن می‌شویم و باقی کارها را مستقیم از داخل همان ماشین ادامه می‌دهیم
از آنجا که روی سیستم مقصد نیز باید از روی میرورها بیسته‌های جدید را نصب کنیم، پس لیست میرورهایی که تازه بروزرسانی کردیم به سیستم جدید منتقل می‌کنیم

<div dir='ltr' align='left'>

```bash
pacstrap /mnt base base-devel linux-lts linux-firmware intel-ucode ntfs-3g btrfs-progs bash-completion dhcpcd iwd vim   

genfstab -U /mnt >> /mnt/etc/fstab

mv /mnt/etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist.back

cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist

arch-chroot /mnt

# set timezone
ln -sf /usr/share/zoneinfo/Asia/Tehran /etc/localtime

# adjust time
hwclock --systohc

# generate locale
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen

# generate language locale.conf
echo "LANGE=en_US.UTF-8" > /etc/locale.conf

# config hostname
echo "${custom-hostname}" > /etc/hostname

# config hosts
echo "127.0.0.1 localhost" > /etc/hosts
echo "::1  localhost" >> /etc/hosts
echo "127.0.1.1 ${custom-hostname}.localdomain ${custom-hostname}" >> /etc/hosts

# bootloader as we are going to use systemd-boot
bootctl  --boot-path=/boot --esp-path=/boot/efi install
```

</div>

یک لیست از پکیج‌هایی که برای یک دسکتاپ حداقلی لازم هست تهیه کردم که ضمیمه‌هست اونها رو یکجا نصب میکنیم ولی در ادامه توی توضیحات درموردشون می‌خونیم که چی هستن و چرا نصب شدن
بجز در قسمت زیری باقی جاهایی که از پکمن استفاده شده نیازی نیست استفاده بشه چون پکیج‌های اون قسمت‌ها رو اینجا داره نصب می کنه

<div dir='ltr' align='left'>

```bash
# enabling chaotic-aur
pacman-key --recv-key FBA220DFC880C036 --keyserver keyserver.ubuntu.com
pacman-key --lsign-key FBA220DFC880C036
pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst' 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'

echo "[chaotic-aur]" >> /etc/pacman.conf
echo "Include = /etc/pacman.d/chaotic-mirrorlist" >> /etc/pacman.conf
 
echo "Color" >> /etc/pacman.conf
echo "ILoveCandy" >> /etc/pacman.conf
echo "ParallelDownloads = 5" >> /etc/pacman.conf

pacman -Sy --needed - < pkg-list.out

# network and firewall
ufw enable
systemctl enable ufw.service
systemctl enable NetworkManager
systemctl enable bluetooth.service
systemctl enable acpid.service
sudo ln -s /usr/bin/vim /usr/bin/vi
pacman -Rns $(pacman -Qtdq)
pacman-optimize
```

</div>

در این قسمت باید بوت‌بولدر سیستم‌دی را کانفیگ کنیم

<div dir='ltr' align='left'>

```bash
# set a loader for boot
## esp is the partition which is mounted for efi system partition
vim esp/loader/loader.conf
---------------------
default      arch.conf
timeout      0
editor       no
console-mode auto

# config the loader entery and set some kernel modules(options)
vim esp/loader/entries/arch.conf
---------------------
title    Arch Linux
linux    /vmlinuz-linux
initrd   /intel-ucode.img
initrd   /initramfs-linux.img
options  root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ elevator=deadline add_efi_memmap rw quiet splash loglevel=3 vt.global_cursor_default=0 plymouth.ignore_serial_consoles rd.systemd.show_status=auto r.udev.log_priority=3 nowatchdog fbcon=nodefer i915 i915.fastboot=1 i915.enable_fbc=1 i915.invert_brightness=1 intel_iommu=on,igfx_off nvidia_drm.modeset=1
```

</div>

درصورتی که فقط گرافیک nvidia دارین در قسمت options  و اونهایی که با i915 شروع شدن رو پاک کنید. اون برای کسایی هست که گرافیک اینتل داشته باشند(چه دوآل چه تکی) این چندتا آپشن رو وارد کنید

<div dir='ltr' align='left'>

```bash
nvidia nvidia_modeset nvidia_uvm nvidia_drm
```

</div>

در این مرحله یک سری تنظمیات برای بهینه‌سازی و شخصی‌سازی بیشتر انجام میدم.
 اینها ضروری نیستن و ممکنه در گذر زمان عوض بشن ولی درحال حاضر
توی ویکی آرچ هستن و برای کمی دستکاری بیشتر استفاده کردم.

<div dir='ltr' align='left'>

```bash
# pacman hook to automate the bootloader update after kernel update
mkdir /etc/pacman.d/hooks/
vim /etc/pacman.d/hooks/100-systemd-boot.hook
---------------------
[Trigger]
Type=Package
Operation=Upgrade
Target=systemd

[Action]
Description=Updating systemd-boot
When=PostTransaction
Exec=/usr/bin/bootctl update

# enabling silent boot
echo "kernel.printk = 3 3 3 3" > /etc/sysctl.d/20-quiet-printk.conf

# hide startx messages
echo "[[ $(fgconsole 2>/dev/null) == 1 ]] && exec startx -- vt1 &> /dev/null" >> .bash_profile
```

</div>

در فایل زیر هم باید udev را با systemd در هوک‌ها عوض کنیم و برای گرافیک اینتل هم intel_agp و i915 رو اضافه کردم
برای گرافیک‌های انویدیا یا ای‌ام‌دی هم همینکار رو بکنید.

<div dir='ltr' align='left'>

```bash
# To hide fsck messages during boot, let systemd check the root filesystem. For this, replace udev hook with systemd
vim /etc/mkinitcpio.conf

HOOKS=( base udev[systemd] fsck [intel_agp i915] ...)

mkinitcpio -P

# set password for root user
passwd

# create another user
useradd -m -G wheel -s /bin/bash <your username>
passwd <your username>
cp ~/.bash_profile /home/<your username>/.bash_profile

# inorder to give new user sudo access
visudo
## type /%wheel hit enter to find the line %wheel ALL=(ALL) ALL type ^ then hit x then type :wq
```

</div>

برای یوزر جدید با استفاده از visudo وارد تنظمیات sudo بشین و خط %wheel ALL=(ALL) AL رو از کامنت دربیارین (هشتگ سر خط رو پاک کنید)

---

## بخش گرافیکی

در این بخش من قصد دارم دسکتاپ کی‌دی‌ای و پلاسما را نصب و کانفیگ کنم
پکیج‌هایی که لازم داریم و تنظیماتی که باید بعد از نسب آن‌ها انجام بشن رو به همراه توضیح مختصری می‌نویسم
برای شناسایی کارت گرافیکی که روی سیستم دارین دستور زیر رو بزنید و بعد هم با پکمن سرچ کنید ببینید بسته‌ی مناسب سخت‌افزار شما کدوم هست

<div dir='ltr' align='left'>

```bash
lspci -v | grep -A1 -e VGA -e 3D

pacman -Ss xf86-video

pacman -S xf86-video-intel1 mesa lib32-mesa
pacman -S nvidia linux-headers nvidia-utils lib32-nvidia-utils nvidia-settings
pacman -S vulkan-icd-loader vulkan-intel lib32-vulkan-icd-loader

# pacman hook to automate initcpio update after nvidia update
vim /etc/pacman.d/hooks/nvidia.hook
---------------------
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=linux
# Change the linux part above and in the Exec line if a different kernel is used

[Action]
Description=Update NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'

# added this line to kernel parameters
nvidia_drm.modeset=1
```

</div>

<div dir='ltr' align='left'>

```bash
pacman -S xorg plasma-desktop plasma-pa sddm sddm-kcm nm-connection-editor network-manager-applet
pacman -S kdegraphics-thumbnailers ffmpegthumbs powerdevil power-profiles-daemon bluez bluez-utils bluedevil
pacman -S smplayer kde-gtk-config baloo kscreen colord-kde ttf-dejavu ttf-liberation

systemctl enable display-manager.service

```

برای کانفیگ کردن ادیتور ویم من از روشی که در [این لینک](https://www.freecodecamp.org/news/vimrc-configuration-guide-customize-your-vim-editor/) گفته شده استفاده کردم. البته روشهای خیلی زیادی وجود داره برای شروع یه نگاهی بهش بندازین
