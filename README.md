# GAMKERS API Hacking Lab v2 — Complete Solutions Guide

> ⚠️ **INSTRUCTOR USE ONLY** — Don't share this with students before they attempt the lab!

---

## Lab Overview

| Vuln | Type | OWASP API | Severity |
|------|------|-----------|----------|
| BOLA/IDOR | Broken Object Level Auth | API1:2023 | Critical |
| Mass Assignment | Unprotected field writes | API3:2023 | High |
| BFLA | Broken Function Level Auth | API5:2023 | High |
| Rate Limit Bypass | Lack of resource limiting | API4:2023 | Medium |

**Users in the DB:**
| UID | Name | Role | Hex ID | UUID |
|-----|------|------|--------|------|
| 1 | Arun Kumar | admin | 0x01 | a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c |
| 2 | Divya Nair | user | 0x02 | e7b4a9f1-3c8d-42e7-9d1a-6f0b2c5e8a3d |
| 3 | Karthik Raja | user | 0x03 | c1d9e4b7-8a2f-4b63-a5c0-7d3e9f1b4a8c |
| 4 | Priya Sharma | manager | 0x04 | f0a3b7c9-2d5e-4f81-b6a4-8c1d3e7f9b2a |
| 5 | Ravi Subbu | user | 0x05 | b8c2d6e0-4f1a-4d93-a7b5-9e3f1c8d2a6b |
| 6 | Sneha Mehta | user | 0x06 | d4e8f2a6-1b3c-4a75-c9d7-0f5b8e2a4c1d |

---

# ⭐ EASY DIFFICULTY

**ID Format:** Plain numbers (`users/1`)  
**Defenses:** All OFF  
**Hints:** Shown  
**Scenarios:** Shown (click to auto-fill)

---

## Easy — BOLA / IDOR

**Goal:** Access another user's private data by changing the ID in the URL.

### Steps:
1. Login as **Divya Nair** (uid 2) from the left panel
2. Select **BOLA / IDOR** from the attack list
3. The URL auto-fills as `users/2` — your own profile, returns 200 OK ✓
4. Manually change URL to `users/1` and click **Send**
5. You receive Arun Kumar's full profile including **credit card, salary, balance**
6. Try `users/3`, `users/4`, `users/5`, `users/6` — all return sensitive data

**Result:** `200 OK` with another user's full data — **IDOR confirmed**

**Fix (how to defend):** Enable **Object Ownership Check** toggle in the Defender panel. Backend should verify `request.user.id === resource.owner_id`.

---

## Easy — Mass Assignment

**Goal:** Write protected fields (`role`, `balance`, `salary`) directly in the request body.

### Steps:
1. Login as **Divya Nair** (uid 2)
2. Select **Mass Assignment** from the attack list
3. Switch to the **Body** tab
4. Replace the default body with:
```json
{
  "name": "Divya Nair",
  "role": "admin",
  "balance": 9999999,
  "salary": 500000
}
```
5. Click **Send** — you get `200 OK` and all fields are written including `role: admin`

**Result:** A regular user promoted themselves to admin — **Privilege Escalation**

**Fix:** Enable **Field Allowlist** toggle. Only allow `name` and `email` to be updated.

---

## Easy — BFLA (Broken Function Level Auth)

**Goal:** Access the admin-only endpoint as a regular user.

### Steps:
1. Login as **Divya Nair** (uid 2) — role: `user`
2. Select **BFLA** from the attack list
3. URL auto-fills as `admin/users`
4. Click **Send**
5. You receive **all 6 users' names, roles, and salaries** — admin-only data

**Result:** `200 OK` with full user directory — **BFLA confirmed**

**Fix:** Enable **Role-Based Access** toggle. Server must check `if role !== 'admin' → 403`.

---

## Easy — Rate Limit Bypass

**Goal:** Flood the server to trigger a crash (DoS).

### Steps:
1. Login as any user
2. Select **Rate Limit Bypass**
3. Ensure **Rate Limit defense is OFF** in the Defender panel
4. Click **Flood x20** scenario chip (or spam **Send** rapidly 6+ times)
5. The responseArea shows **💥 SERVER CRASHED** — DoS achieved

**Bonus — with Rate Limit ON:**
1. Enable **Request Rate Limit** toggle
2. Send 6+ requests rapidly
3. After 5 requests you get `429 Too Many Requests` — server protected

**Fix:** Enable **Request Rate Limit** + **Auto Lockout** toggles.

---

# ⭐⭐ MEDIUM DIFFICULTY

**ID Format:** Hex-encoded (`users/0x01`)  
**Defenses:** All OFF (tighter rate window: 4 req/8s)  
**Hints:** Shown  
**Scenarios:** Hidden — you must craft requests manually

---

## Medium — BOLA / IDOR

**Challenge:** IDs are now hex-encoded. You need to discover the encoding format.

### Steps:
1. Login as **Divya Nair** — notice the sidebar shows `uid: 0x02`
2. Select **BOLA / IDOR** — URL auto-fills as `users/0x02`
3. Send the request to understand your own profile returns normally
4. **Manually change** the URL to `users/0x01` and Send
5. You receive Arun Kumar's data (admin)
6. Enumerate: `0x01`, `0x02`, `0x03`, `0x04`, `0x05`, `0x06`

**Key insight:** Hex `0x01` = decimal `1`, `0x02` = `2`, etc. Standard hex encoding.

**Result:** Same IDOR but you had to figure out the encoding yourself.

---

## Medium — Mass Assignment

**Challenge:** The alert no longer tells you which fields are "dangerous" — you must know.

### Steps:
1. Login as any user, select **Mass Assignment**
2. The body defaults to `{"name":"...","email":"..."}`
3. Try sending it — safe update, `200 OK`
4. Now add protected fields manually:
```json
{
  "name": "Divya Nair",
  "email": "divya@techcorp.in",
  "role": "admin"
}
```
5. Send — `200 OK`, role changed to admin despite no hint telling you `role` was protected

**Key insight:** Common protected fields in any app: `role`, `is_admin`, `balance`, `salary`, `permissions`.

---

## Medium — BFLA

### Steps:
Same as Easy — URL is `admin/users`, no hex encoding on endpoint paths.

1. Login as any non-admin user
2. Select **BFLA**, send request to `admin/users`
3. Get full user directory — **BFLA confirmed**

---

## Medium — Rate Limit

**Rate window tightened:** 4 requests / 8 seconds  

### Steps:
1. Login as any user, select **Rate Limit Bypass**
2. Rapidly send 5+ requests manually (no scenario chips to help you)
3. On the 5th request you get `429 Too Many Requests`
4. Without rate limit ON: sending 8+ rapid requests crashes the server

---

# ⭐⭐⭐ PRO DIFFICULTY

**ID Format:** UUID (`users/a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c`)  
**Defenses:** **Rate Limit is ON** and LOCKED (must bypass it)  
**Hints:** Hidden  
**Scenarios:** Hidden  
**Users visible:** Only your own logged-in user (must discover others)

---

## Pro — BOLA / IDOR

**Challenge:** UUIDs, only own profile visible, must discover other users' UUIDs.

### Steps to discover UUIDs:
1. Login as **Divya Nair** — URL shows `users/e7b4a9f1-3c8d-42e7-9d1a-6f0b2c5e8a3d`
2. Send the request — response includes `"id": 2` revealing the numeric ID
3. **The UUID prefix pattern:** Notice each UUID follows a consistent internal structure. In a real app you'd find UUIDs via:
   - API responses that reference other users (`created_by`, `assigned_to`)
   - Error messages leaking IDs
   - Predictable patterns in UUID v1 (time-based)
4. In this lab, try the UUIDs from the table above — the app accepts them

**Target UUID for admin (uid 1):** `a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c`

### Steps:
1. Login as **Divya Nair**
2. Change URL from your UUID to `users/a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c`
3. Click **Send** — you get Arun Kumar's (admin) full data including card/salary

**Result:** UUID didn't prevent IDOR — the backend still doesn't verify ownership.

---

## Pro — Mass Assignment

### Steps:
1. Login, select **Mass Assignment**
2. URL: `users/e7b4a9f1-3c8d-42e7-9d1a-6f0b2c5e8a3d` (your UUID)
3. Body:
```json
{
  "role": "admin",
  "balance": 9999999,
  "salary": 999999
}
```
4. Send — no hint shown but `200 OK` with escalated privileges

---

## Pro — BFLA

### Steps:
1. Login as any user (even non-admin)
2. Select **BFLA**, URL: `admin/users`
3. Send — **200 OK**, full directory leaked (UUID for BFLA doesn't change endpoint)

---

## Pro — Rate Limit BYPASS

**Challenge:** Rate limiting is **locked ON**. You can't toggle it off. Goal: Understand the real-world implication, NOT bypass the UI toggle (that's locked). The exercise is to observe the defense working.

### What Pro teaches:
1. Login, select **Rate Limit Bypass**
2. Rapidly send requests — after 3 requests (Pro limit) → `429`
3. Rate meter shows `3/3 req`
4. With **Auto Lockout also enabled**: after hitting the limit, your token is locked out entirely

**Real-world bypass techniques** (documented for understanding):
- **IP rotation** — attacker uses different IPs each request
- **Token rotation** — attacker uses multiple API tokens
- **Slow-rate attack** — stay just under the threshold (e.g. 2 req/6s)
- **Distributed attack** — use multiple machines

---

# 🔥 EXPERT DIFFICULTY

**ID Format:** UUID  
**Defenses:** **Rate Limit + Ownership Check** are ON and LOCKED  
**Hints:** Hidden  
**Scenarios:** Hidden  
**Users visible:** Only own profile  
**Rate limit:** 2 req / 5s — extremely tight

---

## Expert — BOLA / IDOR

**Challenge:** UUID IDs + Ownership check is LOCKED ON = the app is seemingly patched. Your goal: demonstrate that even with ownership check, the pattern exists conceptually and test the BFLA attack vector as an alternative.

### The IDOR is blocked — here's what happens:
1. Login as **Divya Nair**
2. Try `users/a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c` (Arun Kumar's UUID)
3. **403 Forbidden** — Ownership check is active

### The pivot — use BFLA instead:
4. Switch to **BFLA**, send to `admin/users`
5. **200 OK** — the admin endpoint has NO ownership check!
6. You see ALL users and their data via the admin endpoint — data exfiltration achieved via a different attack surface

**Lesson:** Fixing IDOR doesn't mean the app is secure — BFLA can expose the same data.

---

## Expert — Mass Assignment & Parameter Pollution (The Real World Exploit)

**Challenge:** Ownership check strictly blocks you from doing a Mass Assignment against another user's UUID. If you try to update your own role to 'admin', the Mass Assignment allowlist blocks it. So how do you attack?

### The Advanced Attack Vector: Parameter Pollution / HTTP Body Override
A very common real-world vulnerability exists when the **authorization checks use the URL parameter**, but the **ORM/database logic uses the JSON body parameter**.

### Steps:
1. Login as **Divya Nair**.
2. Select **Mass Assignment**. The URL will target your own UUID: `users/e7b4...`.
3. If you send a request now, you pass the ownership check because evaluating `url_uid == token_uid` returns `true`.
4. But we are going to inject an overriding `id` parameter into the JSON body to target the Admin (Arun Kumar, UUID: `a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c`):
```json
{
  "id": "a3f8c1d2-7b4e-4a91-bf0c-9e3d5a8f1b2c",
  "balance": 0,
  "salary": 100
}
```
5. Click **Send**! (Be mindful of the 2 req/5s rate limit).
6. **200 OK!** The backend verified you owned the URL profile, but then blindly mapped your JSON body to the database update mechanism, thereby overwriting the Admin's account data.

**Reward:** The UI will pop a special 🔥 **Advanced Bypass** banner celebrating your Parameter Pollution execution.

**Lesson:** An API must ensure that parsed variables from the path (`req.params.id`) and payload (`req.body.id`) do not contradict each other before performing state-changing operations.

## Expert — BFLA

**This is the main exploit vector in Expert mode.**

### Steps:
1. Login as **any non-admin** user
2. Select **BFLA**
3. URL: `admin/users`
4. Be careful of the 2 req/5s rate limit — wait between attempts
5. **200 OK** — full user directory including all UUIDs, roles, salaries

**Critical insight for Expert:** The student should notice:
- IDOR is blocked (ownership check locked)
- Mass Assignment is harder (must be slow, ownership checked)
- **But BFLA is still completely open** — the admin endpoint has no role guard
- This teaches that fixing one class of vuln doesn't secure the app

---

## Expert — Rate Limit

**Challenge:** Only 2 requests per 5 seconds. Auto Lockout is recommended.

### Steps:
1. Try to send 3+ requests quickly
2. `429 Too Many Requests` on the 3rd
3. With lockout enabled: token is locked until page refresh
4. This simulates a hardened production rate limiter

**Real-world bypass techniques (educational):**
- Use multiple `Authorization` tokens (different accounts)
- Rotate IPs via proxy chains
- Implement exponential backoff to stay under threshold
- Use distributed bots

---

# Summary Cheat Sheet

| Attack | Easy | Medium | Pro | Expert |
|--------|------|--------|-----|--------|
| IDOR | `users/2` | `users/0x02` | UUID required | UUID + blocked |
| Mass Assign | Any field in body | Know fields yourself | UUID path | **Parameter Pollution** (Body ID Override) |
| BFLA | `admin/users` | `admin/users` | `admin/users` | `admin/users` ← main vector |
| Rate Flood | Spam Send | Manual spam only | Defense ON (observe) | 2 req/5s — very strict |
| Crash Server | 6 rapid hits | 8 rapid hits | N/A (rate limit locked) | N/A |

---

## OWASP References

- [API1:2023 - Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [API3:2023 - Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- [API4:2023 - Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)
- [API5:2023 - Broken Function Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/)
