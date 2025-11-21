# 🏠 Home PC Setup Guide - Betfair MCP Server

**Purpose:** Run betfair-mcp server from your home computer (non-German IP)  
**Why:** Avoid Germany GEO-blocking on VPS  
**Status:** ✅ Recommended deployment method  
**Last Updated:** 2025-11-21 22:57:56

---

## 🌍 WHY HOME PC?

### ✅ Non-German IP = NO GEO-BLOCKING
```
Your Home (your home country) → Betfair API:
  ├─ IP detected: your home country ✅
  ├─ GEO-blocking: NONE ✅
  ├─ Account APIs: Working ✅
  └─ Market APIs: Working (with LIVE key) ✅
```

### ❌ VPS Germany = BLOCKED
```
VPS (Falkenstein, Germany) → Betfair API:
  ├─ IP detected: Germany ❌
  ├─ GEO-blocking: ACTIVE ❌
  ├─ Account APIs: Working ✅
  └─ Market APIs: BLOCKED ❌
```

**Official Betfair Statement:**
> "The Betfair API does not return market data for users accessing the service from Germany or German-based IP addresses due to legal restrictions."

**Source:** https://support.developer.betfair.com/hc/en-us/articles/360004831131

---

## 📋 PREREQUISITES

### Required:
- ✅ **Operating System:** Windows 10/11, macOS 10.15+, or Linux
- ✅ **Python:** Version 3.10 or higher
- ✅ **Internet:** Stable connection (non-German IP)
- ✅ **Betfair Account:** Active and verified
- ✅ **SSL Certificates:** Generated from Betfair (see below)
- ✅ **Claude Desktop** or **Claude Code CLI**

### Optional:
- Git (for cloning repository)
- Virtual environment manager (venv/conda)

---

## 🔐 STEP 1: SSL CERTIFICATES SETUP

### 1.1 Generate SSL Certificate (one-time)

**On your home PC (Windows/Mac/Linux):**

```bash
# Generate 2048-bit RSA key pair
openssl genrsa -out client-2048.key 2048

# Generate certificate signing request (CSR)
openssl req -new -key client-2048.key -out client-2048.csr

# When prompted, enter:
# Country: (your country code)
# State/Province: (your region)
# Locality: (your city)
# Organization: (your name or leave blank)
# Common Name: (your email)

# Generate self-signed certificate (10 years)
openssl x509 -req -days 3650 -in client-2048.csr \
  -signkey client-2048.key -out client-2048.crt
```

### 1.2 Upload Certificate to Betfair

1. Open: https://myaccount.betfair.com/accountdetails/mysecurity?showAPI=1
2. Login to your Betfair account
3. Find "Automated Bot Traders (API)" section
4. Click "Upload Certificate"
5. Open `client-2048.crt` in text editor
6. Copy entire contents (including BEGIN/END lines)
7. Paste into Betfair upload form
8. Submit

**Verification:** Certificate status should show "Active"

### 1.3 Organize Certificates

```bash
# Create certificates directory
mkdir ~/betfair_certs

# Move certificates
mv client-2048.key ~/betfair_certs/
mv client-2048.crt ~/betfair_certs/
mv client-2048.csr ~/betfair_certs/  # backup

# Set correct permissions (Linux/Mac)
chmod 600 ~/betfair_certs/client-2048.key  # CRITICAL!
chmod 644 ~/betfair_certs/client-2048.crt
```

**Windows:** Right-click key file → Properties → Security → Limit to your user only

---

## 💾 STEP 2: INSTALL BETFAIR-MCP

### 2.1 Download Repository

**Option A: Git Clone (recommended)**
```bash
# Clone from GitHub (after repo is created)
git clone https://github.com/yourusername/extgange-betfair-mcp.git
cd betfair-mcp-server
```

**Option B: Download ZIP**
1. Go to GitHub repository
2. Click "Code" → "Download ZIP"
3. Extract to desired location
4. Open terminal in extracted folder

### 2.2 Create Virtual Environment

```bash
# Navigate to project directory
cd betfair-mcp-server  # (or your extracted folder)

# Create virtual environment
python3 -m venv .venv

# Activate virtual environment
# Linux/Mac:
source .venv/bin/activate

# Windows (Command Prompt):
.venv\Scripts\activate.bat

# Windows (PowerShell):
.venv\Scripts\Activate.ps1
```

### 2.3 Install Dependencies

```bash
# Make sure venv is activated (you should see (.venv) in prompt)

# Install required packages
pip install fastmcp betfairlightweight aiolimiter python-dotenv

# Verify installation
pip list | grep -E "(fastmcp|betfairlightweight|aiolimiter)"
```

**Expected Output:**
```
aiolimiter      1.1.0
betfairlightweight  2.20.0
fastmcp         2.12.5
```

---

## 🔧 STEP 3: CONFIGURATION

### 3.1 Create .env File

```bash
# Copy example file
cp .env.example .env

# Edit .env file
nano .env  # (or use your text editor)
```

**Add your credentials:**
```bash
BETFAIR_USERNAME=your_betfair_username
BETFAIR_PASSWORD=your_betfair_password
BETFAIR_APP_KEY=1.0-DELAY  # or 1.0-LIVE (after activation)
BETFAIR_CERTS_PATH=/absolute/path/to/betfair_certs

# Example (Windows):
# BETFAIR_CERTS_PATH=C:/Users/YourName/betfair_certs

# Example (Linux/Mac):
# BETFAIR_CERTS_PATH=/home/username/betfair_certs
```

**IMPORTANT:** Use **absolute paths**, not relative!

### 3.2 Find Your App Key

1. Go to: https://apps.betfair.com/visualisers/api-ng-account-operations/
2. Login to Betfair
3. Select operation: `getDeveloperAppKeys`
4. Enter Session Token (or refresh page after login)
5. Click "Execute"
6. Copy `appKey` value from results

**You'll see TWO keys:**
- `1.0-DELAY` - Free, delayed data (for testing)
- `1.0-LIVE` - £299, real-time data (for production)

**Start with DELAY key for testing!**

---

## 🖥️ STEP 4: CLAUDE DESKTOP CONFIGURATION

### 4.1 Locate Config File

**Mac:**
```bash
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Windows:**
```
%APPDATA%\Claude\claude_desktop_config.json
```

**Linux:**
```bash
~/.config/Claude/claude_desktop_config.json
```

### 4.2 Add MCP Server Configuration

**Open config file** (create if doesn't exist) and add:

```json
{
  "mcpServers": {
    "betfair-mcp": {
      "command": "/absolute/path/to/.venv/bin/python",
      "args": ["-m", "betfair_mcp.server"],
      "env": {
        "BETFAIR_USERNAME": "your_username",
        "BETFAIR_PASSWORD": "your_password",
        "BETFAIR_APP_KEY": "1.0-DELAY",
        "BETFAIR_CERTS_PATH": "/absolute/path/to/betfair_certs"
      }
    }
  }
}
```

**Replace placeholders:**
- `/absolute/path/to/.venv/bin/python` → Your Python path
- Your credentials
- Your certs path

**Windows example:**
```json
{
  "mcpServers": {
    "betfair-mcp": {
      "command": "C:\\Users\\YourName\\betfair-mcp-server\\.venv\\Scripts\\python.exe",
      "args": ["-m", "betfair_mcp.server"],
      "env": {
        "BETFAIR_USERNAME": "your_username",
        "BETFAIR_PASSWORD": "SecurePass123",
        "BETFAIR_APP_KEY": "1.0-DELAY",
        "BETFAIR_CERTS_PATH": "C:\\Users\\YourName\\betfair_certs"
      }
    }
  }
}
```

**Mac/Linux example:**
```json
{
  "mcpServers": {
    "betfair-mcp": {
      "command": "/home/yourname/betfair-mcp-server/.venv/bin/python",
      "args": ["-m", "betfair_mcp.server"],
      "env": {
        "BETFAIR_USERNAME": "your_username",
        "BETFAIR_PASSWORD": "SecurePass123",
        "BETFAIR_APP_KEY": "1.0-DELAY",
        "BETFAIR_CERTS_PATH": "/home/yourname/betfair_certs"
      }
    }
  }
}
```

### 4.3 Restart Claude Desktop

```bash
# Close Claude Desktop completely
# Reopen Claude Desktop

# MCP server will start automatically in background
```

---

## ✅ STEP 5: VERIFICATION

### 5.1 Test Account APIs

**Open Claude Desktop chat:**

```
Call betfair_get_account_balance
```

**Expected output:**
```
Account Balance:
Available to Bet: £XXX.XX
Current Exposure: £0.00
Wallet: UK
```

**If this works → Authentication is OK! ✅**

### 5.2 Test Events APIs (DELAY Key)

```
Call betfair_list_event_types
```

**Expected output with DELAY key:**
```
Event Types:
No event types found.
```

**This is NORMAL with DELAY key!** 
- DELAY key blocks programmatic API access to events/markets
- Account APIs work fine ✅
- Events/Markets APIs return empty ❌

**To get full data → Activate LIVE key (£299)**

### 5.3 Verify No GEO-Blocking

If you see empty results, it's because of:
1. ✅ DELAY key limitation (expected)
2. ❌ NOT GEO-blocking (you're on Non-German IP!)

**With LIVE key, you WILL see full data!** 🎉

---

## 🔓 STEP 6: ACTIVATE LIVE KEY (Optional - £299)

### When to activate LIVE key?
- ✅ After successful DELAY key testing
- ✅ When you need real-time market data
- ✅ For production betting workflows
- ✅ To unlock all 7 MCP tools

### How to activate:

1. **Apply for activation:**
   - Go to: https://developer.betfair.com/get-started/
   - Select "Exchange API > For My Personal Betting"
   - Complete application form
   - Wait for approval (usually 24-48 hours)

2. **Pay activation fee:**
   - £299 one-time fee
   - Debited from your Betfair account balance
   - Ensure account is funded

3. **Update configuration:**
   ```bash
   # Edit .env file
   nano .env
   
   # Change:
   BETFAIR_APP_KEY=1.0-LIVE  # (use your LIVE key value)
   ```

4. **Restart Claude Desktop**

5. **Test again:**
   ```
   Call betfair_list_event_types
   ```
   
   **Expected with LIVE key:**
   ```
   Event Types:
   1. Soccer (ID: 1) - 150 markets
   2. Tennis (ID: 2) - 80 markets
   3. Horse Racing (ID: 7) - 200 markets
   ...
   ```

   **SUCCESS! Full data! 🎉**

---

## 🔧 TROUBLESHOOTING

### Issue: "Missing required environment variables"

**Cause:** .env file not loaded or wrong paths

**Fix:**
1. Verify .env file exists in project root
2. Check all variables are set (no empty values)
3. Use absolute paths (not relative)
4. Restart Claude Desktop

---

### Issue: "CERT_AUTH_REQUIRED"

**Cause:** SSL certificates not found or wrong permissions

**Fix:**
1. Verify certificates exist in BETFAIR_CERTS_PATH
2. Check permissions: `chmod 600 client-2048.key`
3. Verify certificate is uploaded to Betfair
4. Use absolute path in config

---

### Issue: "LOGIN_FAILED" or "INVALID_SESSION_TOKEN"

**Cause:** Wrong username/password

**Fix:**
1. Verify credentials in .env file
2. Test login on betfair.com manually
3. Check for typos in username/password
4. Ensure account is verified (KYC)

---

### Issue: Empty event types (with LIVE key)

**Cause:** Probably not activated correctly

**Fix:**
1. Verify LIVE key is "Active" in Betfair portal
2. Wait 24 hours after activation
3. Check account balance (£299 was debited?)
4. Contact Betfair support if persists

---

### Issue: Python not found

**Cause:** Wrong Python path in config

**Fix:**
```bash
# Find Python path
which python  # Linux/Mac
where python  # Windows

# Use full path in claude_desktop_config.json
# Example: /usr/bin/python3 or C:\Python312\python.exe
```

---

## 📊 EXPECTED RESULTS MATRIX

| Tool | DELAY Key | LIVE Key (£299) |
|------|-----------|-----------------|  
| **betfair_get_account_balance** | ✅ Works | ✅ Works |
| **betfair_get_account_details** | ✅ Works | ✅ Works |
| **betfair_list_event_types** | ❌ Empty | ✅ Full data |
| **betfair_list_events** | ❌ Empty | ✅ Full data |
| **betfair_list_competitions** | ❌ Empty | ✅ Full data |
| **betfair_list_market_catalogue** | ❌ Empty | ✅ Full data |
| **betfair_get_market_prices** | ❌ Empty | ✅ Full data |

**Summary:**
- DELAY key: 2/7 tools working (28%)
- LIVE key: 7/7 tools working (100%) ✅

---

## 🎯 SUCCESS CHECKLIST

After setup, you should have:
- [x] SSL certificates generated and uploaded
- [x] Virtual environment created and activated
- [x] Dependencies installed (FastMCP, betfairlightweight)
- [x] .env file configured with credentials
- [x] Claude Desktop config updated
- [x] Account APIs returning data
- [x] No GEO-blocking errors (Non-German IP!)
- [ ] LIVE key activated (optional, £299)
- [ ] All 7 tools returning data (with LIVE key)

---

## 📚 ADDITIONAL RESOURCES

- **Betfair Developer Portal:** https://developer.betfair.com/
- **API Documentation:** https://betfair-developer-docs.atlassian.net/
- **SSL Certificate Help:** https://myaccount.betfair.com/accountdetails/mysecurity?showAPI=1
- **App Keys:** https://apps.betfair.com/visualisers/api-ng-account-operations/
- **GEO-blocking Info:** https://support.developer.betfair.com/hc/en-us/articles/360004831131

---

## 💡 TIPS & BEST PRACTICES

1. **Keep certificates secure:**
   - Never commit to Git
   - Set restrictive permissions (600)
   - Backup in secure location

2. **Use .env for credentials:**
   - Never hardcode in config files
   - Add .env to .gitignore
   - Use different keys for dev/prod

3. **Monitor your usage:**
   - Check Betfair account balance
   - Watch for rate limiting (95 req/min)
   - Review API logs regularly

4. **Start with DELAY key:**
   - Test everything first
   - Verify no GEO-blocking
   - Then decide on LIVE key

5. **Home PC considerations:**
   - Keep computer running during usage
   - Stable internet connection required
   - Consider static IP or dynamic DNS

---

## 🆘 SUPPORT

**Issues with setup?**
1. Review troubleshooting section above
2. Check all prerequisites met
3. Verify credentials are correct
4. Test Betfair login manually

**Issues with Betfair API?**
1. Check Betfair status page
2. Verify account is active and verified
3. Contact Betfair support

**Issues with MCP server?**
1. Check server logs in Claude Desktop
2. Test manual server start: `python -m betfair_mcp.server`
3. Review MASTER_README.md for known issues

---

**Setup Guide Version:** 1.0  
**Last Updated:** 2025-11-21 22:57:56  
**Maintained By:** Project Contributors

**Good luck! 🚀**