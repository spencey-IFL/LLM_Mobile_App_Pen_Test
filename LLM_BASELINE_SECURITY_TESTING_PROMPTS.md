# LLM Prompts for Baseline Android Security Testing Coverage

**Instructions:** For baseline coverage, copy and paste each of the following sections into your LLM (Copilot, ChatGPT, Claude, etc.) with the relevant artifacts attached or pasted.

---

## 1. Android Manifest Analysis

### Prompt:
```
Analyze the attached AndroidManifest.xml for security vulnerabilities. 

Report on ALL instances and severity levels for:

CRITICAL ISSUES:
- debuggable="true"
- android:allowBackup="true"
- Exported Activities/Services/Broadcast Receivers without proper permissions
- Overpermissive permissions (e.g., INTERNET, READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE, READ_CONTACTS)
- Custom URI schemes (deeplinks) without proper validation
- System-level permissions (SYSTEM_ALERT_WINDOW, PACKAGE_USAGE_STATS)

HIGH SEVERITY ISSUES:
- minSdkVersion < 24 (API 24 is Android 7.0; recommend >= 31)
- targetSdkVersion < 31 (App may run with legacy permissions)
- Hardcoded API keys, Firebase keys, AWS keys, S3 bucket names
- Hardcoded URLs containing:
  - http:// (unencrypted)
  - Credentials (username:password@)
  - API endpoints with "admin", "internal", "secret"
  - Firebase database URLs (.firebaseio.com)
  - AWS service URLs (.amazonaws.com)
- Network Security Configuration: <domain-config cleartextTrafficPermitted="true">
- Network Security Configuration trusting user-installed CAs (<trust-anchors><certificates src="user"/>)

MEDIUM SEVERITY ISSUES:
- FileProvider with overly broad path specifications (android:path=".")
- Custom permissions with protectionLevel="normal" or not specified
- Receiver for android.intent.action.BOOT_COMPLETED
- Intent filters without categories or data restrictions

INFORMATIONAL:
- Any Base64-encoded strings that appear to be keys
- Commented-out code with credentials
- TODO/FIXME comments mentioning security
- Build-time secrets or environment variables
- Certificate pinning implementation (or lack thereof)
- ProGuard/R8 obfuscation configuration

For each finding:
1. Quote the exact line from the manifest
2. Explain the security impact
3. Provide a remediation recommendation

Finally, summarize: Total critical/high/medium issues found and overall risk assessment.
```

---

## 2. Source Code Security Review

### Prompt:
```
Review the provided source code for security vulnerabilities. This is a mobile/web application security assessment.

Scan for and report on:

AUTHENTICATION & SESSION MANAGEMENT:
- Hardcoded credentials (usernames, passwords, API keys, tokens)
- OAuth/OIDC implementation flaws (missing state parameter, no PKCE, weak redirect validation)
- Session token storage location (memory vs. disk vs. SharedPreferences vs. Keystore)
- Token expiration and refresh logic
- Multi-factor authentication bypass opportunities

CRYPTOGRAPHY & ENCODING:
- Use of weak/deprecated algorithms (MD5, SHA1, DES, RC4)
- Hardcoded encryption keys
- Insecure random number generation (Random instead of SecureRandom)
- Base64 encoding used as encryption (Base64 is encoding, not encryption)
- SSL/TLS pinning implementation (or absence of it)

DATA STORAGE:
- Sensitive data stored in:
  - SharedPreferences (unencrypted)
  - SQLite databases without encryption
  - Files in external storage
  - Logcat output
  - Memory without clearing
- File permissions (world-readable/writable)
- Backup exclusion for sensitive data

API & NETWORK:
- Unencrypted HTTP requests
- Missing or weak HTTP security headers (X-Frame-Options, Content-Security-Policy, etc.)
- Hardcoded API endpoints or server URLs
- No certificate pinning
- Insecure deserialization (ObjectInputStream, pickle, etc.)
- Request signing/HMAC validation

INPUT VALIDATION:
- Path traversal vulnerabilities
- SQL injection in queries
- Command injection
- XML/XXE injection
- Intent injection
- Insufficient input sanitization

AUTHORIZATION:
- Client-side-only authorization checks
- Missing server-side permission validation
- Direct object references (IDORs)
- Privilege escalation paths

LOGGING & ERROR HANDLING:
- Sensitive data logged (tokens, passwords, PII)
- Verbose error messages exposing internals
- Stack traces in responses
- Debug information left in production

For each finding:
1. Cite the file and line number(s)
2. Quote the vulnerable code
3. Explain the security impact
4. Provide a secure code example for remediation

Summarize the top 5 most critical vulnerabilities.
```

---

## 3. TruffleHog Secret Scanning

### Prompt:
```
I have run the following TruffleHog command:
trufflehog filesystem /path/to/source --json > trufflehog-results.json

The results have been saved to a JSON file. I am now providing you the JSON output.

Analyze the TruffleHog results and provide:

1. SUMMARY STATISTICS:
   - Total potential secrets detected
   - Count by type (API Key, Private Key, Password, Auth Token, AWS Key, etc.)
   - High-confidence vs. low-confidence findings
   - Verified secrets (if VerifyResult shows "verified": true)

2. DETAILED FINDINGS TABLE:
   Create a table with columns:
   - Secret Type
   - Confidence/Verified Status
   - Filepath
   - Line Number
   - Snippet (first 20 chars of the secret, redacted)
   - Remediation Status (Active/Rotated/Mitigated)

3. SEVERITY ASSESSMENT:
   For each finding marked as "verified" or high confidence:
   - Explain the security impact
   - Assess which systems or credentials are at risk
   - Recommend immediate remediation steps

4. PROCESS IMPROVEMENTS:
   - Suggest .gitignore rules to prevent future leaks
   - Recommend secret management tools (HashiCorp Vault, AWS Secrets Manager, etc.)
   - Suggest pre-commit hooks to prevent commits with secrets

5. FALSE POSITIVES:
   - Flag any low-confidence findings that appear to be false positives
   - Explain why they're likely benign

Finally, create a summary of "VERIFIED SECRETS REQUIRING IMMEDIATE ACTION" and provide rotation/remediation guidance for each.
```

---

## 4. Logcat Analysis

### Prompt:
```
I will now provide you with a logcat dump from an Android device. This was captured while a user logged in to the app and exercised all major features.

Please analyze the logcat output for the following security issues:

AUTHENTICATION & CREDENTIALS:
- Cleartext passwords, usernames, or email addresses
- Session tokens, JWT tokens, or Bearer tokens
- OAuth tokens or refresh tokens
- MFA codes or recovery codes
- API keys or secret keys
- Any Base64-encoded tokens that can be decoded

SENSITIVE DATA EXPOSURE:
- Personally Identifiable Information (PII): names, email, phone, addresses, SSN, passport, tax ID
- Financial data: credit card numbers (even partial), bank account numbers, transaction details
- Health information
- Location data with high precision
- Biometric data
- Usernames tied to sensitive actions

ANDROID-SPECIFIC ISSUES:
- Device identifiers (IMEI, Serial Number, Android ID)
- Package installation history
- WiFi network names (SSIDs)
- Bluetooth device names/addresses
- File paths exposing internal app structure

IMPLEMENTATION FLAWS:
- URLs with embedded credentials (http://user:pass@host)
- Unencrypted data transmission indicators
- SQL queries with user input (SQL injection indicators)
- Stack traces or exception details
- Database schema information
- Hardcoded server addresses or endpoints
- Debug/test mode enabled in production
- Feature flags or rollout percentages

For each finding:
1. Quote the exact log line
2. Timestamp of the log entry
3. Process/Package name
4. Severity (Critical/High/Medium/Low)
5. Remediation recommendation

Summarize:
- Total sensitive data leaks found
- Most critical data exposed
- Root causes and patterns
- Recommended fixes
```

---

## 5. Frida & Objection Runtime Analysis

### Prompt:
```
I have set up Frida Server on an Android device and am using Objection for interactive runtime analysis.

Guide me through a comprehensive runtime security assessment using Objection. For each step, explain what security vulnerabilities we're testing for:

STEP 1: ENVIRONMENT MAPPING
Commands:
- env (environment variables)
- env | grep -i "key\|secret\|token\|pass\|api\|auth\|db\|host\|url"

Analysis:
- Are sensitive configuration files stored in the app's private data directory (/data/data/[package]/) or world-accessible locations?
- Do environment variables contain credentials or secrets?
- Are there any hardcoded paths to sensitive resources?

STEP 2: MEMORY ANALYSIS
Commands:
- memory dump all

Analysis:
- Search the dump for: API keys, tokens, passwords, usernames, email addresses
- Look for encryption keys or key derivation material
- Check for cached authentication credentials
- Identify session tokens or refresh tokens
- Search for OAuth/OIDC material (code_verifier, access_token, id_token, refresh_token)
- Look for Base64-encoded secrets that can be decoded
- Identify what data is retained in memory after logout

STEP 3: KEYSTORE INSPECTION
Commands:
- android keystore list

Analysis:
- What keys are stored and for what purpose?
- Are keys protected with Strong Box or Trusted Execution Environment (TEE)?
- Can keys be exported (if so, the keystore is inadequately protected)?
- Are encryption keys used for sensitive data storage?
- Is PIN/biometric protection enforced?

STEP 4: DATABASE ACCESS
Commands:
- android shell
- sqlite3 /data/data/[package]/databases/*.db

Analysis:
- What databases exist and what data do they contain?
- Is sensitive data (passwords, tokens, PII, financial data) stored unencrypted?
- Is encryption used? (SQLCipher, Android Keystore-backed encryption)
- Can we extract and query the database directly?
- Are SQL queries properly parameterized (SQL injection risk)?

STEP 5: FILE SYSTEM INSPECTION
Commands:
- file system ls /data/data/[package]/
- file system pwd
- file system cat [filepath]

Analysis:
- What files exist in the app's private data directory?
- Are there configuration files, cached data, or backup files?
- Do SharedPreferences contain sensitive data?
- Are there world-readable files (permission 644 or 777)?
- Are there private keys or certificates stored in plaintext?

STEP 6: CLIPBOARD INSPECTION
Commands:
- android clipboard monitor

Analysis:
- Is sensitive data (tokens, passwords, PII) being copied to clipboard?
- For how long is data retained?
- Can other apps access this data?

STEP 7: SCREENSHOT CAPABILITY
Commands:
- android ui screenshot

Analysis:
- Can we capture the app's screen during sensitive operations?
- Are sensitive fields (passwords, tokens) visible in UI?

STEP 8: SSL PINNING TEST
- Are all connections to backend services encrypted?
- Is certificate pinning implemented?
- Can we MITM the connection by installing a custom CA?

STEP 9: DEEP LINKING TEST
- Can we invoke Activities/Intents directly?
- Are there deeplinks that bypass authentication?
- Can we access protected data via deeplinks?

For each finding:
1. Describe what was discovered
2. Severity level (Critical/High/Medium/Low)
3. Security impact
4. Recommendation for remediation

Finally, provide:
- Summary of runtime security posture
- Top vulnerabilities identified
- Evidence/artifacts from the analysis
```

---

## 6. Custom Frida Hooks Development

### Prompt:
```
Based on the vulnerabilities we've discovered during our Android security assessment, I need to create custom Frida hooks to further exploit or validate the findings.

Here is what we've discovered so far:
[List key vulnerabilities: e.g., "Tokens stored in SharedPreferences", "Unencrypted database", "Authentication bypass", etc.]

For EACH vulnerability, create a Frida hook that:

1. HOOKS RELEVANT FUNCTIONS:
   - Identify the Java/Kotlin functions involved
   - Hook method entry/exit points
   - Intercept arguments and return values

2. DEMONSTRATES THE VULNERABILITY:
   - Shows how data is being stored/transmitted insecurely
   - Extracts sensitive data at runtime
   - Bypasses security controls
   - Logs evidence of the vulnerability

3. INCLUDES PROPER ERROR HANDLING:
   - Try/catch blocks
   - Null checks
   - Type validation

4. PROVIDES CLEAR OUTPUT:
   - Logs findings with timestamps
   - Highlights sensitive data discovered
   - Shows function call stack or context

Example template for Frida hooks:

VULNERABILITY: SharedPreferences storing tokens unencrypted
HOOK TARGETS:
- SharedPreferences.Editor.putString() / putBoolean() / etc.
- SharedPreferences.getString()
- SharedPreferences.getAll()

HOOK CODE:
[Provide the actual Frida JavaScript code]

Repeat for each vulnerability. Include hooks for:
- Authentication token retrieval/storage
- Encryption/decryption operations
- Database writes/reads
- Network requests (headers, body, TLS interception)
- File I/O operations on sensitive files
- Memory allocation of sensitive data
- Clipboard access
- Deeplink invocations
- Permission checks

For each hook:
1. Explain what it does
2. Provide the Frida code
3. Explain how to run it
4. Show example output
5. Explain how to interpret the results
```

---

## 7. Exploring the Attack Surface

### Prompt:
```
Based on our security assessment of this Android application, I want to understand the full attack surface and how a malicious actor could exploit the vulnerabilities we've discovered.

Here are the key vulnerabilities we've found:
[List: debuggable=true, exported activities, path traversal, IDOR, weak cryptography, tokens in memory, etc.]

For EACH vulnerability, explain:

1. ATTACK VECTOR:
   - How would a malicious app or attacker exploit this?
   - What's the realistic attack scenario?
   - What are the prerequisites (app installed, device rooted, MITM position, etc.)?

2. IMPACT ASSESSMENT:
   - What data can be accessed/modified/exfiltrated?
   - What actions can be performed?
   - What's the scope of the breach?
   - How many users/accounts could be compromised?

3. ATTACKER TOOLING:
   - What tools would an attacker use (Frida, Objection, ADB, custom malware, etc.)?
   - What code/scripts would they write?
   - How could they automate the exploit?

4. BUSINESS IMPACT:
   - Financial loss
   - Data breach
   - Compliance violations (GDPR, PCI-DSS, HIPAA, etc.)
   - Reputational damage
   - User trust erosion

5. DEFENSIVE MEASURES:
   - How to detect this attack in progress?
   - Logging/monitoring that should be in place?
   - Incident response procedures?

THEN, create an INTEGRATED ATTACK SCENARIO:
- Assume a malicious actor has access to your application
- Chain multiple vulnerabilities together (e.g., exported activity + path traversal + unencrypted database)
- Show how they would steal authentication tokens, access user data, perform unauthorized transactions, etc.
- Provide a step-by-step attack walkthrough
- Include the tools and commands they'd use
- Estimate the time to full compromise

Finally, provide:
- Risk score (1-10) for each vulnerability
- Overall application security posture risk score
- Recommended prioritization for remediation
- Quick wins vs. structural fixes
```

---

## 8. Compliance & Regulatory Mapping

### Prompt:
```
Based on our security assessment findings, map the vulnerabilities to regulatory compliance requirements:

For each vulnerability found, identify violations of:

MOBILE SECURITY STANDARDS:
- OWASP Mobile Top 10 (M1-M10)
- NIST Mobile Security Guidelines
- CWE (Common Weakness Enumeration)

COMPLIANCE FRAMEWORKS:
- GDPR (if processing EU user data)
- CCPA/CPRA (if processing California resident data)
- PCI-DSS (if processing payment card data)
- HIPAA (if processing health information)
- SOC 2 Type II
- ISO 27001

RELEVANT STANDARDS:
- CVSS v3.1 scoring for each vulnerability
- CWE mapping
- OWASP categorization

For each vulnerability:
1. List applicable regulations
2. Explain the violation
3. Potential fines/penalties
4. Remediation timeline requirements
5. Evidence needed for compliance audit

Provide a compliance risk summary and prioritized remediation roadmap.
```

---

## 9. Create Pen Test Report Finding

### Prompt:
```
I will provide you security vulnerability findings from my Android penetration test. For each finding, create a structured pen test report entry with the following format:

---

## Finding: [Vulnerability Name]

**Severity:** [Critical | High | Medium | Low]

### Description
Brief explanation of what the vulnerability is and how it manifests in the application.

### Evidence
- Specific file path and line number(s)
- Code snippet or configuration demonstrating the issue
- Tool output (Frida hook results, Logcat logs, etc.)
- Screenshot or proof of concept if applicable

### Impact
- What data/systems are at risk
- Who could exploit this (authenticated user, malicious app, MITM attacker, etc.)
- Potential business impact (data breach, unauthorized access, compliance violation, etc.)
- CVSS v3.1 score and rating

### Recommendations
- [First actionable remediation step]
- [Second actionable remediation step]
- [Third actionable remediation step]
- [Timeline for remediation]

### References
- CWE ID (e.g., CWE-200: Exposure of Sensitive Information)
- OWASP Mobile Top 10 category (e.g., M1: Improper Platform Usage)
- Related security standards or compliance requirements

---

Repeat this format for each vulnerability I provide. Keep descriptions concise (2-3 sentences), evidence specific with line numbers, impact focused on business consequences, and recommendations brief but actionable.

Provide all findings in this structured format suitable for inclusion in a professional penetration test report.
```

---

## Summary: Using These Prompts

**For each testing phase:**

1. **Preparation**: Gather artifacts (manifest, source code, APK, logs, memory dumps)
2. **Copy-Paste**: Select the relevant prompt section
3. **Attach/Include**: Provide the artifact (code, config, logs, etc.)
4. **Review & Refine**: Iterate with follow-up questions for clarification
5. **Document**: Save the LLM's response for your report

**Best practices:**
- Run these in sequence (manifest → source → secrets → logcat → runtime → hooks)
- Cross-reference findings across prompts
- Prioritize by severity and exploitability
- Create proof-of-concept exploits for critical findings
- Include evidence/screenshots for each vulnerability
- Document all findings with remediation guidance

