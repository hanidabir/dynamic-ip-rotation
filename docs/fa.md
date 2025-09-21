## معرفی کلی

هر سرور Vultr به طور پیش‌فرض یک **IP عمومی اصلی** دارد که قابل حذف یا تغییر نیست. برای مدیریت بهتر و ایجاد انعطاف‌پذیری در آدرس‌های IP سرور، از **Reserved IPها** استفاده می‌شود.

یک **Reserved IP**، یک آدرس IPv4 عمومی اضافی است که می‌توانید آن را از طریق API خریداری، متصل و جدا کنید. با استفاده از Reserved IP می‌توانید بدون قطعی سرویس، IP اختصاصی سرور را تغییر دهید.

***

## مراحل کلی کار

1. **تولید کلید API**: یک کلید API با مجوزهای کافی بسازید.
2. **ساخت Reserved IP**: یک Reserved IP در منطقه سرور خود ایجاد کنید.
3. **وصل کردن Reserved IP**: آن را به سرور متصل کنید.
4. **پیکربندی سرویس‌ها**: نرم‌افزارها را روی IP جدید تنظیم کنید.
5. **برای تغییر IP بعدی**:
   - (در صورت نیاز) detach کردن Reserved IP فعلی از سرور
   - (در صورت نیاز) attach کردن Reserved IP جدید
6. **حذف Reserved IP**: اگر دیگر نیاز ندارید، پاک کنید.

***

## 1. تولید کلید API

- به داشبورد Vultr > حساب > API بروید.
- یک کلید جدید بسازید یا از کلید موجود استفاده کنید.
- کلید را امن نگه دارید.

***

## 2. گرفتن لیست سرورها (Instances)

```powershell
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
}

(Invoke-RestMethod -Uri "https://api.vultr.com/v2/instances" -Headers $headers -Method GET).instances | Format-List id, label, main_ip, region
```

نمونه خروجی:

```
id      : abcdef12-3456-7890-abcd-ef1234567890
label   : my-server-1
main_ip : 192.0.2.1
region  : ewr
```

***

## 3. بررسی وضعیت اتصال Reserved IP

```powershell
$reservedIP = Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID" -Headers $headers -Method GET
$reservedIP.reserved_ip | Format-List id, region, ip_type, subnet, subnet_size, label, instance_id
```

نمونه خروجی:

```
id          : RES_ID
region      : ewr
ip_type     : v4
subnet      : 192.0.2.100
subnet_size : 32
label       : My Reserved IP
instance_id : abcdef12-3456-7890-abcd-ef1234567890
```

اگر `instance_id` با شناسه سرور شما یکی باشد، یعنی Reserved IP متصل است.

***

## 4. ساخت Reserved IP

```powershell
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
    "Content-Type" = "application/json"
}

$body = @{
    region = "your-region-code"
    ip_type = "v4"
    label = "My Reserved IP"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips" -Method POST -Headers $headers -Body $body
```

***

## 5. وصل کردن Reserved IP به سرور

```powershell
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
    "Content-Type" = "application/json"
}

$body = @{
    instance_id = "YOUR_INSTANCE_ID"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID/attach" -Method POST -Headers $headers -Body $body
```

***

## 6. جدا کردن Reserved IP از سرور

```powershell
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
}

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID/detach" -Method POST -Headers $headers
```

***

## 7. حذف (Delete) Reserved IP

```powershell
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
}

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID" -Method DELETE -Headers $headers
```

***

## 8. بررسی حذف Reserved IP

```powershell
Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID" -Headers $headers -Method GET
```

اگر IP حذف شده باشد، پاسخ به صورت خطا و پیام مانند زیر خواهد بود:

```
error           error_type
-----           ----------
reserved ip not found
```

***

## نکات مهم

- هر سرور یک IP اصلی دارد که قابل حذف یا تغییر نیست.
- هر Reserved IP، هزینه اضافه در بر دارد.
- با attach/detach Reserved IP بدون قطعی قابل تغییر است.
- پاسخ API را بعد از هر درخواست حتما بررسی کنید.
- جایگزین کردن تمامی مقادیر `YOUR_API_KEY`، `RESERVED_IP_ID`، `YOUR_INSTANCE_ID` و `your-region-code` الزامی است.

***

## لینک‌های مفید

- مستندات API ولتر: https://www.vultr.com/api/
- مستندات Reserved IP: https://docs.vultr.com/products/network/reserved-ips
- راهنمای جداکردن Reserved IP: https://docs.vultr.com/products/network/reserved-ips/management/detachment
