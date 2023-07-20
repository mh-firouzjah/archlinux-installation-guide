# راهنمای نصب آرچ

<div dir="rtl" align="right">

من در اینجا سعی کردم سه مرحله مختلف برای استفاده از آرچ لینوکس رو تفکیک کنم که هر مرحله برای قشر خاصی مفید هست

۱. در مرحله اول فقط یک سیستم عامل خالی نصب می‌کنم که بجز برنامه های لازم برای اتصال به اینترنت و همین طوری یک فایروال چیز اضافه‌ای روش نیست
  این روال ممکنه برای افراد ماهر و آشنا به آرچ کافی باشه تا دیگه بعدش بتونن سیستم رو بوت کنن و هرکاری که نیاز دارن انجام بدن، 
  ضمن اینکه برای یک سخت‌افزار خاص و کوچیک تا همین جاش کافیه که بشه به عنوان یک کامپیوتر ازش استفاده کرد

۲. مرحله بعدی بازم برای افراد آشنا به لینوکس هست اما برنامه‌هایی که بنظر برای بهینه سازی سیستم یا انجام برخی کارها لازم بودن رو اضافه کردم و همینطور
  برخی سرویس‌ها که باید اجرا بشن تا سیستم بهتر کار کنه. درواقع هدف این بود که اگر خواستیم به صورت فقط ترمینالی از سیستم استفاده کنیم 
  بیشتر پکیج‌هایی که نیاز روزمره باشن رو داشته باشیم. هرچند تکمیل نشده این لیست چرا که خب فعلا همین لیست رو برای مرحله آخر هم استفاده 
  کردم و اگر چیزی بیشتر بهش اضافه میکردم تو مرحله بعدی باید پاکسازی انجام میدادم. برای مثال فایل منیجر ترمینالی نداره یا ...

۳. مرحله آخر هم برای افرادی هست که زیاد با لینوکس یا به طور خاص آرچ آشنا نیستن.
  اینجا سعی کردم یک دسکتاپ پلاسما و برخی از ابزارهای کی‌دی‌ای یا سایر برنامه‌های گرافیکی لازم رو نصب کنم.
  برای داشتن یک ورک‌استیشن سبک و کارآمد و البته نه زیاد شلوغ، تا هرکس به فراخور نیاز خودش برنامه‌های بیشتری که لازم داره رو نصب کنه

در کل من نه ریپوزیتوری خاصی اضافه کردم و نه چیزی از آرچ بودن سیستم رو تغییر دادم. یعنی اینطوری نیست که توزیع جدید لینوکس باشه یا 
دستکاری خاصی روی آرچ صورت گرفته باشه، هرچی در آخر کار هست همون آرچ هست. حتی اگر خواستین برنامه‌های نصب شده از لیست رو پاک کنید
میتونید از همون لیست‌ها استفاده کنید و با پکمن اونارو پاک کنید و هرچی خودتون خواستین رو نصب کنید.
من فقط سعی کردم پروسه نصب آرچ ساده و عملی توضیح بدم.


در این روش آرچ را به صورت `efi` و با `systemd-boot` نصب و از فایل سیستم `btrfs` استفاده می‌کنیم.

**  **مدنظر داشته باشید**
`systemd-boot`
هرچند برای روش نصب انتخاب شده
ولی اگر فقط یک سیستم‌عامل دارید از این روش استفاده کنید
و برای دوآل بوت یا غیره از گراب استفاده کنید.
چون سیستم‌دی-بوت کل درایوی که برای
efi
انتخاب می‌کنید را اورراید می‌کنه و مشکلاتی به همراه داره که
حداقل روشی که اینجا برای نصب استفاده شده پاسخگوی اونها نیست
***و خلاصه از ما گفتن بود***.
من چون فقط یک سیستم‌عامل دارم این را انتخاب کردم.

---

## قدم اول تهیه فایل ISO و ساختن یک فلش Bootable از آن

**
بهتره از نسخه
`Torrnet`
برای دانلود استفاده کنید

ابتدا به [صفحه‌ی دانلود آرچ](https://archlinux.org/download/) مراجعه کنید و فایل ایزو را به همراه `signature` مربوطه دانلود کنید

سپس با استفاده از [GnuPG](https://wiki.archlinux.org/title/GnuPG) فایل ایزوی دانلود شده رو با این روش `تائید امضا` بررسی کنید

<div dir="ltr" align="left">

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

<br/>

### بوت‌ایبل کردن فلش

برای اینکار روشهای زیادی وجود داره و برنامه‌های گرافیکی هم هستن که کار رو ساده‌تر کردن اما روش ترمینالی اینکار استفاده از دستور زیر هست
- درصورتی  که نیاز به برنامه گرافیکی دارید توی ویکی اکثر توزیع‌ها به برنامه `Etcher`
اشاره شده و برنامه ساده‌ای هست

<div dir="ltr" align="left">

```bash
dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress
```

</div>

برای اطلاعات بیشتر می‌تونید [این لینک](https://wiki.archlinux.org/title/USB_flash_installation_medium#Using_the_ISO_as_is_(BIOS_and_UEFI)) رو مطالعه کنید


<details>

  <summary>خطاها‌ی زمان بوت شدن</summary>

۱. ممکنه سیستم شما به Secure Boot حساس باشه و در نتیجه نتونه از روش فلش بوت بشه پس باید این وضعیت رو از منوی بایوس خاموش کنید
  برای اینکار سیستم رو ری‌استارت کنید و دکمه‌ی f12 یا f8 یا f2 رو بزنید تا وارد منوی بایوس بشین بین تب‌های موجود حرکت کنید و این وضعیت رو خاموش کنید

۲. ممکنه سیستم شما روی حالت uefi قرار نداشته باشه و درنتیجه بازم نمیتونه از روی فلش بوت بشه پس از منوی بایوس این حالت رو فعال کنید

۳. خطای *No Bootable Device*
  بعد از نصب سیستم و تموم شدن کار اگر ری‌استارت کردین و خطایی دیدن که سیستم نتونسته یک فایل بوت پیدا کنه بازم باید وارد منوی بایوس بشین و
  از روی گزینه‌ای که برای انتخاب Trusted bootloaders هست وارد بشین و فایل بگردین فایل `EFI/systemd/systemd-bootx64.efi` رو انتخاب کنید براش یک اسم دلخواه بذارین و تنظمیات رو ذخیره کنید و ری‌استارت کنید

</details>

<br/>

برای اینکه مطمئن بشیم فلش ما به صورت efi بوت شده دستور زیر رو میزنیم

<div dir="ltr" align="left">

```bash
ls /sys/firmware/efi/efivars
```

</div>

اگر این دستور خروجی نداشته باشه یعنی درحال حاضر این فلش به صورت efi بوت‌ایبل و درنتیجه بوت نشده پس باید دوباره مراحل قبلی رو طی کنید

---

## اتصال به اینترنت

برای اتصال به اینرنت یا از کابل استفاده می‌کنید که خب نیاز به تنظیمات یا کارخاصی ندارید؛ یا از وای‌فای میخواین وصل بشین که روشی که توضیح میدم از ویکی آرچ اقتباس شده و همین رو پیش میریم

راه ساده‌ش میتونید یکجا و در یک دستور بزنید:

<div dir="ltr" align="left">

```bash
iwctl --passphrase <wifi password> station <device e.g wlan0> connect <SSID means wifi-name>
```

</div>


<details>
 <summary>راه طولانی، اگر اسم وای‌فای یادتون رفته یا روش قبلی کار نکرد:</summary>
دستور `ip link` رو بزنید تا ببینید دیوایس‌های سیستم شناخته شدن و فعال هستن

<div dir="ltr" align="left">

```bash
ip link
```

</div>

دستور ‍`iwctl` رو بزنید تا وارد محیط اینتراکتیوش بشید. بعد از اینکه وارد محیط اینتراکتیوش بشین یک تغییر کوچیک توی ترمینال می‌بینید به این شکل

<div dir="ltr" align="left">

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

<div dir="ltr" align="left">

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

<br/>

برای اطلاعات بیشتر به [این لینک](https://wiki.archlinux.org/title/Iwd#iwctl) از ویکی آرچ مراجعه کنید

با دستور پینگ چک کنید که به اینترنت وصل شدین یا نه

<div dir="ltr" align="left">

```bash
ping -c 4 archlinux.org
```

</div>

## تنظیم ساعت سیستم

<div dir="ltr" align="left">

```bash
timedatectl set-ntp true
```

</div>

## بروزرسانی میرورهای پکیج منیجر

برای اینکه در ادامه و برای نصب بسته‌های لازم کمی سرعت دانلود از اینترنت بهتری داشته باشیم
لازمه تا لیست میرورهای مخازن را بروزرسانی کنیم. برای این کار از بسته‌ی reflector استفاده می‌کنیم.
ابتدا باید آن را نصب کنیم و سپس لیست میرورها را آپدیت کنیم.
برای احتیاط قبل از آپدیت کردن از لیست فعلی یک بک‌آپ تهیه می‌کنیم
(من از میرورهای آلمان، فرانسه، انگلیس و هلند استفاده کردم).

<div dir="ltr" align="left">

```bash
cp /etc/pacman.d/mirrorlist  /etc/pacman.d/mirrorlist.bac

pacman -Sy reflector

reflector --ipv6 -c DE,FR,GB,NL -p https -a 24 --sort rate -l 30 --score 20 -f 10 --save /etc/pacman.d/mirrorlist

```

</div>

---

## پارتیشن بندی دیسک

این قسمت شاید یکم با بقیه‌ی راهنماهای نصب آرچ فرق کنه و البته پارتیشن بندی یکم حواس جمع میخواد، که یهو نزنید دیسک رو فرمت کنید و فایلهاتون رو پاک کنید!

دستور `fdisk -l‍` یا `lsblk` برای شناسایی درایوهای سیستم بزنید

حداقل دو تا پارتیشن لازم داریم:
- یکی برای efi و بوت شدن سیستم. حجم ۵۱۲ مگ رو میشه برای حالت تک کرنل امن درنظر گرفت  
  - ولی برای امنیت بیشتر ۱ گیگ بدین
- و یکی هم برای خود سیستم‌عامل و درایوهای هوم و غیره

از ابزار `cfdisk‍` یا `fdsik` برای پارتیشن بندی استفاده کنید. طرز کار ساده‌ای دارن، ابزارهای دیگه هم فرقی نداره استفاده بشن و مهم خروجی کار هست. نهایتا در خروجی کار همچین چیزی لازمه:

<div dir="ltr" align="left">

| partition | size            | fs-type        | mount point | partition                 |
| :---:     | :---:           | :---:          | :---:       | :---:                     |
| efi       | 1 GiB           | EFI system     | /mnt/boot    | /dev/efi_partition |
| root      | remaining space | Linux file sys | /mnt        | /dev/root_partition       |

</div>

<details>
  <summary>اگر چندتا کرنل میخواین نصب کنید یا گراب استفاده می‌کنید</summary>

کرنل‌هایی که نصب می‌کنید و مثلا گراب و متعلقات گراب(مثل تم و این چیزای اضافیش - البته ما که گراب نمی‌زنیم) هم توی همین پارتیشن EFI قرار می‌گیرن پس حواستون باشه اگر قرار چندتا کرنل همزمان نصب کنید این پارتیشن رو باید بزرگتر درنظر بگیرید.
البته یه راه دیگه‌ای هم وجود داره می‌تونید این پارتیشن رو کم حجم بسازید و فقط فایل‌های .efi که لودر هستن رو داخلش قرار بدین ولی باید یک پارتیشن /boot جداگانه درست کنید تا extended boot partition یا به اختصار XBOOTLDR باشه و اون رو جداگانه مانت کنید. در این صورت یعنی ما باید ۳ تا پارتیشن درست کنیم:

<div dir="ltr" align="left">

| partition  |       size        | filesystem-type         | mount point   | partition                      |
|   :---:    |       :---:       |     :---:               |     :---:     |           :---:                |
| efi        |      512 MiB      |   EFI system            |   /mnt/efi    | /dev/efi_partition      |
| boot       |       2 GiB       |   Linux extended boot   |   /mnt/boot   | /dev/extended_boot_partition   |
| root       |  remaining space  |   Linux file sys        |   /mnt        | /dev/root_partition            |

</div>

اون آخر کار هم باید از این دستور برای ایجاد سیستم‌دی بوت استفاده کنید:

`bootctl --esp-path=/mnt/efi --boot-path=/mnt/boot install`

</details>

<br/>

ساختن پارتیشن `swap` الزامی نیست بعدا هم می‌تونید اضافه کنید! تصمیم گیری با خودتون ولی اگر تصمیم گرفتین از zswap که
به صورت پیشفرض در لینوکس فعال هست استفاده کنید پس حتما باید یه پارتیشن سوآپ یا یک سوآپفایل ایجاد کنید.

<br/>

<div dir="ltr" align="left">

```bash
$ fdisk -l
$ fdisk -w /dev/nvme0n1

# Welcome to fdisk (util-linux 2.38.1).
# Changes will remain in memory only, until you decide to write them.
# Be careful before using the write command.

Command (m for help): g
# Created a new GPT disklabel (GUID: 8B0BD377-21F7-5242-9338-AD62411BA6FE).

Command (m for help):  n
Partition number (1-128, default 1): # hit enter
First sector (2048-15441886, default 2048): # hit enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15441886, default 15439871): +1G
# Created a new partition 1 of type 'Linux filesystem' and of size 1 GiB

Command (m for help): n
Partition number (3-128, default 3): # hit enter
First sector (9439232-15441886, default 9439232): # hit enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (9439232-15441886, default 15439871): # hit enter
# Created a new partition 3 of type 'Linux filesystem' and of size 2.9 GiB

Command (m for help): p
# ...
# Device          Start     End    Sectors  Size  Type
# /dev/nvme0n1p1     2048  1050623 1048576  512M  Linux filesystem
# /dev/nvme0n1p2  9439232 15439871 6000640  2.9G  Linux filesystem

Command (m for help): t
Partition number (1-2, default 2): 1
Partition type or alias (type L to list all): 1
#Changed type of partition 'Linux filesystem' to 'EFI System'

Command (m for help): p
# ...
# Device           Start      End  Sectors  Size  Type
# /dev/nvme0n1p1     2048  1050623 1048576  1G    EFI System
# /dev/nvme0n1p2  9439232 15439871 6000640  2.9G  Linux filesystem

Command (m for help): w
# The partition table has been altered

```

</div>

## ساختن فایل سیستم

اول پارتیشن بوت منیجر رو فرمت می‌کنیم، دستور زیر رو بزنید

<div dir="ltr" align="left">

```bash
mkfs.fat -F 32 -n EFI /dev/nvme0n1p1
```
</div>

- فلگ n توی این مورد برای لیبل دادن به پارتیشن استفاده شده و آپشنال(اختیاری) هست


برای پارتیشن اصلی هم دستور زیر رو می‌زنیم.
اما اگر یک پارتیشن ویندوزی دارین که میخواین به همون فرمت ویندوزی باقی بمونه
و فرمتش نمی‌کنید یا فرمت می‌کنید ولی
بازم میخواین که توی ویندوز قابل استفاده باشه نیازه پکیج
ntfs-3g
نصب کنید همچنین برای احتیاط یک پکیج برای
btrfs
هم چک می‌کنیم که نصب هست یا نه

<div dir="ltr" align="left">

```bash
mkfs.btrfs -f -L ROOT /dev/nvme0n1p2
```

</div>

از اینجا به بعد من از
`nvme0n1p2`
استفاده می‌کنم، حواستون باشه که
`nvme0n1p2`
رو باید با اسم درایو مربوطه روی سیستم خودتون جایگزین کنید

بریم سراغ ساختن ساب‌ولیوم‌ها، اول باید پارتیشن روت رو مانت کنیم و بعد ساب‌ولیوم‌ها رو روش بسازیم

<div dir="ltr" align="left">

```bash
mount /dev/nvme0n1p2 /mnt
```

</div>

به این ساب‌ولیوم‌ها نیاز داریم

> @, @home, @root

این ها هم دستوراتی که استفاده می‌کنیم و اختصارشون

<div dir="ltr" align="left">

> su = subvolume,
> cr = create,
> li = list

</div>

ساختن ساب‌ولیوم‌ها

<div dir="ltr" align="left">

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

<details>
  <summary>چرا توی اسلش بوت مانتش کردم و یک گیگ فضا بهش دادم - بازم فضا کمه؟</summary>

در اصل توی پارتیشن efi قراره چندتا فایل کوچیک و کم حجم که پسوند efi دارن قرار بگیره
و این فایلها توسط بایوس در لحظه روشن شدن سیستم استفاده میشن تا درنهایت کل سیستم رو استارت کنن. 
برای همین شاید ببینید موقع نصب سایر توزیع‌ها وقتی حالت اتوماتیک پارتیشن بندی رو انتخاب می‌کنید اونا شاید ۳۰۰ مگ فضا برای این پارتیشن درنظر بگیرن.
درواقع اگه فقط همون چندتا فایل efi که برای بایوس لازم داریم بخوان توی اون پارتیشن باشن همون ۳۰۰ مگ هم زیاده 
ولی ما چون سیستم‌دی-بوت میزنیم وقتی داریم Entry برای بوت لودر میسازیم مجبوریم که 
فایل انتری و فایلهای کرنل و initramfs رو یکجا و داخل دایرکتوری esp یا همون پارتیشنی که برای بوت میسازیم قرار بدیم.

راه ساده و بی‌دردسرش این بود که بیایم و پارتیشن رو یکم بزرگتر و با حجم بیشتر بگیریم و کرنل و همه چیزش رو هم بذاریم همین جا 
بوت لودر و انتری پوینت‌هاش هم همین جا باشن.

البته یه نکته‌ی دیگه هم وجود داره و اون اینکه اگر بازم دیدین فضایی که جدا کردین داره کم میاد میتونید از قابلیت کمپرس کردن 
توی mkinitcpio استفاده کنید تا initramfs و فال‌بکش رو براتون فشرده کنه!
درحالت عادی فشرده سازی انجام میشه ولی لول فشرده کردنش زیاد نیست. میتونید عدد بالاتری بهش بدین و بیشتر فشرده کنید تا خروجی خیلی کم حجم‌تر بشه ولی طبعا 
انتظار اینم داشته باشید که زمان بیشتری طول میکشه تا دستور mkinitcpio کارش رو تموم بکنه!

```bash
COMPRESSION_OPTIONS=(-9) #=> this will be considred as Max level for each algorithm
or
COMPRESSION_OPTIONS=(-v -5 --long) #=> to save space for custom kernels (especially with a dual boot setup)
```

</details>

<br />

<div dir="ltr" align="left">

```bash
bop=ssd,discard,noatime,nodiratime,compress-force=zstd:8,commit=30,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@

mount -o ${bop} /dev/nvme0n1p2 /mnt

mkdir -p /mnt/{boot/efi,hdd,home,root}

mount -o ${bop}home /dev/nvme0n1p2 /mnt/home

mount -o ${bop}root /dev/nvme0n1p2 /mnt/root

mount /dev/nvme0n1p1 /mnt/boot

ntfs-3g -o rw,nls=utf8,noatime,windows_names /dev/sda1 /mnt/hdd
```

</div>

---

## نصب پکیج‌های پایه

در این مرحله حداقل‌هایی رو نصب می‌کنیم که میشه گفت روی سیستم مدنظر یک آرچ خیلی سبک و پایه رو داشته باشیم که یک فایروال و یک ادیتور و همین طور اتصال به اینترنت داره و بعدش دیگه هرطور دلتون می‌خواد می‌تونید این سیستم رو شخصی‌سازیش کنید و کاملتر کنید.

کدی که در ادامه اومده شامل مراحل زیر هست:
- نصب پکیج‌های پایه و همین طور فریم‌ور لینوکس و میکروکد برای پردازند اینتل و پکیج‌های کمکی برای مانت کردن پارتیشن‌های ویندوزی یا همون ntfs و پارتیشن‌های btrfs به همره یک پکیج کمکی برای کامل کردن دستورات بش با زدن دکمه تب و فایروال و بسته‌های لازم برای اتصال به اینترنت و در آخر ادیتور ویم
- ایجاد فایل fstab که نشون میده بعد از هر بار بوت شدن سیستم عامل، پارتیشن‌ها چطوری مانت بشن
- چون یبار با رفلکتور میرورهای پکمن رو بر اساس سرعت دانلود کردن ازشون مرتب کردیم همین لیست رو میبریم رو سیستم مقصد
- حالا با chroot میریم وارد سیستمی که نصب کردیم میشیم تا تنظیمات پایه‌ای رو انجام بدیم که بشه این سیستم رو بوت کرد و بهش لاگین کرد
- لوکال تایم رو با توجه به منطقه زمانی ست میکنیم
- ساعت سیستم رو هم تنظیم میکنیم
- زبان سیستم و تنظیمات نمایش تاریخ و ساعت و غیره رو ایجاد میکنیم و از روش جنریت میکنیم
- زبان کلی سیستم و کیبورد رو تنظیم میکنیم
- هاست نیم یا هون اسمی که موقع اتصال بلوتوث، هات اسپات یا شبکه کردن چندتا سیستم و ... دیده میشه رو ست میکنیم
- تنظیمات شبکه پایه رو انجام میدیم
- سیستم‌دی بوت رو نصب میکنیم و بعدش بوت لودر رو مینویسیم
- چون از سیستم‌دی برای بوت کردن سیستم استفاده کردیم mkinitcpio رو باید آپدیت کنیم تا initramfs رو دوباره جنریت کنیم (این همون فایلی هست که سیستم‌دی موقع بوت ازش استفاده میکنه)
- برای یوزر روت یک پسورد میذاریم
- چون ادیتور vim رو نصب کردیم همین رو به صورت سمبلیک لینک میکنیم به vi که اگر برنامه‌ای به صورت پیشفرض دنبال vi رفت از همین vim  استفاده بشه(vi محیطش با vim یکم فرق داره!)
- یه یوزر جدید میسازیم عضو گروه wheel میکنیم و براش پسورد میذاریم و همین طور با visudo میریم به کل گروه wheel اجازه استفاده از سودو به شرط داشتن پسورد میدیم
- فایروال رو تنظیم و بعدش فعال میکنیم
- سرویسهای لازم برای ایجاد اتصال به اینترنت با کابل یا بی سیم رو فعال میکنیم



<div dir="ltr" align="left">

```bash
pacstrap /mnt base base-devel linux-lts linux-firmware intel-ucode ntfs-3g btrfs-progs zram-generator firewalld dhcpcd iwd vim

genfstab -U /mnt >> /mnt/etc/fstab

mv /mnt/etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist.back

cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist

arch-chroot /mnt

# set timezone
ln -sf /usr/share/zoneinfo/Asia/Tehran /etc/localtime

# adjust time
hwclock --systohc

# generate locale
sed -i '/en_US.UTF-8/s/^#\s*//g' /etc/locale.gen

locale-gen

# generate language locale.conf
echo "LANGE=en_US.UTF-8" > /etc/locale.conf

# set consolefont
echo -e "FONT=LatArCyrHeb-14\nFONT_MAP=UTF-8" > /etc/vconsole.conf

# set hostname
hostname=my_bluetooth_name

echo "${hostname}" > /etc/hostname

# set hosts
echo -e "127.0.0.1 localhost\n::1  localhost" > /etc/hosts
echo "127.0.1.1 ${hostname}.localdomain ${hostname}" >> /etc/hosts

# initiate bootloader
bootctl install

# to set a loader edit `/boot/loader/loader.conf` and define a default entry
vim /boot/loader/loader.conf
---------------------
default arch-lts.conf
timeout 0
editor no
console-mode max
```
</div>

**اینجا رو حواستون باشه!**
مخصوصا تو قسمت آپشن‌ها اگر چیزی رو اشتباه تایپی داشته باشین بعدا سیستم بوت نمیشه
و پیدا کردن ارور هم خیلی سخت میشه!
ولی اگر موقع بوت مشکل سوئیچ به روت داشت یا کلا بوت نمیشد
و به منوی بایوس برمی‌گشت ممکنه
توی نوشتن این قسمت اشتباهی انجام داده باشین پس میتونید با
chroot
شدن یبارم اینها رو چک کنید
(کافیه پارتیشن بوت رو مانت کنید توی mnt و محتوای این فایلها رو بررسی کنید)

<div dir="ltr" align="left">

```bash
vim /boot/loader/entries/arch-lts.conf
---------------------
title Arch Linux LTS
linux /vmlinuz-linux-lts
initrd /intel-ucode.img
initrd /initramfs-linux-lts.img
options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw quiet nowatchdog add_efi_memmap acpi_osi=Linux

vim /boot/loader/entries/fallback.conf
---------------------
title Fallback
linux /vmlinuz-linux-lts
initrd /intel-ucode.img
initrd /initramfs-linux-lts-fallback.img
options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw quiet nowatchdog add_efi_memmap acpi_osi=Linux

# check if boot entry is recognized and has no errors
bootctl list

# edit `/etc/mkinitcpio.conf` at `HOOKS=` replace `udev` hook with `systemd`.
HOOKS=(base udev[systemd] fsck ...)

# regenerate initramfs
mkinitcpio -P

# set password for root user
passwd

# create another user
useradd -m -G wheel -s /bin/bash <your username>
passwd <your username>

# use vim as vi
ln -s /usr/bin/vim /usr/bin/vi

# inorder to give new user sudo access
visudo
## type /%wheel hit enter to find the line %wheel ALL=(ALL) ALL type ^ then hit x then type :wq

# network and firewall
systemctl enable firewalld.service
systemctl enable dhcpcd.service
systemctl enable iwd.service

# exit from chroot
exit

# un mount partitions
umount /mnt/boot
umount /mnt

# reboot and remove the live media
reboot
```

</div>

تا اینجا شما یک آرچ مستقل رو روی سیستم مقصد نصب کردین. الان میتونید اون رو بوت کنید و دیگه از روی این سیستم جدید شروع کنید کاملش کنید.

---

## بعد از نصب

خب الان دیگه باید موفق شده باشین به سیستم تازه نصب شده خودتون لاگین کنید.
از اینجا به بعد اختیاری هست منتهی بعضی از کارهایی که میخوام بگم خیلی خیلی بهتره که انجام بدین
و بعضیای دیگه هم دیگه سلیقه خودتون هست.
برای مثال اینکه درایورهای کارت گرافیک یا کارت صدا و این جور چیزا رو نصب کنید
خب مشخصه که ضروری هست، ولی اینکه از دسکتاپ پلاسما استفاده کنید
یا کلا دسکتاپ نزنید یا دسکتاپ دیگری انتخاب کنید سلیقه خودتون باشه.


### Replacing ZSWAP with ZRAM

<div dir="ltr" align="left">

```bash
echo "blacklist zswap" > /etc/modprobe.d/disable-zswap.conf

vim /etc/systemd/zram-generator.conf
---------------------
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
swap-priority = 100
fs-type = swap

vim /etc/sysctl.d/99-vm-zram-parameters.conf
---------------------
vm.swappiness = 180
vm.watermark_boost_factor = 0
vm.watermark_scale_factor = 125
vm.page-cluster = 0
```

### Add intel graphics kernel parameters

```bash
echo "options i915 enable_rc6=1 enable_fbc=1 fastboot=1" > /etc/modprobe.d/intel.conf

mkinitcpio -P
```

</div>

### شخصی سازی آرچ و نصب دسکتاپ پلاسما

<div dir="ltr" align="left">

```bash
echo -e "Color\nILoveCandy\nParallelDownloads = 3" | tee -a /etc/pacman.conf >/dev/null

pacman -S --needed - < pkgs.txt

systemctl enable $(cat services.txt)

# Install pikaur as pacman wraper
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/pikaur.git /tmp
cd /tmp/pikaur
makepkg -fsri

pikaur -S - < aur.txt

cp ./fonts.conf ~/.config/fontconfig/

fc-cache -f -v

pacman -Rnscu $(pacman -Qtdq)
```

</div>

## حالتهای خاص

سرویس acpid به فشرده شدن کلیدهای جهت یا arrow keys هم حساسیت نشون میده
و این باعث میشه لاگ سیستم رو الکی پر کنه
برای جلوگیری از این رفتارش تو ویکی آرچ این راه حل رو گفتن:

<div dir="ltr" align="left">

```bash
vim /etc/acpi/events/buttons
---------------------
event=button/(up|down|left|right|kpenter)
action=<drop>
```

</div>

اگر گوگل کروم استفاده می‌کنید اینها هم بد نیست اضافه کنید

<div dir="ltr" align="left">

```bash
vim ~/.config/chrome-flags.conf
---------------------
#--incognito
--process-per-site
--disk-cache-dir="$XDG_RUNTIME_DIR/chromium-cache"
--js-flags=--noexpose_wasm
--disable-features=UseSkiaRenderer
```

</div>

برای اجرای سرویس profile-sync-daemon هم باید درحالت یوزر خودتون
و بدون سودو اینکارها رو انجام بدین

<div dir="ltr" align="left">

```bash
psd
# First time running psd so please edit ~/.config/psd/psd.conf to your liking and run again.
systemctl --user enable psd
Created symlink ~/.config/systemd/user/default.target.wants/psd.service → /usr/lib/systemd/user/psd.service.
systemctl --user start psd
```

</div>

در صورت استفاده از دو مانیتور
این اسکریپت کمک میکنه تا تصویر روی مانیتور اکسترنال بیاد،
البته که این مشکل رو داشتین که مانیتور اکسترنال تصویر نداشت.

<div dir="ltr" align="left">

edit `/usr/share/sddm/scripts/Xsetup` and add:

---------------------
```sh
#!/bin/sh

# run xrandr to see which monitors you have
intern=eDP1
extern=HDMI1

if xrandr | grep "$extern disconnected"; then
    xrandr --output "$extern" --off --output "$intern" --auto
else
    xrandr --output "$intern" --off --output "$extern" --auto
    ## To leave the default monitor enabled when an external monitor is connected, replace the else clause with
    # xrandr --output "$intern" --primary --auto --output "$extern" --right-of "$intern" --auto
fi
```

</div>

برای شناسایی ماوس و تاچ پد در صفحه SDDM این تغییرات رو لازم داشتم:

<div dir="ltr" align="left">

edit/create `/etc/X11/xorg.conf.d/20-touchpad.conf`
---------------------
```bash
Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"

        Option "Tapping" "on"
        Option "NaturalScrolling" "on"
        Option "MiddleEmulation" "on"
        Option "DisableWhileTyping" "on"
EndSection
```

edit/create `/etc/X11/xorg.conf.d/20-mouse.conf`
---------------------
```bash
Section "InputClass"
        Identifier "libinput mouse catchall"
        MatchIsPointer "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"

        Option "MiddleEmulation" "on"
        Option "DisableWhileTyping" "on"
EndSection
```

</div>

این دوتا هم برای اینکه درست سینک باشه

<div dir="ltr" align="left">

edit/create `/etc/sddm.conf.d/kde_settings.conf`
---------------------
```bash
[General]
Session=plasma.desktop

[Wayland]
Enable=false
```

edit/create `/etc/X11/xorg.conf.d/20-intel.conf`
---------------------
```bash
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
EndSection
```

</div>

---

برای کانفیگ کردن ادیتور ویم من از روشی که در 
[این لینک](https://www.freecodecamp.org/news/vimrc-configuration-guide-customize-your-vim-editor/) گفته شده استفاده کردم. البته روشهای خیلی زیادی وجود داره برای شروع یه نگاهی بهش بندازین

</div>