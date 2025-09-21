## Overview

Every Vultr server (instance) has a default **main public IP** which cannot be removed or replaced. To manage IP addresses flexibly, including changing IPs without downtime, use **Reserved IPs**.

A **Reserved IP** is an additional public IPv4 address you can create and manage via Vultr's API. By routing services through a Reserved IP, you can switch IPs programmatically without affecting your server’s main IP.

---

## Workflow Summary

1. **Generate an API Key**: Create an API key with sufficient permissions in Vultr dashboard.
2. **Create a Reserved IP**: Buy/create a Reserved IP in the same region as your server.
3. **Attach Reserved IP**: Link the Reserved IP to your server.
4. **Configure your services**: Update your apps to use the Reserved IP.
5. **Change IP later**:
   - Detach the current Reserved IP from the server.
   - Attach a new Reserved IP.
6. **Delete Reserved IP**: Release the Reserved IP if no longer needed.

---

## 1. Generate an API Key

- Go to Vultr dashboard > Account > API.
- Generate or reuse an API key.
- Save it securely; it won’t be shown again.

---

## 2. List Instances (Servers)

```bash
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
}

(Invoke-RestMethod -Uri "https://api.vultr.com/v2/instances" -Headers $headers -Method GET).instances | Format-List id, label, main_ip, region
```

Example output:

```
id      : abcdef12-3456-7890-abcd-ef1234567890
label   : my-server-1
main_ip : 192.0.2.1
region  : ewr

id      : 12345678-abcd-ef12-3456-7890abcdef12
label   : my-server-2
main_ip : 192.0.2.2
region  : ewr
```

---

## 3. Check Reserved IP Connection Status

```bash
$reservedIP = Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID" -Headers $headers -Method GET
$reservedIP.reserved_ip | Format-List id, region, ip_type, subnet, subnet_size, label, instance_id
```

Example output:

```
id          : RES_ID
region      : ewr
ip_type     : v4
subnet      : 192.0.2.100
subnet_size : 32
label       : My Reserved IP
instance_id : abcdef12-3456-7890-abcd-ef1234567890
```

If `instance_id` matches your server’s ID, the Reserved IP is attached.

---

## 4. Create (Purchase) a Reserved IP

```bash
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

---

## 5. Attach Reserved IP to a Server

```bash
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
    "Content-Type" = "application/json"
}

$body = @{
    instance_id = "YOUR_INSTANCE_ID"
} | ConvertTo-Json

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID/attach" -Method POST -Headers $headers -Body $body
```

Expected output: _Empty response means success._

---

## 6. Detach Reserved IP from a Server

```bash
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
}

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID/detach" -Method POST -Headers $headers
```

Empty response indicates success.

---

## 7. Delete Reserved IP (Release)

```bash
$headers = @{
    "Authorization" = "Bearer YOUR_API_KEY"
}

Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID" -Method DELETE -Headers $headers
```

Empty response indicates success. This stops billing for the Reserved IP.

---

## 8. Check Reserved IP Deletion

```bash
Invoke-RestMethod -Uri "https://api.vultr.com/v2/reserved-ips/RESERVED_IP_ID" -Headers $headers -Method GET
```

If the Reserved IP is deleted, response is an error such as:

```
error             error_type
-----             ----------
reserved ip not found
```

---

## Important Notes

- Main IP of your server **cannot** be changed or deleted.
- Reserved IPs are billed separately.
- Attach and detach Reserved IPs programmatically to avoid downtime.
- Always verify API response to catch errors.
- Replace all placeholders (`YOUR_API_KEY`, `RESERVED_IP_ID`, `YOUR_INSTANCE_ID`, `your-region-code`) with your actual values.

---

## Useful Links

- Vultr API Documentation: https://www.vultr.com/api/
- Reserved IPs Official Docs: https://docs.vultr.com/products/network/reserved-ips
- Detaching Reserved IP Guide: https://docs.vultr.com/products/network/reserved-ips/management/detachment
