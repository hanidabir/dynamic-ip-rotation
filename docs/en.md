## Overview

Every Vultr server (instance) comes with a default **main public IP** which cannot be removed or replaced. However, to achieve flexible IP management, including changing the IP address attached to your server without downtime, you can use **Reserved IPs**.

A **Reserved IP** is an additional public IPv4 address that you can purchase, attach to a server, and detach as needed via Vultr's API. By routing your services through the Reserved IP, you gain the ability to switch IPs programmatically without affecting your server’s main IP.

***

## Workflow Summary

1. **Reserved IP purchase and creation**: Buy/create a Reserved IP in the same region as your server.
2. **Attach Reserved IP**: Link this Reserved IP to your server.
3. **Configure your services**: Update your application or server configurations to use the Reserved IP instead of the main IP.
4. **To change IP later**:  
   - Detach the current Reserved IP from the server via API.  
   - Attach a new Reserved IP via API.  
   - Your services continue clicking to the new Reserved IP with no downtime related to main IP changes.

***

## Detailed Steps with Vultr API

### 1. Generate an API Key

- Go to your Vultr dashboard > Account > API.  
- Generate a new API key or use an existing one.  
- Ensure the key has sufficient permissions to manage Reserved IPs and servers.

***

### 2. Create (Purchase) a Reserved IP

Send a POST request to create a new Reserved IP in the desired region:

```bash
curl -X POST "https://api.vultr.com/v2/reserved-ips" \
    -H "Authorization: Bearer {YOUR_API_KEY}" \
    -H "Content-Type: application/json" \
    -d '{
          "region": "your-region-code",
          "ip_type": "v4",
          "label": "My Reserved IP"
        }'
```

- Replace `"your-region-code"` with your server’s region (e.g., `ewr`, `fra`, `sfo`).
- The response will include the `reserved_ip_id` to use in subsequent API calls.

***

### 3. Attach Reserved IP to Your Server

Attach the Reserved IP to your server instance to make it active:

```bash
curl -X POST "https://api.vultr.com/v2/reserved-ips/{reserved_ip_id}/attach" \
    -H "Authorization: Bearer {YOUR_API_KEY}" \
    -H "Content-Type: application/json" \
    -d '{
          "instance_id": "your-instance-id"
        }'
```

- Replace `{reserved_ip_id}` with the ID from the Reserved IP creation step.
- Replace `"your-instance-id"` with your server’s ID.

***

### 4. Detach Reserved IP from Server

If you want to change IPs later, detach the current Reserved IP from your server:

```bash
curl -X POST "https://api.vultr.com/v2/reserved-ips/{reserved_ip_id}/detach" \
    -H "Authorization: Bearer {YOUR_API_KEY}"
```

***

### 5. Delete/Release Reserved IP (Optional)

If you want to permanently release the Reserved IP so you no longer pay for it:

```bash
curl -X DELETE "https://api.vultr.com/v2/reserved-ips/{reserved_ip_id}" \
    -H "Authorization: Bearer {YOUR_API_KEY}"
```

***

## Important Notes

- **Main IP cannot be removed:** Your server always has a permanent main IP which cannot be replaced or deleted.
- **Reserved IP comes with additional cost** and is billed separately.
- Use Reserved IPs to configure your applications (e.g., bind web server, database listeners, firewall rules) to ensure all traffic flows via the Reserved IP.
- Changing IPs through Reserved IP attach/detach does not affect your server uptime or require server restarts.
- Always check API responses for errors after each request.

***

## Useful Links

- Vultr API Documentation: https://www.vultr.com/api/
- Reserved IPs Official Docs: https://docs.vultr.com/products/network/reserved-ips
- Detaching Reserved IP Guide: https://docs.vultr.com/products/network/reserved-ips/management/detachment
