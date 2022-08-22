# راهنمای نصب آرچ

این مطلب در راستای کمک به نصب آرچ لینوکس تهیه شده  
در این روش آرچ را به صورت efi و با systemdboot نصب و از فایل سیستم btrfs استفاده می‌کنیم

 - سعی نمی‌کنم همه‌ی اصطلاحات رو ترجمه کنم و اکثرا کلمات انگلیسی رو فقط فارسی می‌نویسم تا دوستانی که با این کلمات آشنا نیستن هم خونده باشن، بعضی از این کلمات مثل `محیط اینتراکتیو` و موارد دیگه هم چون در عمل نشون داده میشه بهتر درک میشه و در آخر اینکه هرچی هم منابع فارسی پیدا کنید باز مجبورید یه روزی به سراغ منابع انگلیسی برید و خب آشنایی با کلمات انگلیسی اونجا کار رو ساده‌تر می‌کنه
  
---

## قدم اول: تهیه فایل ISO و ساختن یک فلش bootable از آن    

- ابتدا به [صفحه‌ی دانلود آرچ](https://archlinux.org/download/) مراجعه کنید و فایل ایزو را به همراه signature مربوطه دانلود کنید
- سپس با استفاده از [GnuPG](https://wiki.archlinux.org/title/GnuPG) فایل ایزوی دانلود شده رو با این روش `تائید امضا` بررسی کنید:  
> gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig   

 <details dir="rtl">
  <summary>تائید امضا</summary>

>  مطابق اطلاعات [این لینک](https://www2.cs.arizona.edu/stork/packagemanagersecurity/attacks-on-package-managers.html)
  در صورتی که پکیج‌ها بروز رسانی نشوند امکان آلوده شدن آنها به بدافزارها افزایش پیدا می‌کند
  بدین جهت پکیج‌مینجرها طوری طراحی شدند تا پکیج‌ها(بسته‌ها) را به صورت خودکار بروزرسانی کنند
  اما در صورتی که خود این پکیج‌منیجرها از قبل آلوده باشند امکان اینکه مجددا بسته‌های آلوده به سیستم اضافه کنند یا بسته‌های آلوده‌ی قدیمی را درست بروزرسانی نکنند وجود خواهد داشت.
  پکیج‌منیجرها معمولا با دسترسی‌های بالا و بدون محدودیت اجرا می‌شوند تا قادر به اعمال تغییرات و بروزرسانی در سطوح مختلف سیستم باشند؛
  بنابراین، اقدامات مدیر بسته بر کل سیستم تأثیر می گذارد  
  تکنیک‌های مثل man in the middle یا سایر روش‌ها ممکنه استفاده شده باشن و برای مثال وب سایت توزیع مینت در سالهای دورتر مورد چنین حمله‌های قرار گرفته بود و فایل ایزو که روی سایت 
  بود و ملت دانلود می‌کردن درواقع آلوده بود و خب مدتی بعد گندش در اومد. خلاصه اینکه این مرحله رو یکم جدی بگیرید به نفع خودتون هست

 </details>

- بوت‌ایبل کرن فلش    
  برای اینکار روشهای زیادی وجود داره و برنامه‌های گرافیکی هم هستن که کار رو ساده‌تر کردن اما روش ترمینالی اینکار استفاده از دستور زیر هست:  
   ‍‍
> dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress  

   - برای اطلاعات بیشتر می‌تونید [این لینک](https://wiki.archlinux.org/title/USB_flash_installation_medium#Using_the_ISO_as_is_(BIOS_and_UEFI)) رو مطالعه کنید
  
 <details dir="rtl">
  <summary>خطاها‌ی زمان بوت شدن</summary>  

  ۱- ممکنه سیستم شما به Secure Boot حساس باشه و در نتیجه نتونه از روش فلش بوت بشه پس باید این وضعیت رو از منوی بایوس خاموش کنید
  برای اینکار سیستم رو ری‌استارت کنید و دکمه‌ی f12 یا f8 یا f2 رو بزنید تا وارد منوی بایوس بشین بین تب‌های موجود حرکت کنید و این وضعیت رو خاموش کنید

  ۲- ممکنه سیستم شما روی حالت uefi قرار نداشته باشه و درنتیجه بازم نمیتونه از روی فلش بوت بشه پس از منوی بایوس این حالت رو فعال کنید

  ۳- خطای *No Bootable Device*  
  بعد از نصب سیستم و تموم شدن کار اگر ری‌استارت کردین و خطایی دیدن که سیستم نتونسته یک فایل بوت پیدا کنه بازم باید وارد منوی بایوس بشین و
  از روی گزینه‌ای که برای انتخاب Trusted bootloaders هست وارد بشین و فایل بگردین فایل bootx64.efi رو انتخاب کنید براش یک اسم دلخواه بذارین و تنظمیات رو ذخیره کنید و ری‌استارت کنید
  
 </details>

- برای اینکه مطمئن بشیم فلش ما به صورت efi بوت شده دستور زیر رو میزنیم:  

> ls /sys/firmware/efi/efivars

  اگر این دستور خروجی نداشته باشه یعنی درحال حاضر این فلش به صورت efi بوت‌ایبل و درنتیجه بوت نشده پس باید دوباره مراحل قبلی رو طی کنید

---

## اتصال به اینترنت

برای اتصال به اینرنت یا از کابل استفاده می‌کنید که خب نیاز به تنظیمات یا کارخاصی ندارید؛ یا از وای‌فای میخواین وصل بشین که روشی که توضیح میدم از ویکی آرچ اقتباس شده و همین رو پیش میریم:
  - دستور `ip link` رو بزنید تا ببینید دیوایس‌های سیستم شناخته شدن و فعال هستن  
  - دستور ‍`iwctl` رو بزنید تا وارد محیط اینتراکتیوش بشید. بعد از اینکه وارد محیط اینتراکتیوش بشین یک تغییر کوچیک توی ترمینال می‌بینید به این شکل:  
> [iwd]#
  - میتونید دستور `help` رو بزنید تا راهنمای این ابزار براتون بیاد و مطالعه کنید
  - برای اینکه دیوایس وایرلس سیستم رو پیدا کنید دستور `device list` رو بزنید
  - برای اینکه این دیوایس بتونه وای‌فای‌های اطراف پیدا کنه دستور `station device scan` که در اینجا کلمه‌ی *device* رو باید اسم دیوایس وایرلس سیستم خودتون عوض کنید
  - با دستور `station device get-networks` که دوباره کلمه‌ی دیواس رو باید اصلاح کنید، می‌بینید که چه وای‌فای‌هایی رو پیدا کرده
  - حالا با دستور `station device connect SSID` می‌تونید به وای‌فای مدنظرتون وصل بشین.(SSID = اسم وای‌فای، برای اسم وای‌فای چند کاراکتر اول رو بنویسید و دکمه تب کیبورد رو بزنید.)
  - اگر نیاز به پسورد برای وای‌فای باشه یک پرامپت باز میشه تا پسورد رو بگیره،‌ همچنین میتونید یکجا و در یک دستور بزنید(نیازی به گفتن نداره که `passphrase` دومی هم همون پسورد وای‌فای هست)  
> iwctl --passphrase passphrase station device connect SSID  
  - برای اطلاعات بیشتر به [این لینک](https://wiki.archlinux.org/title/Iwd#iwctl) از ویکی آرچ مراجعه کنید
  - با دستور پینگ چک کنید که به اینترنت وصل شدین یا نه
> ping -c 4 archlinux.org
  - ساعت سیستم رو تنظیم کنید `timedatectl set-ntp true`
---

## پارتیشن بندی دیسک

این قسمت شاید یکم با بقیه‌ی راهنماهای نصب آرچ فرق کنه و البته پارتیشن بندی یکم حواس جمع میخواد، که یهو نزنید دیسک رو فرمت کنید و فایلهاتون رو پاک کنید و وا ویلا نشه!  

  - دستور `fdisk -l‍` یا `lsblk` برای شناسایی درایوهای سیستم بزنید
  - حداقل دو تا پارتیشن لازم داریم. یکی برای efi و بوت شدن سیستم و یکی هم برای خود سیستم‌عامل و درایوهای هوم و غیره،‌ از ابزار `cfdisk‍` یا `fdsik` برای پارتیشن بندی استفاده کنید. طرز کار ساده‌ای دارن، ابزارهای دیگه هم فرقی نداره استفاده بشن و مهم خروجی کار هست. نهایتا در خروجی کار همچین چیزی لازمه:

| partition  |       size        | filesystem-type| mount point |          partition          |
|   :---:    |       :---:       |     :---:      |     :---:   |            :---:            |
| boot       |      512 MiB      |   EFI system   | `/mnt/boot` | `/dev/efi_system_partition` |
| [SWAP]	    | More than 512 MiB |   Linux swap   |  `swapon`   | `/dev/swap_partition`       |
| root       |  remaining space  | Linux file sys | `/mnt`      | `/dev/root_partition`       |

  - ساختن پارتیشن `swap` الزامی نیست بعدا هم می‌تونید اضافه کنید! تصمیم گیری با خودتون 
  <details dir="rtl">
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
 
   - بنظرم [اینجا](https://forum.endeavouros.com/t/swapfile-vs-swap-partition-vs-no-swap-at-all/10932/6) خوب خلاصمون کرد
  - یک نکته‌ای هم دوستانی که می‌خوان از هایبرنیت استفاده کنن مدنظر داشته باشن که این عمل هرچند سرعت بوت شدن رو بیشتر می‌کنه(روی ssd که زیاد محسوس نیست) و ممکنه 
  از فوایدش خوشتون بیاد ولی در عمل بسیار بیشتر از بوت شدن معمولی به سیستم فشار میاد مخصوصا به SSD ها. حالا اینکه یکی میگه ssd خریدم که ازش استفاده کنم حالش رو ببرم
  پس هایبرنیت هم می‌کنم. یا یکی میگه خب این هایبرنیت که برام مهم نیست پس الکی سخت‌افزارم رو خراب نکنم نظراتی هست که هرکدوم محترم هست  
  - نکته‌ای که بنظرم حائز اهمیت هست و بهتره درموردش تحقیق کنید اینه که هاردهای SSD برخلاف هاردهای HDD عمر مفیدشون به عملکرد نوشتن و پاک‌کردن وابسته هست. بدین صورت که در 
  هاردهای قدیمی‌تر و نوری یا HDD خراب شدن اینها به خاطر خراب شدن هدشون بود و عملا نوشتن یا پاک‌کردن تاثیری رو اون دیسک نوری نمی‌ذاشت اما در SSD ها تعداد دفعات نوشتن و پاک‌کردن
  اثر مستقیم روی عمر قطعه داره و در همین راستا هرچی بتونیم عملیات‌های نوشتن و پاک‌کردن کمتری روی SSD تحمیل کنیم عمر مفیدش بیشتر میشه. این قضیه برای نسل‌های اولیه SSD ها 
  حیاتی بود چون واقعا و در عمل خیلی زود پر می‌شدن و می‌سوختن، اما نسل‌های جدیدتر خیلی بهتر شدن و از طرفی سیستم‌عامل‌ها و حتی سخت‌افزارها هم برای کار با SSD ها ارتقا پیدا کردن
  و شاید این ترس امروز خیلی کمتر حس بشه، اما بد نبود که درموردش بنویسم    
  - البته در مراحل پارتیشن بندی از یک سری فلگ برای fstab استفاده می‌کنم که در راستای بهینه‌تر مصرف کردن SSD باشه و عمر مفید بیشتری براش باقی بمونه
 
 </details>
 
  <details dir="rtl">
    <summary>برای پارتیشن بندی من از `fdisk` استفاده می‌کنم </summary>  

   - اول دستور `fdisk /dev/sdX` رو میزنیم تا وارد محیط این ابزار بشیم(اگر از ssd های nvme استفاده میکنید بجای sdX باید nvme0n1 یا همچین چیزی رو بزنید[تب بزنید خودش کامل میکنه] 
   و در غیر این صورت هم باید `X` رو با حرفی که برای پارتیشن مدنظرتون هست عوض کنید)  
   -   با نوشتن علامت سوال `m` و زدن اینتر (نوشتن دستور فلان و زدن اینتر رو خلاصه کنیم، با زدن دستور فلان) لیست راهنمای این ابزار باز میشه که میتونید در هر مرحله بهش روجوع کنید  
   - با زدن دستور`g` جدول بندی دیسک رو به حالت GPT تغییر میده که برای نصب ما این حالت لازمه    
   - با دستور `n` یک پارتیشن جدید معرفی می‌کنیم که اولین پارتیشن رو برای بوت منیجر درنظر می‌گیریم و حجم یک گیگ و فایل سیستم efi رو براش درنظر میگیریم
   درواقع اول n رو می‌زنیم، برای سوال بعدیش دیافلتش که ۱ هست کافیه و فقط اینتر رو می‌زنیم، برای سوال بعدیش هم فقط اینتر رو میزنیم، برای سوال بعدی علامت + رو بذارید و 512M رو 
   بدون فاصله بعدش بنویسد، که یعنی یک گیگ براش حجم ببره(میشه +1G هم نوشت.) و اینتر رو میزنیم تا پارتیشن اول ایجاد بشه    
   - با دستور `p` میتونیم چک کنیم که پارتیشن مدنظر ایجاد شده و سایزش هم درسته، می‌تونیم با دستور `d` هم پارتیشنی رو حذف کنیم      
   -  مجدد روال رو طی می‌کنیم تا پارتیشن‌های بیشتری که لازم هستن رو بسازیم مطابق جدول بالا. دستور `n` رو میزنیم. سوال اول و دوم رو فقط اینتر می‌زنیم، 
   و سوال چالشی اینجاست! برای swap چقدر حجم ببریم؟ این واقعا جواب‌های مختلفی داره ولی برای هایبرنیت حجم رم + ۲ گیگ برای اطمینان از از دست نرفتن دیتا و برای سایر سیستم‌ها 
   اگر رم سیستم کمتر از ۲ گیگ هست یک گیگ و اگر بین ۲ تا ۴ گیگ هست ۲ گیگ و برای ۴ گیگ و بیشتر هم ۴ گیگ مقادیر مرسومی هستن، ولی همونطور که گفته شد اگر ویرچوآل مموری
   بیشتری لازم دارین مقدارش رو بیشتر درنظر بگیرید. ضمن اینکه می‌تونید کلا این مرحله رو رد بشین و swap نسازید  
   - برای آخرین پارتیشن که قراره روت و هوم و سایر پارتیشن‌ها رو داخلش و با استفاده از subvolume بسازیم. مجدد دستور `n` رو می‌زنیم، دو سوال اول رو فقط اینتر می‌زنیم و این دفعه 
   سوال سوم هم اینتر می‌زنیم چون همه‌ی حجم باقی‌مونده رو میخوایم بدیم به این پارتیشن  
   - دستور `p` رو یکبار دیگه بزنیم تا مطمئن بشیم همه چیز همونطور که لازمه درست شده  
   - دستور `t` رو می‌زنیم تا تایپ پارتیشن‌ها رو هم مشخص کنیم، در جواب سوال اول عدد ۱ رو می‌زنیم و برای سوال دوم هم مجدد عدد ۱ رو می‌دیم تا پارتیشن از تایپ efi باشه
   مجددا دستور `t` رو میزنیم و این‌بار عدد ۲ رو می‌دیم تا تایپ پارتیشن دومی رو مشخص کنیم، به سوال بعدیش عدد ۱۹ رو میدیم و تایپ linux swap رو انتخاب می‌کنیم
   و برای پارتیشن بعدی هم پیش‌فرض خودش درسته و نیازی به تغییر نداره
   - دستور `w` رو می‌زنیم تا تغییرات روی دیسک رایت بشه و از fdisk خارج بشیم

 </details>

## ساختن فایل سیستم    
  -  اول پارتیشن بوت منیجر رو فرمت می‌کنیم، دستور زیر رو بزنید   
> mkfs.fat -F 32 -n EFI /dev/efi_system_partition   # efi_system_partition = /dev/sda1 or /dev/nvme0n1p1    
  - برای پارتیشن swap دستور  
> ‍mkswap -L SWAP /dev/your_drive2  # your_drive2 = /dev/sda2 or /dev/nvme0n1p2  
  - برای پارتیشن اصلی هم دستورات رو می‌زنیم
> mkfs.btrfs -f -L ROOT /dev/your_drive3 # your_drive3 = /dev/sda3 or /dev/nvme0n1p3  
  - از اینجا به بعد من از nvme0n1p3 استفاده می‌کنم، حواستون باشه که nvme0n1p3 رو باید با اسم درایو مربوطه روی سیستم خودتون جایگزین کنید

 <details dir="rtl">
  <summary>درباره‌ی فایل سیستم btrfs بخونیم</summary> 

 درواقع این فایل سیستم زیاد قدیمی نیست و حتی بعضیام میگن که استیبل نشده، ولی خب توزیع‌های زیادی رفتن سراغش و امکانات جالبی هم داره. برای مثال میتونه یک جور LVM بسازه که volume رو میتونه روی چند دیوایس یا هارددیسک پیاده سازی و یا subvolume هایی که روی یک volume میسازیم هم درواقع پوزیکس فایلهایی میشن که هرچند واقعا درایو یا پارتیشن نیستن ولی در ادامه می‌بینیم که ما اونها رو mount می‌کنیم. از مزایای این ساب‌ولیوم‌ها اینه که از فضای ولیومی که روشن قرار دارن اشتراکی استفاده می‌کنن و درواقع دیگه نیازی نیست نگران این باشیم که وای درایو فلانم داره پر میشه از یکی دیگه حجم جدا کنم به این اضافه کنم. از دیگر مزایای فایل سیستم btrfs میشه به قابلیت فشرده سازی و compress داده‌ها اشاره کرد که اینم در ادامه می‌بینیم چجوریه و مزیتش اینه که عملیات نوشتن و پاک کردن روی هارد کمتر میشه و روی defragmentaion هم تاثیر میذاره ولی خوب یا بد بودن این اثرگذاری رو باید بررسی کنیم.  
یک مورد خوب دیگه هم استفاده از اسنپ‌شات‌هایی بسیار کم حجم هست به طوری که اسنپ‌شات‌ها یجوری شبیه گیت فقط تغییرات رو ثبت می‌کنن و کلی ایمیج سنگین از کل پارتیشن‌های ما نمیسازن. btrfs این امکان رو میده تا تنظیم کنیم برای چه پارتیشن‌ها و حالا که بلدیم ساب‌ولیوم‌هایی اتوماتیک اسنپ‌شات تولید بشه یا اصلا تو اسنپ‌شات اونارو لحاظ نکنه و [امکانات بیشتر](https://btrfs.readthedocs.io/en/latest/index.html)  
 
 </details>

 <details dir="rtl">
  <summary>درباره‌ی LVM بخونیم</summary>
 
  مفهومی به اسم LVM یا Logical Volume Management در نیای کامپیوتر وجود دارد، قبل از این مفهوم سیستم عامل دیسک‌های حافظه را به صورت یک بسته‌ی مستقل که روی آن پارتیشن‌هایی وجود داشت یا خود سیستم‌عامل روی آن ایجاد می‌کرد می‌شناخت؛ در این صورت ما در آخرین لایه‌ی سیستمی در فضای پارتیشن‌های به ظرفیت هاردیسک فیزیکی محدود بودیم و حداکثر فضایی که برای یک پارتیشن میشد متصور بود برابر با حداکثر ظریفیت یک هارد دیسک بود. اما LVM این محدودیت را از بین می‌برد، به این صورت که هارددیسک‌های فیزیکی که به صورت خلاصه PV(Physical Volume) نامیده می‌شوند در یک گروه قرار می‌گیرند و تشکیل یک Volume Group یا به اختصار VG می‌دهند. هر VG به صورت یک دیسک درنظر گرفته می‌شود درحالیکه خود ممکن است از یک یا چند PV یعنی دیسک‌های فیزیکی تشکیل شده باشد. بر روی یک VG که از نظر سیستم‌عامل حالا یک دیسک حافظه هست(`/dev/sda`) گروه‌هایی از Logical Volume ها ایجاد می‌شود. به Logical Volume اختصارا LV گفته می‌شود و می‌توان آن را مشابه با `/dev/sda1` فرض کرد که ما می‌توانیم فایل‌سیستم‌های خود مثل ext4, ntfs و... را روی آن‌ها ایجاد کنیم و داده‌هایمان را روی آنها بریزیم. LVM درواقع مثل یک پرده‌ی پوشاننده روی هارددیسک‌های فیزیکی قرار می‌گیرد و آنها را از دید سیستم‌عامل مخفی می‌کند و VG را به عنوان هارددیسک جا می‌زند درنتیجه می‌توان تعداد هاردیسک‌های فیزیکی سیستم را کم یا زیاد کرد ولی سیستم‌عامل از این تغییر آگاه نباشد، یا حافظه‌های اختصاص یافته به VG ها یا LV ها را به صورت داینامیک و پویاتری تغییر داد و در مدیریت حافظه بسیار دست بازی‌تری به ما می‌دهد. 
 </details>

 <details dir="rtl">
  <summary>درباره‌ی Compression بخونیم</summary>
 
  در فایل سیستم BtrFS یا ‌B-Tree File System می‌توان بدون فشرده‌سازی یا با یکی از سه الگوریتم مختلف برای فشرده‌سازی دیتا استفاده کرد، ZLIB، LZO و ZSTD که مطابق با بنچمارک‌ها ZLIB پرفورمنس و سرعت عمل کمتری داشته و ZSTD در چند مورد از LZO بهتر بوده است. اما برای مثال توزیع مانجارو در نصب به صورت BTRFS از الگوریتم LZO استفاده کرده است(ممکن است در نسخه‌های دیگرش این الگوریتم را عوض کند). اما من تصمیم گرفتم درمورد الگوریتم ZSTD بنویسم. این الگوریتم دارای چندین سطح فشرده سازی هست که مطابق با مستنداتش از سطح ۱-۳ فشرده‌سازی سبک‌تر و کمتری انجام می‌دهد و به اصطلاح real-time هست، این یعنی سرعت عمل بیشتر و همین طور فشار کمتری به پردازنده(cpu) و از سطح ۴-۹ که فشرده‌سازی جدی‌تری انجام می‌دهد و طبعا کمی سرعت کمتر و همچنین سربار بیشتر برای پردازنده و از سطح ۱۰-۱۵ هم جهت تلاش بر نهایت توان فشرده‌سازی که البته شاید تاثیری به‌سزایی در خروجی نداشته باشد ولی قطعا افت سرعت و همچین لود سنگین‌تری روی پردازنده خواهد داشت. به صورت پیش‌فرض درصورتی که خاصیت فشرده‌سازی zstd برای این فایل سیستم فعال کنیم در سطح ۳ از آن استفاده می‌کند. ولی خب من ترجیح می‌دهم سطح ۸ را انتخاب کنم، همانطور که پیشتر اشاره کردم فشرده کردن دیتا در عمل نوشتن و پاک‌کردن از روی هارددیسک تاثیر مفیدی دارد و برداشت من از مستندات این الگوریتم این است که انتخاب سطح ۹ حد وسط هست. همچنین مطابق اطلاعاتی که از ویکی آرچ بدست آوردم اعمال فشرده سازی اجباری هرچند قدری پردازنده باز اضافی تحمیل می‌کند اما در نهایت برای فشرده‌سازی بهینه‌تر خواهد بود چرا که درحالت عادی ممکن است برای بعضی از داده‌ها فشرده‌سازی صرف‌نظر شود و سیستم با تشخصی خود از اینها بگذرد. به صورت پیش فرض و فقط برای فعال کردن فشرده سازی کافی‌است پاریتشن مدنظر را با دستوری مشابه دستوری زیر مانت کنیم  

> mount -o compress=zstd /dev/sdx /mnt  
 
 اطلاعات بیشتر و لینک‌های بنچمارک‌ها و سایر اطلاعات رو از [ویکی آرچ](https://wiki.archlinux.org/title/Btrfs#Compression) بخونید  
 مطابق تصویر زیر من یک تست گرفتم روی سیستم مانجارو که دارم از روش این مطلب رو می‌نویسم و می‌بینید سرعت فشرده‌سازی و از فشرده‌سازی درآوردن برای سطوح تست شده چقدر بوده
 
 ![Screenshot_20220822_003822](https://user-images.githubusercontent.com/44068054/185809006-1012ca80-f0ef-4661-b073-da02fac4d513.png)
  
 
  </details>
 
 - بریم سراغ ساختن ساب‌ولیوم‌ها، اول باید پارتیشن روت رو مانت کنیم و بعد ساب‌ولیوم‌ها رو روش بسازیم  
> mount /dev/nvme0n1p3 /mnt
 
 - به این ساب‌ولیوم‌ها نیاز داریم
> @, @home, @log, @pkg, @tmp, @snapshots

<details dir="rtl">
  <summary>ساب‌ولیوم‌ها</summary>
 
 - `@` – This is the main root subvolume /.
 - `@home` – This is the home directory. This consists of most of your data including Desktop and Downloads.
 - `@log` – Contains logs, temp. files, caches, games, etc.
 - `@pkg` – Contains all the pacman packages
 - `@tmp` – Contains certain temporory files and caches
 - `@snapshots` – Directory to store snapshots.
 
</details>
 
 - این ها هم دستوراتی که استفاده می‌کنیم و اختصارشون
>  su = subvolume
>  cr = create
>  li = list
 
 - ساختن ساب‌ولیوم‌ها
  btrfs su cr /mnt/@
  btrfs su cr /mnt/@home
  btrfs su cr /mnt/@root
  btrfs su cr /mnt/@srv
  btrfs su cr /mnt/@log
  btrfs su cr /mnt/@cache
  btrfs su cr /mnt/@tmp
  btrfs su li /mnt
 
