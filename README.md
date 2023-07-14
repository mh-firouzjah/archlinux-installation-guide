# راهنمای نصب آرچ

<div dir="rtl">

در این روش آرچ را به صورت `efi` و با `systemd-boot` نصب و از فایل سیستم `btrfs` استفاده می‌کنیم

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

## قدم اول تهیه فایل ISO و ساختن یک فلش bootable از آن

**
بهتره از نسخه
Torrnet
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

### بوت‌ایبل کرن فلش  

برای اینکار روشهای زیادی وجود داره و برنامه‌های گرافیکی هم هستن که کار رو ساده‌تر کردن اما روش ترمینالی اینکار استفاده از دستور زیر هست  

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

ساعت سیستم رو تنظیم کنید

<div dir="ltr" align="left">

```bash
timedatectl set-ntp true
```

</div>

## بروزرسانی میرورهای مدیر بسته

برای اینکه در ادامه و برای نصب بسته‌های لازم کمی سرعت دانلود از اینترنت بهتری داشته باشم لازم است تا لیست میرورهای مخازن را بروزرسانی کنیم. برای این کار از
بسته‌ی reflector استفاده می‌کنیم. ابتدا باید آن را نصب کنیم و سپس لیست میرورها را آپدیت کنیم. قبل از آپدیت کردن از لیست فعلی یک بک‌آپ تهیه می‌کنیم برای احتیاط

<div dir="ltr" align="left">

```bash
cp /etc/pacman.d/mirrorlist  /etc/pacman.d/mirrorlist.bac

pacman -Sy reflector

reflector -c DE,FR,GB,NL -p https -a 24 --sort rate -l 20 -f 10 --save /etc/pacman.d/mirrorlist

```

</div>

---

## پارتیشن بندی دیسک

این قسمت شاید یکم با بقیه‌ی راهنماهای نصب آرچ فرق کنه و البته پارتیشن بندی یکم حواس جمع میخواد، که یهو نزنید دیسک رو فرمت کنید و فایلهاتون رو پاک کنید!  

دستور `fdisk -l‍` یا `lsblk` برای شناسایی درایوهای سیستم بزنید

حداقل دو تا پارتیشن لازم داریم:
- یکی برای efi و بوت شدن سیستم. حجم ۵۱۲ مگ رو میشه برای حالت تک کرنل امن درنظر گرفت ولی برای امنیت بیشتر ۱ گیگ بدیم. 
- و یکی هم برای خود سیستم‌عامل و درایوهای هوم و غیره،‌ از ابزار `cfdisk‍` یا `fdsik` برای پارتیشن بندی استفاده کنید. طرز کار ساده‌ای دارن، ابزارهای دیگه هم فرقی نداره استفاده بشن و مهم خروجی کار هست. نهایتا در خروجی کار همچین چیزی لازمه:

<div dir="ltr" align="left">

| partition | size            | fs-type        | mount point | partition                 |
| :---:     | :---:           | :---:          | :---:       | :---:                     |
| efi       | 1 GiB           | EFI system     | /mnt/efi    | /dev/efi_partition |
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

ساختن پارتیشن `swap` الزامی نیست بعدا هم می‌تونید اضافه کنید! تصمیم گیری با خودتون

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

<br/>

<div dir="ltr" align="left">

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
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15441886, default 15439871): +1G
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
Partition number (1-2, default 2): 1
Partition type or alias (type L to list all): 1
#Changed type of partition 'Linux filesystem' to 'EFI System'.

Command (m for help): p
# ...
# Device           Start      End  Sectors  Size  Type
# /dev/nvme0n1p1     2048  1050623 1048576  1G    EFI System
# /dev/nvme0n1p2  9439232 15439871 6000640  2.9G  Linux filesystem

Command (m for help): w
# The partition table has been altered.

```

</div>

## ساختن فایل سیستم

اول پارتیشن بوت منیجر رو فرمت می‌کنیم، دستور زیر رو بزنید

<div dir="ltr" align="left">

```bash
mkfs.fat -F 32 -n EFI /dev/nvme0n1p1
```

</div>

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

> su = subvolume
> cr = create
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

<div dir="ltr" align="left">

```bash
bop=ssd,discard,noatime,compress-force=zstd:8,commit=30,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@

mount -o ${bop} /dev/nvme0n1p2 /mnt

mkdir -p /mnt/{boot/efi,hdd,home,root}

mount -o ${bop}home /dev/nvme0n1p2 /mnt/home

mount -o ${bop}root /dev/nvme0n1p2 /mnt/root

mount /dev/nvme0n1p1 /mnt/boot

ntfs-3g -o rw,uid=0,gid=0,umask=000,nls=utf8,noatime,windows_names /dev/sda1 /mnt/hdd
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
pacstrap /mnt base base-devel linux-lts linux-firmware intel-ucode ntfs-3g btrfs-progs zram-generator ufw dhcpcd iwd vim

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
print "LANGE=en_US.UTF-8" > /etc/locale.conf

# set consolefont
print "FONT=LatArCyrHeb-14\nFONT_MAP=UTF-8" > /etc/vconsole.conf

# set hostname
hostname=my_bluetooth_name

print "${hostname}" > /etc/hostname

# set hosts
print "127.0.0.1 localhost\n::1  localhost" > /etc/hosts
print "127.0.1.1 ${hostname}.localdomain ${hostname}" >> /etc/hosts

# initiate bootloader
bootctl install

# to set a loader edit `/boot/loader/loader.conf` and define a default entry
print "default arch-lts.conf\ntimeout 0\neditor no\nconsole-mode max" > /boot/loader/loader.conf

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

# disabling zswap, zram will be enabled later
print "blacklist zswap" > /etc/modprobe.d/disable-zswap.conf

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


# network and firewall
ufw default deny incoming
ufw default allow outgoing
ufw enable
systemctl enable ufw.service
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
از اینجا به بعد اختیاری هست منتهی بعضی از کارهایی که میخوام بگم خیلی خیلی بهتره که انجام بدین و بعضیای دیگه هم دیگه سلیقه خودتون هست.  
برای مثال اینکه درایورهای کارت گرافیک یا کارت صدا و این جور چیزا رو نصب کنید خب مشخصه که ضروری هست، ولی اینکه از دسکتاپ پلاسما استفاده کنید یا کلا دسکتاپ نزنید یا دسکتاپ دیگری انتخاب کنید سلیقه خودتون باشه.


### add intel graphics kernel parameters

```bash
print "options i915 enable_rc6=1 enable_fbc=1 fastboot=1" > /etc/modprobe.d/intel.conf

mkinitcpio -P
```

### شخصی سازی آرچ و نصب دسکتاپ پلاسما

<div dir="ltr" align="left">

```bash
print "Color\nILoveCandy\nParallelDownloads = 3" | tee -a /etc/pacman.conf >/dev/null

pacman -S --needed - < pkg-list.out

systemctl enable NetworkManager
systemctl enable bluetooth.service
systemctl enable acpid.service

pacman -Rnsuc $(pacman -Qtdq)

```

</div>

## حالتهای خاص

در صورت استفاده از دو مانیتور این اسکریپت کمک میکنه تا تصویر روی مانیتور اکسترنال بیاد، البته که این مشکل رو داشتین که مانیتور اکسترنال تصویر نداشت.

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

برای شناسایی ماوس و تاچ پد در صفحه SDDM این تغییرات رو لازم داشتم:

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

این دوتا هم برای اینکه درست سینک باشه

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

---

برای کانفیگ کردن ادیتور ویم من از روشی که در [این لینک](https://www.freecodecamp.org/news/vimrc-configuration-guide-customize-your-vim-editor/) گفته شده استفاده کردم. البته روشهای خیلی زیادی وجود داره برای شروع یه نگاهی بهش بندازین
