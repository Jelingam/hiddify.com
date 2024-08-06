# تونل 4to6

تونل 4to6 مشابه تونل 6to4 است با مزایای زیر:

1. آدرس IP مؤثر شما بین دو سرور IPv6 خواهد بود.
2. IPv6 ارزان‌تر است و در صورت مسدود شدن به راحتی قابل تغییر است.
3. آدرس‌های IPv4 شما در هر دو طرف از مسدود شدن محافظت خواهند شد.

## نمای فنی

این تنظیم شامل دو نود، A و B است:

```
Tehran ------------------GFW----------------- Tokyo 
   A <---------------------------------------> B
```

یک تونل بین این دو نود ایجاد می‌شود:

```
Tehran ------------------------------------------GFW----------------------------------------- Tokyo 
   A                                                                                          B
| virtual IPv4 <---- 4to6 ----> real IPv6 | <------IPv6-----> | real IPv6 <---- 4to6 ----> virtual IPv4 |
```

بسته‌های IPv4 در بسته‌های IPv6 کپسوله شده و از طریق شبکه IPv6 منتقل می‌شوند. اگر یک IP مسدود شود، یک آدرس IPv6 خواهد بود که به راحتی می‌توان آن را جایگزین کرد، زیرا آدرس‌های IPv6 معمولاً از IPv4 ارزان‌تر هستند.

## نحوه استفاده

شما به دو سرور با IPv6 نیاز دارید.

#### سرور A

```bash
sudo curl -s https://raw.githubusercontent.com/meshya/4to6-tunnel/main/scripts/install.sh | bash
```

| فیلد | مقدار |
|------|-------|
| E0   | IPv6 سرور A |
| E2   | IPv6 سرور B |
| E4   | 192.168.1.1/24 |

#### سرور B

```bash
sudo curl -s https://raw.githubusercontent.com/meshya/4to6-tunnel/main/scripts/install.sh | bash
```

| فیلد | مقدار |
|------|-------|
| E0   | IPv6 سرور B |
| E2   | IPv6 سرور A |
| E4   | 192.168.1.2/24 |

## تست

#### روی سرور A

```bash
ping 192.168.1.2
```

#### روی سرور B

```bash
ping 192.168.1.1
```

## هشدار

اگر یکی از سرورهای شما از شبکه 192.168.x.x استفاده می‌کند، ممکن است نیاز به استفاده از 172.16.x.x/12 (مثلاً 172.16.0.1/12 و 172.16.0.2/12) داشته باشید. اگر مطمئن نیستید، یک issue باز کنید و خروجی `ip addr` را شامل کنید.

## گزینه‌های استفاده

سه روش برای استفاده از تونل وجود دارد:

1. IP/Port Forwarding
2. استفاده از Xray در هر دو سرور
3. استفاده از روش‌های تونلینگ سفارشی

### IP/Port Forwarding

برای کسانی که با IP/Port Forwarding آشنا نیستند، توصیه می‌شود بیشتر در این زمینه مطالعه کنند و تنظیمات را به صورت دستی انجام دهند.

[این مقاله](https://tecadmin.net/setting-up-a-port-forwarding-using-iptables-in-linux/) راهنمایی برای تنظیم Port Forwarding ارائه می‌دهد.

در اینجا یک راهنمای تنظیم سریع آمده است:

برای فوروارد کردن یک پورت واحد (مثلاً پورت 80)، از تنظیمات زیر استفاده کنید (فرض می‌شود IPهای مجازی 192.168.1.1 برای سرور A و 192.168.1.2 برای سرور B هستند).

#### مرحله 1

#سرور A
```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.2:80 
iptables -t nat -A POSTROUTING -j MASQUERADE 
```

#### مرحله 2

اگر تنظیمات کار کرد، دستورات را به `local.rc` اضافه کنید.

### استفاده از Xray در هر دو سرور

خدمات Xray را روی هر دو سرور نصب کنید و سرور A را طوری تنظیم کنید که داده‌ها را از طریق پراکسی روی سرور B ارسال کند. این تنظیمات با استفاده از 3x-ui ساده‌تر می‌شود. برای کمک، با من تماس بگیرید (من به عنوان meshya در همه جا و meshyah در تلگرام در دسترس هستم).

نکته: از 192.168.1.2 به جای IP واقعی سرور B در هنگام تنظیم Xray روی سرور A استفاده کنید.

### استفاده از روش‌های تونلینگ سفارشی

اگرچه توصیه نمی‌شود، می‌توانید از روش‌های تونلینگ سفارشی مانند [Reverse TLS](https://github.com/radkesvat/ReverseTlsTunnel) یا [Fake TLS](https://github.com/radkesvat/FakeTlsTunnel) روی تونل 4to6 استفاده کنید اگر با این تکنیک‌ها آشنا هستید.

نکته: از IPهای مجازی (192.168.x.x) به جای IPهای واقعی استفاده کنید.

## نکته

هنگام استفاده از [روش 2](#use-xray-on-both-servers)، قوانین مسیریابی را تنظیم کنید و از [پروژه دامنه‌های میزبانی شده در ایران](https://github.com/bootmortis/iran-hosted-domains) استفاده کنید تا ترافیک را برای وب‌سایت‌ها و خدمات میزبانی شده در ایران، به خصوص شاپرک و خدمات پرداخت، از یک IP ایرانی هدایت کنید. این تنظیم باعث می‌شود کاربران نیازی به تغییر پراکسی خود نداشته باشند.

## منابع

1. [تونل 4to6 - Meshya](https://github.com/meshya/4to6-tunnel)

2. [مستندات پروژه لینوکس - تونل‌سازی ip4 در ip6](https://tldp.org/HOWTO/Linux+IPv6-HOWTO/ch10.html)

3. [RFC2437](http://www.faqs.org/rfcs/rfc2473.html)

<br>
<br>
<br>

مقاله و مخزن اصلی توسط [Meshya](https://github.com/meshya/4to6-tunnel)