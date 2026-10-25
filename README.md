# Custom DNS Client–Server Application
### Computer Networks Project — Slot: F1 | Team: Team 13

A high-performance **Custom DNS Server & Client System** implementing:
- **Custom Application-Layer DNS Protocol** (UDP DatagramSockets, pipe-delimited)
- **3-Tier Name Resolution Hierarchy**: In-Memory TTL Cache → MongoDB Atlas Cloud History → Local Authoritative Database (`dns_records.txt`) → Upstream Fallback
- **TTL-Based Caching Engine**: Per-record Time-to-Live, active eviction, sub-millisecond hot lookups
- **Persistent Cloud Telemetry**: Real-time logging & atomic search-frequency tracking with **MongoDB Atlas**
- **Dual Interface Architecture**: Native **Java Swing Desktop GUI** + Modern **Web Dashboard** (HTML5/CSS3/JavaScript)
- **Offline Fault Tolerance**: Automatic CSV persistence fallback when offline

---

## VTOP Submission Details

| Item | Requirement | File / Link |
| :--- | :--- | :--- |
| **Assignment 8** | Project Report (PDF) | [`docs/F1_Team13_Report.pdf`](docs/F1_Team13_Report.pdf) |
| **Assignment 9** | Project Presentation (PPTX) | [`docs/F1_Team13_PPT.pptx`](docs/F1_Team13_PPT.pptx) |
| **Assignment 10** | GitHub Repository URL | [**https://github.com/adithyahanuman/DNS-Client**](https://github.com/adithyahanuman/DNS-Client) |
| **Source Archive** | Full Source Code ZIP | [`docs/F1_Team13_SourceCode.zip`](docs/F1_Team13_SourceCode.zip) |
| **Screenshots** | Functionality Screenshots ZIP | [`docs/F1_Team13_Screenshots.zip`](docs/F1_Team13_Screenshots.zip) |

---

## Team Details (Slot: F1 — Team 13)
- **Narne Adithya Hanuman**
- **G Sreemanth**
- **V. Umika**

---

## Project Structure

```
CustomDNS/
├── DNSServer.java          ← UDP DNS Server (port 5454)
├── DNSClient.java          ← CLI DNS Client
├── DNSHandler.java         ← Per-request thread handler
├── DNSCache.java           ← In-memory TTL cache
├── DNSGui.java             ← Swing Desktop GUI (all-in-one)
├── DNSWebServer.java       ← HTTP API server (port 8090)
├── MongoHistoryService.java← MongoDB Atlas + CSV persistence
├── SearchRecord.java       ← Model: search history entry
├── DnsQueryRecord.java     ← Model: individual query log
├── ShowHistory.java        ← CLI history viewer
├── CacheStressTest.java    ← Cache benchmark/stress test
├── ChromeHistoryImporter.java ← Import Chrome browser history
├── MongoDnsTest.java       ← MongoDB connection tester
├── dns_records.txt         ← Local DNS records (domain=ip)
├── dns_search_history.csv  ← CSV fallback for history
├── dns_queries.csv         ← CSV fallback for query log
├── index.html              ← Web dashboard frontend
├── style.css               ← Web dashboard styles
├── app.js                  ← Web dashboard JavaScript
├── .env                    ← MongoDB connection config  ← EDIT THIS
├── .env.example            ← Template
├── pom.xml                 ← Maven build file
└── lib/                    ← Pre-bundled JARs
    ├── mongodb-driver-sync-4.11.1.jar
    ├── mongodb-driver-core-4.11.1.jar
    ├── bson-4.11.1.jar
    ├── slf4j-api-1.7.36.jar
    ├── slf4j-simple-1.7.36.jar
    └── sqlite-jdbc-3.45.1.0.jar
```

---

## Step 1 — Set Up MongoDB Atlas (Free)

> Skip to Step 2 if you already have a MongoDB URI.

### 1.1 Create a Free Account
1. Go to **https://www.mongodb.com/cloud/atlas/register**
2. Sign up with Google / email — it's free
3. Choose **"Free Shared" (M0)** cluster → click **Create**
4. Select any cloud provider (AWS/GCP/Azure) and any region near you

### 1.2 Create a Database User
1. Left menu → **Database Access** → **Add New Database User**
2. Authentication: **Password**
3. Username: e.g. `dns_user`
4. Password: click **Autogenerate** → **Copy** it now
5. Built-in Role: **Read and write to any database**
6. Click **Add User**

### 1.3 Allow Your IP Address
1. Left menu → **Network Access** → **Add IP Address**
2. Click **Allow Access from Anywhere** (or add your specific IP)
3. Click **Confirm**

### 1.4 Get Your Connection String
1. Left menu → **Database** → click **Connect** on your cluster
2. Choose **"Drivers"**
3. Driver: **Java**, Version: **4.3 or later**
4. Copy the connection string — it looks like:
   ```
   mongodb+srv://dns_user:<password>@cluster0.abcde.mongodb.net/?retryWrites=true&w=majority&appName=CustomDNS
   ```
5. Replace `<password>` with your actual password from step 1.2

### 1.5 Paste into `.env`
Open [`.env`](.env) and replace the `MONGODB_URI` line:
```ini
MONGODB_URI=mongodb+srv://dns_user:YOUR_PASSWORD@cluster0.abcde.mongodb.net/?retryWrites=true&w=majority&appName=CustomDNS
MONGODB_DATABASE=custom_dns
MONGODB_COLLECTION=search_history
```

---

## Step 2 — Compile the Project

Open a terminal in the project folder (`CustomDNS/`) and run:

```powershell
cd "c:\Users\narne\Desktop\computer networks\CustomDNS\CustomDNS"
javac -cp ".;lib/*" *.java
```

You should see no errors. `.class` files will be created.

---

## Step 3 — Run the Application

### Option A — Swing GUI (Recommended, All-in-One)
```powershell
java -cp ".;lib/*" DNSGui
```
This opens a desktop window with:
- **Server Tab** — Start/Stop the DNS server, see live logs, cache table
- **Client Tab** — Look up any domain, see results and history

### Option B — Command Line (Two Terminals)

**Terminal 1 — Start DNS Server:**
```powershell
java -cp ".;lib/*" DNSServer
```
Expected output:
```
════════════════════════════════════════════
          CUSTOM DNS SERVER
════════════════════════════════════════════
Loading DNS records from: dns_records.txt
  Loaded: google.com              → 142.250.195.14
  ...
[MongoDB] Successfully connected to custom_dns.search_history
Server started successfully
IP   : 192.168.x.x
Port : 5454
Waiting for requests...
```

**Terminal 2 — Start DNS Client:**
```powershell
java -cp ".;lib/*" DNSClient
```
Then type any domain name:
```
Enter domain name: google.com
  Domain       : google.com
  IP Address   : 142.250.195.14
  Resolution   : [DATABASE LOOKUP] (cached)
  TTL / Info   : 60s
  Response Time: 12 ms
  Status       : RESOLVED [OK]
```

### Option C — Web Dashboard
**Terminal 1** — start the DNS server (see above)  
**Terminal 2** — start the web server:
```powershell
java -cp ".;lib/*" DNSWebServer
```
Then open your browser: **http://localhost:8090**

---

## Other Useful Commands

### View Search History (CLI)
```powershell
java -cp ".;lib/*" ShowHistory
```

### Run Cache Stress Test / Benchmark
```powershell
java -cp ".;lib/*" CacheStressTest
```

### Test MongoDB Connection Only
```powershell
java -cp ".;lib/*" MongoDnsTest
```

### Import Chrome Browser History
```powershell
java -cp ".;lib/*" ChromeHistoryImporter
```

---

## Custom DNS Protocol

| Message | Format |
|---------|--------|
| Request | `DNS\|REQUEST\|<domain>` |
| Success | `DNS\|RESPONSE\|<domain>\|<ip>\|<source>\|<ttl>` |
| Error   | `DNS\|ERROR\|<domain>\|<reason>` |

**Sources:** `CACHE` / `MONGODB` / `DNS_LOOKUP` / `REAL_DNS`  
**Errors:** `DOMAIN_NOT_FOUND` / `INVALID_REQUEST_FORMAT` / `INVALID_DOMAIN_NAME`

---

## Resolution Tiers (in order)

```
Query arrives
    │
    ├─ 1. Check DNSCache (in-memory, TTL-based)  ──► CACHE HIT → return instantly
    │
    ├─ 2. Check MongoDB search_history            ──► MONGODB HIT → cache + return
    │
    ├─ 3. Check dns_records.txt (local file)      ──► FOUND → save to MongoDB + cache + return
    │
    └─ 4. Real DNS lookup (InetAddress)           ──► RESOLVED → save to MongoDB + cache + return
              │
              └─ If all fail → DNS|ERROR|<domain>|DOMAIN_NOT_FOUND
```

---

## Configuration

| Setting | File | Default |
|---------|------|---------|
| MongoDB URI | `.env` → `MONGODB_URI` | `mongodb://localhost:27017` |
| Database name | `.env` → `MONGODB_DATABASE` | `custom_dns` |
| Collection | `.env` → `MONGODB_COLLECTION` | `search_history` |
| DNS Server port | `DNSServer.java` line 43 | `5454` |
| Cache TTL | `DNSCache.java` line 30 | `60 seconds` |
| HTTP dashboard port | `DNSWebServer.java` line 30 | `8090` |
| UDP timeout (client) | `DNSClient.java` line 46 | `5000 ms` |

---

## Pre-loaded DNS Records

Defined in [`dns_records.txt`](dns_records.txt):

| Domain | IP |
|--------|----|
| google.com | 142.250.195.14 |
| github.com | 140.82.121.4 |
| openai.com | 104.18.32.47 |
| stackoverflow.com | 151.101.193.69 |
| wikipedia.org | 208.80.154.224 |
| amazon.com | 176.32.103.205 |
| microsoft.com | 20.112.52.29 |
| facebook.com | 157.240.221.35 |
| localhost | 127.0.0.1 |
| server.local | 192.168.1.10 |
| ... | ... |

Add more by editing `dns_records.txt` with the format `domain=ip` (one per line).

---

## Requirements

- **Java 17** or later
- **Internet access** (for MongoDB Atlas + real DNS fallback lookups)
- All library JARs are pre-bundled in `lib/` — no Maven install needed
