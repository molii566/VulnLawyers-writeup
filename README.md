## Vuln Lawyers Pentest Report

## Client: VulnLawyers
## Date: July 23, 2025
## Testers: Tyler Ramsby Team
## Scope: carbon.ctfio.com (Web applications only)

## 1. Executive Summary

## This engagement simulated a real-world penetration test against VulnLawyers’ web application environment. 
Using open‑source intelligence (OSINT), subdomain enumeration, directory brute‑forcing, and targeted authentication testing, 
we identified multiple critical vulnerabilities, including IDOR and insufficient access controls, 
which allowed unauthorized disclosure of user credentials and case management manipulation.
Each finding includes a detailed description, proof of concept (PoC), and remediation recommendations.

## 2. Scope

**In‑Scope Assets:**

- Main application: `carbon.ctfio.com`
- Public domain: `vulnlawyers.co.uk`
- Subdomains:
  - `api.carbon.ctfio.com`
  - `data.carbon.ctfio.com`

## **Out‑of‑Scope:** Underlying infrastructure beyond DNS enumeration.

## 3. Methodology

1. **OSINT & Reconnaissance**
   - Researched public domains and DNS records.
2. **Subdomain Enumeration**
   - Utilized `ffuf` with SecLists wordlists to discover active subdomains.
3. **Directory & Endpoint Discovery**
   - Performed directory brute‑forcing against each host.
4. **Authentication Testing**
   - Inspected login workflows and POST/GET endpoints.
   - Brute‑forced credentials using `seclists` wordlists.
5. **Access Control & IDOR Testing**
   - Exploited insecure direct object references in profile endpoints.
6. **Privilege Escalation / Business Logic Abuse**
   - Leveraged valid credentials to modify or delete sensitive data.

---

## 4. Findings

### 4.1 OSINT & Subdomain Discovery

- Discovered two key subdomains:
  - `api.carbon.ctfio.com`
  - `data.carbon.ctfio.com`
- Querying API returned application metadata and a hidden flag:
  ```json
  {
    "name": "VulnLawyers Website API",
    "version": "2.1.04",
    "flag": "[^FLAG^E78DEBBFDFBEAFF1336B599B0724A530^FLAG^]"
  }
  ```

> **Flag #1:** E78DEBBFDFBEAFF1336B599B0724A530

### 4.2 Directory Enumeration

| Path            | Status | Notes                                      |
| --------------- | ------ | ------------------------------------------ |
| `/css/`         | 301    | Redirects to `/css/`                       |
| `/images/`      | 301    | Redirects to `/images/`                    |
| `/js/`          | 301    | Redirects to `/js/`                        |
| `/denied`       | 401    | Access restricted                          |
| `/login`        | 302    | Redirects to `/denied` initially           |
| `/lawyers-only` | 200    | Staff portal discovered on redirected link |

- `/login` GET revealed a hidden link and informational flag:

```html
Access to this portal can now be found here <a href="/lawyers-only">/lawyers-only</a>
Flag: [^FLAG^FB52470E40F47559EBA87252B2D4CF67^FLAG^]
```

> **Flag #2:** FB52470E40F47559EBA87252B2D4CF67

### 4.3 User Enumeration & Authentication

- Accessed `data.carbon.ctfio.com/users` endpoint:

```json
{
  "users": [
    {"id":0,"name":"Yusef Mcclain","email":"yusef.mcclain@vulnlawyers.ctf"},
    {"id":1,"name":"Shayne Cairns","email":"shayne.cairns@vulnlawyers.ctf"},
    {"id":2,"name":"Eisa Evans","email":"eisa.evans@vulnlawyers.ctf"},
    {"id":3,"name":"Jaskaran Lowe","email":"jaskaran.lowe@vulnlawyers.ctf"},
    {"id":4,"name":"Marsha Blankenship","email":"marsha.blankenship@vulnlawyers.ctf"}
  ],
  "flag":"[^FLAG^25032EB0D322F7330182507FBAA1A55F^FLAG^]"
}
```

> **Flag #3:** 25032EB0D322F7330182507FBAA1A55F

- Brute‑forced passwords against each email; discovered `jaskaran.lowe@vulnlawyers.ctf:summer`.
- Logged in to Jaskaran’s account and retrieved first staff flag on `/lawyers-only` dashboard:

```html
Staff Portal
[^FLAG^7F1ED1F306FC4E3399CEE15DF4B0AE3C^FLAG^]
```

> **Flag #4:** 7F1ED1F306FC4E3399CEE15DF4B0AE3C

### 4.4 IDOR & Sensitive Data Exposure

- POSTing updates to `/lawyers-only-profile` appeared normal,
- but GETting `/lawyers-only-profile-details/{id}` returned cleartext passwords when changing the `id` parameter.
- Changed `id=2` to retrieve Shayne Cairns’s credentials:

```json
{
  "id":2,
  "name":"Shayne Cairns",
  "email":"shayne.cairns@vulnlawyers.ctf",
  "password":"q2V944&#2a1^3p",
  "flag":"[^FLAG^938F5DC109A1E9B4FF3E3E92D29A56B3^FLAG^]"
}
```

> **Flag #5:** 938F5DC109A1E9B4FF3E3E92D29A56B3

### 4.5 Privilege Abuse & Case Deletion

- Logged in as Shayne Cairns, navigated to `Current Cases`. Authorized to delete cases.
- Deleted the `Evil Corp Vs Jones Animal Charity` case, triggering final flag:

> **Flag #6:** [^FLAG^B38BAE0B8B804FCB85C730F10B3B5CB5^FLAG^]

---

## 5. Remediation Recommendations

1. **Implement Proper Access Controls:** Validate user authorization on each profile endpoint; ensure ID parameters are matched to authenticated user context.
2. **Encrypt Sensitive Data:** Never return plaintext passwords or sensitive fields in API responses.
3. **Secure Directory Listings:** Restrict or remove unused directories and enforce consistent response codes.
4. **Rate Limit & Monitor:** Apply brute‑force protections on authentication endpoints.
5. **Audit Logging:** Log critical actions (e.g., profile updates, case deletions) with sufficient context for forensic analysis.

---

## 6. Conclusion

This assessment uncovered critical flaws in access control and data exposure within VulnLawyers’ staff portal. 
Addressing these issues will significantly strengthen the organization’s security posture and prevent unauthorized access or data leakage. 
Continuous security reviews and adherence to secure coding practices are recommended.

## Author: Mo Lii
## (LinkedIn:](https://www.linkedin.com/in/mahamud-abdirahman-151493375/)
