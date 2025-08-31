## مقدمه

هر سرور (Instance) در Vultr یک آدرس **main IP** اصلی دارد که غیرقابل حذف یا تغییر است؛ بنابراین، برای تغییر خودکار آدرس IP متصل به سرور با API، باید از **Reserved IP** استفاده کنیم.
آدرسی که باید جداگانه خریده و آن را به سرور وصل (attach) یا جدا (detach) کنیم تا کنترل کامل IP سرور در دست ما باشد.

***

## خلاصه روند کار

1. **خرید و ساخت Reserved IP** در منطقه سرور خودتان
2. **اتصال Reserved IP به سرور**  
3. **تنظیم سرویس‌ها روی Reserved IP** (به جای main IP)  
4. **برای تغییر IP در آینده:**  
   - ابتدا Reserved IP فعلی را با API جدا کنید.
   - سپس، Reserved IP جدید را با API وصل کنید.
   - سرور اصلی و main IP بدون تغییر باقی می‌مانند و سرویس‌ها از طریق Reserved IP جدید ادامه می‌یابند.

***

## مراحل کامل استفاده از API Vultr

### ۱. گرفتن API Key

- وارد حساب Vultr شوید، بخش Account > API.
- یک API Key جدید بسازید یا کلید موجود را کپی کنید.
- مطمئن شوید کلید دسترسی به مدیریت Reserved IP و سرورها را دارد.

***

### ۲. ساخت (خرید) Reserved IP جدید

درخواستی با دستور زیر برای ساخت Reserved IP ارسال کنید:

```bash
curl -X POST "https://api.vultr.com/v2/reserved-ips" \
     -H "Authorization: Bearer {API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{
           "region": "کد-ناحیه-سرور",
           "ip_type": "v4",
           "label": "Reserved IP من"
         }'
```

- به جای `"کد-ناحیه-سرور"` منطقه سرور خود را قرار دهید (مثلاً `ewr` یا `sfo`).  
- در پاسخ، شناسه Reserved IP ساخته شده (reserved_ip_id) را دریافت خواهید کرد.

***

### ۳. اتصال Reserved IP به سرور

```bash
curl -X POST "https://api.vultr.com/v2/reserved-ips/{reserved_ip_id}/attach" \
     -H "Authorization: Bearer {API_KEY}" \
     -H "Content-Type: application/json" \
     -d '{
           "instance_id": "شناسه-سرور"
         }'
```

- شناسه Reserved IP را جایگزین `{reserved_ip_id}` کنید.  
- شناسه سروری که می‌خواهید IP به آن متصل شود را جایگزین `instance_id` نمایید.

***

### ۴. جدا کردن Reserved IP از سرور

برای جدا کردن IP فعلی هنگام تغییر:

```bash
curl -X POST "https://api.vultr.com/v2/reserved-ips/{reserved_ip_id}/detach" \
     -H "Authorization: Bearer {API_KEY}"
```

***

### ۵. آزادسازی (حذف) Reserved IP (اختیاری)

اگر می‌خواهید IP را آزاد کنید و هزینه آن دیگر پرداخت نشود:

```bash
curl -X DELETE "https://api.vultr.com/v2/reserved-ips/{reserved_ip_id}" \
     -H "Authorization: Bearer {API_KEY}"
```

***

## نکات مهم

- ‌به ازای هر سرور، main IP سرور همواره ثابت و غیرقابل حذف است.  
- هر Reserved IP، هزینه جداگانه دارد.  
- برای دسترسی به سرویس‌های خود از Reserved IP استفاده کنید (مثلاً تنظیمات وب‌سرور، فایروال و DNS).  
- تغییر IP با detach و attach Reserved IP کاملاً بدون قطعی سرویس انجام می‌شود.  
- بعد از هر درخواست API، پاسخ را بررسی کنید تا از موفقیت عمل آگاهی پیدا کنید.

***

## منابع و مستندات

- [مستندات رسمی Vultr API (انگلیسی)](https://www.vultr.com/api/)  
- [مستندات Reserved IP در Vultr](https://docs.vultr.com/products/network/reserved-ips)  
- [راهنمای جدا کردن Reserved IP](https://docs.vultr.com/products/network/reserved-ips/management/detachment)
