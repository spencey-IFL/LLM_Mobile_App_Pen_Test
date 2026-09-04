# Android Security Testing Instruction Set for LLM Prompts

Structured LLM prompts for baseline mobile app security testing (iOS/Android). Covers manifest analysis, source code review, secrets scanning, logcat inspection, Frida hooks, attack surface analysis, compliance mapping, and penetration test reporting.

---

## 🎯 Purpose

This page provides a curated collection of Language Model (LLM) prompts designed to support baseline security coverage of mobile application penetration tests. These prompts serve as structured starting points for common testing scenarios, helping ensure consistent, comprehensive assessment of mobile apps across multiple platforms (iOS/Android) and architectures.

---

## ⚠️ Scope

This is a baseline reference, **not a complete testing framework**. Each prompt captures a typical pentest scenario and provides foundational questions and investigation steps. However:

- **Findings require deeper exploitation** — Discovering a vulnerability is the first step; validating its impact, exploitability, and business risk requires additional investigation specific to your target application
- **Validation is mandatory** — All findings must be validated in the target environment. Generic prompt outputs may not directly apply to your specific app architecture, configuration, or controls
- **Tailoring is expected** — Use these prompts as templates; adapt them to your target's technology stack, threat model, and business context

---

## 📋 Overview

This guide provides a collection of LLM prompts organised into 9 testing phases for comprehensive Android application security assessments. Each prompt is designed to be copy-pasted or the entire instruction set uploaded into Claude, ChatGPT, or Copilot with relevant artifacts attached.

**Use this for:**
- Baseline security assessments
- Vulnerability identification and prioritization
- Proof-of-concept development
- Compliance and regulatory mapping
- Professional penetration test reporting

---

## 🚀 How to Use This Page

### Step 1: Setup Your Environment
1. Create a workspace in Visual Studio Code (ensuring Copilot is integrated)
2. Save source repositories to workspace
3. Prepare all testing artifacts (see "Prerequisites" section below)

### Step 2: Gather Artifacts
- AndroidManifest.xml
- Source code (or decompiled APK via jadx/apktool)
- TruffleHog scan results (JSON)
- Logcat dump from user activity session
- Memory dumps (heap, runtime artifacts)
- Frida hook results
- Network proxy logs (optional)

### Step 3: Execute Testing Sequence
Follow these phases in order:
- Phase 1: Manifest Analysis
- Phase 2: Source Code Review
- Phase 3: Secret Scanning (TruffleHog)
- Phase 4: Logcat Analysis
- Phase 5: Frida & Objection Runtime Testing
- Phase 6: Custom Frida Hooks
- Phase 7: Attack Surface Exploration
- Phase 8: Compliance Mapping
- Phase 9: Pen Test Report Formatting

### Step 4: Use Each Prompt
1. Select the relevant prompt from `LLM_BASELINE_SECURITY_TESTING_PROMPTS.md`
2. Copy the entire prompt (between the triple backticks)
3. Feed it to an LLM (Claude, ChatGPT, Copilot, etc.)
    - Alternatively, upload the entire instruction sey - `LLM_BASELINE_SECURITY_TESTING_PROMPTS.md`
4. Include target-specific context:
   - App name and platform
   - Known technologies (frameworks, libraries, APIs)
   - Business context (what data it handles)
   - Environment (dev/staging/production)
5. Review LLM output carefully
6. Ask follow-up questions for clarification

### Step 5: Validate & Document Findings
- **Validation is mandatory:** Reproduce each finding in the target environment
- Ensure it actually poses risk to your specific app
- Do not report LLM suggestions that don't validate
- Use Phase 9 template to format each validated vulnerability
- Include Evidence, Impact, and Recommendations
- Document clear reproduction steps
- Cross-reference with other phases
- Prioritize by severity and exploitability

---

## 📁 Testing Phases

### Phase 1: Android Manifest Analysis
**Target:** `AndroidManifest.xml`  
**Looks for:**
- Debuggable apps (debuggable="true")
- Backup enabled (allowBackup="true")
- Exported components without protection
- Overpermissive permissions
- Hardcoded credentials/URLs
- Network security config issues
- SDK version vulnerabilities

**Output:** Critical/High/Medium/Low findings with remediation

---

### Phase 2: Source Code Security Review
**Target:** Decompiled source code (jadx, apktool output, or sourcecode provided by team)  
**Looks for:**
- Authentication & session management flaws
- Cryptographic weaknesses
- Data storage vulnerabilities
- API security issues
- Input validation flaws
- Authorization bypasses
- Logging sensitive data

**Output:** Vulnerability assessment with code citations

---

### Phase 3: TruffleHog Secret Scanning
**Target:** `trufflehog-results.json`  
**Command:**
```bash
trufflehog filesystem /path/to/source --json > trufflehog-results.json
```
Also display results to the terminal

**Looks for:**
- API keys (verified and unverified)
- Private keys
- Passwords and credentials
- Auth tokens
- AWS keys
- Base64-encoded secrets

**Output:** Severity-prioritized list with rotation guidance

---

### Phase 4: Logcat Analysis
**Target:** Logcat dump captured during user activity  
**Setup:**
```bash
adb logcat > logcat-output.txt
# User logs in and exercises all features
# Stop logcat (Ctrl+C)
```

**Looks for:**
- Credentials and auth tokens
- PII (names, emails, phone numbers, SSN)
- Financial data (card numbers, account numbers)
- Device identifiers
- URLs with sensitive data
- Stack traces / debug output

**Output:** Data leakage report with line references

---

### Phase 5: Frida & Objection Runtime Testing
**Target:** Running app with Frida server  
**Setup:**
```bash
# Device must have Frida server running
objection -g com.package.name start
```

**Tests:**
- Environment variable inspection
- Memory dump analysis
- Keystore listing
- Database access (SQLite)
- File system inspection
- Clipboard monitoring
- Screenshot capability
- Keyboard caching
- SSL pinning validation

**Output:** Runtime security posture assessment

---

### Phase 6: Custom Frida Hooks
**Target:** Specific vulnerable functions identified in phases 1-5  
**Creates:** Frida JavaScript hooks to:
- Intercept function calls
- Extract sensitive data at runtime
- Demonstrate vulnerability exploitation
- Validate security assumptions

**Output:** Working Frida hook code with usage examples

---

### Phase 7: Attack Surface Exploration
**Target:** Integrated analysis of all findings  
**Develops:**
- Attack vectors for each vulnerability
- Chained/multi-step exploits
- Malicious actor scenarios
- Tool usage (Frida, ADB, custom malware)
- Business impact assessment
- Detective/preventive controls

**Output:** Comprehensive threat model and attack scenarios

---

### Phase 8: Compliance & Regulatory Mapping
**Target:** Vulnerability-to-regulation correlation  
**Maps to:**
- OWASP Mobile Top 10
- CWE (Common Weakness Enumeration)
- GDPR / CCPA / PCI-DSS / HIPAA / SOC2 / ISO27001
- CVSS v3.1 scoring
- Regulatory penalties and timelines

**Output:** Compliance risk assessment and remediation roadmap

---

### Phase 9: Penetration Test Report Formatting
**Target:** Structured findings for executive reporting  
**Format for each vulnerability:**
- **Description:** Brief explanation
- **Evidence:** File paths, line numbers, code snippets, proof-of-concept
- **Impact:** Affected systems, business consequences, CVSS score
- **Recommendations:** Bulleted remediation steps with timeline
- **References:** CWE, OWASP, compliance standards

**Output:** Professional pen test report entries ready for final documentation

---

## 🛠 Prerequisites

### Tools Required
- **Android SDK tools:** ADB, apktool, jadx (for decompilation)
- **Python tools:** TruffleHog (`pip install trufflehog`)
- **Runtime hooks:** Frida and Frida server (`pip install frida frida-tools`)
- **LLM access:** ChatGPT, Claude, or GitHub Copilot

### Artifacts to Gather
1. APK file or source code repository
2. AndroidManifest.xml (from decompiled APK)
3. All source code files (Java/Kotlin)
4. Network proxies logs (if available)
5. Device with debuggable app installed
6. Frida server on test device
7. User activity captured in logcat

---

## 📊 Usage Workflow

```
Step 1: Setup & Preparation
├── Extract/decompile APK (apktool, jadx)
├── Run TruffleHog scan
├── Capture logcat during usage
└── Start Frida server on device

Step 2: Analysis Phases (Sequential)
├── Phase 1: Manifest Analysis
├── Phase 2: Source Code Review
├── Phase 3: TruffleHog Results
├── Phase 4: Logcat Inspection
└── Phase 5: Runtime Testing

Step 3: Exploitation & Validation
├── Phase 6: Create Frida Hooks
├── Phase 7: Attack Scenarios
└── Develop PoCs for critical findings

Step 4: Reporting
├── Phase 8: Compliance Mapping
├── Phase 9: Format Report Findings
└── Create executive summary & remediation roadmap
```

---

## 💡 Best Practices

### Testing Sequence
- Follow the phases in order (1 → 9)
- Earlier phases inform later phases
- Cross-reference findings across prompts
- Prioritize by severity and exploitability

### Validation (Critical)
- **Do not report unvalidated findings** — LLM suggestions must be independently verified
- **Reproduce in target environment** — Generic outputs may not apply to your app
- **Test with actual user flows** — Validate that vulnerabilities exist and are exploitable
- **Document reproduction steps** — Don't rely on LLM explanation; create your own proof-of-concept
- **Assess actual impact** — Confirm business/security risk in context of target app
- **Check mitigating controls** — What existing security measures might reduce severity?

### LLM Interaction
1. Copy entire prompt section (between ``` markers)
2. Paste into your LLM chat
3. Provide target-specific context (app name, platform, technologies)
4. Provide artifact (code, config, logs) with sensitive data redacted
5. Ask follow-up questions for clarification and deeper analysis
6. Request additional analysis or specific examples
7. Validate findings in your test environment before accepting results
8. Save/document the LLM output AND your validation results

### Artifact Collection
- Use relative paths in logs for redaction
- Don't share token values, only indicators
- Include context (app version, OS version, test date)
- Capture full stack traces
- Document test environment setup
- Redact sensitive data before feeding to LLM

### Remediation Guidance
- Prioritize critical/high severity fixes
- Include implementation difficulty estimates
- Provide secure code examples
- Recommend security libraries/frameworks
- Suggest code review and testing strategies
- Validate that fixes actually remediate the vulnerability

---

## 📋 Deliverables Checklist

- [ ] Manifest analysis report
- [ ] Source code vulnerability assessment
- [ ] TruffleHog scan results with remediation timeline
- [ ] Logcat analysis with data leakage findings
- [ ] Runtime security assessment from Frida/Objection
- [ ] Custom Frida hooks (working code)
- [ ] Attack surface threat model
- [ ] Compliance mapping (OWASP, CWE, regulatory)
- [ ] Penetration test report (Phase 9 format)
- [ ] Executive summary & risk ratings
- [ ] Prioritized remediation roadmap

---

## 🔗 References

### Security Standards
- [OWASP Mobile Top 10](https://owasp.org/www-project-mobile-top-10/)
- [NIST Mobile Security Guidelines](https://csrc.nist.gov/publications/detail/sp/800-163/rev-1/final)
- [CWE Top 25](https://cwe.mitre.org/top25/)

### Tools
- [Frida](https://frida.re/) - Dynamic instrumentation
- [Objection](https://github.com/sensepost/objection) - Mobile security testing
- [APKTool](https://ibotpeaches.github.io/Apktool/) - APK decompilation
- [JADX](https://github.com/skylot/jadx) - Dex decompiler
- [TruffleHog](https://github.com/trufflesecurity/trufflehog) - Secret scanning

### Compliance
- GDPR, CCPA, HIPAA, PCI-DSS, SOC 2 Type II, ISO 27001

---

## 📝 Example Workflow

```bash
# 1. Extract APK
apktool d app.apk -o app_decompiled/

# 2. Run TruffleHog
trufflehog filesystem app_decompiled/ --json > trufflehog-results.json

# 3. Capture Logcat
adb logcat > logcat-output.txt
# [User logs in and uses app]
# [Stop with Ctrl+C]

# 4. Start Frida server
adb push frida-server-14.2.18-android-arm64 /data/local/tmp/
adb shell chmod +x /data/local/tmp/frida-server-14.2.18-android-arm64
adb shell /data/local/tmp/frida-server-14.2.18-android-arm64

# 5. Start objection
objection -g com.example.app start

# 6. Use LLM prompts for each phase
# Copy prompt from LLM_BASELINE_SECURITY_TESTING_PROMPTS.md
# Paste into ChatGPT/Claude
# Attach relevant artifact
# Iterate and document findings
```

---

## ❓ FAQ

**Q: Which LLM should I use?**  
A: Claude, ChatGPT-5, or Copilot all work well. Claude tends to provide more detailed security analysis.

**Q: Can I skip phases?**  
A: Not recommended. Earlier phases provide context for later phases and help identify which specific functions/areas to focus on.

**Q: How long does a full assessment take?**  
A: 2-5 days depending on app complexity, from initial artifact collection through final report.

**Q: What if I find a critical vulnerability?**  
A: Document it immediately, develop a PoC, estimate business impact, and prioritise for remediation. Notify stakeholders.

**Q: Can I use these prompts for iOS?**  
A: Most prompts are adaptable. Swap Android-specific tools (apktool, Frida Android hooks) with iOS equivalents (Frida iOS, Cycript, LLDB).

---

## 📞 Support

For issues or questions:
1. Review the relevant phase prompt
2. Check the references and tool documentation
3. Iterate with your LLM for clarification
4. Document your findings with evidence

---

**Last Updated:** 2026-09-04  
**Version:** 1.0
