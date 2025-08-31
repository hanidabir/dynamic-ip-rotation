# Managing Reserved IP Addresses on Vultr Using API

Basic steps:
1. Obtain an API key from the Vultr API page.
2. Create a Reserved IP in the server's region.
3. Attach the Reserved IP to the server to activate it.

<br />

When you need to change the Reserved IP:
1. Detach the current Reserved IP from the server.
2. (Optional) Delete the old Reserved IP to stop billing.
3. Create a new Reserved IP in the same region.
4. Attach the new Reserved IP to the server.

<br />

How to do these?  
[English Version](https://github.com/hanidabir/static-ip-rotation/blob/main/docs/en.md) | [نسخه فارسی](https://github.com/hanidabir/static-ip-rotation/blob/main/docs/fa.md)

---

**Notes:**

- The server's main IP is permanent and cannot be removed.  
- Configure your services to use the Reserved IP.  
- All these actions are performed via API without manual intervention.
