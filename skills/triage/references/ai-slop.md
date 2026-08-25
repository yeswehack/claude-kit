# AI-Slop Patterns in Bug Bounty Reports

> Internal reference for the `triage` skill (and the always-on rules).
> Not an invocable skill. `triage` reads this during its AI-slop pass.

Read the draft report and flag everything that looks like unverified LLM
output. Be strict. Reports get rejected for these patterns all the time
on YesWeHack.

## Critical red flags (any one of these → DO NOT SUBMIT)

### 1. Hallucinated facts
- A **CVE ID** that doesn't exist or doesn't match the described bug.
  → Verify every CVE referenced. Wrong CVEs are the #1 giveaway.
- **Endpoints or parameters** that don't exist on the target.
  → If the hunter says `/api/v2/admin/users?debug=true`, that path must
  resolve. Generic API paths invented by an LLM are a red flag.
- **Library/version** claims without a fingerprint.
  → "The target uses lodash 4.17.20 which is vulnerable to..." — was that
  actually fingerprinted, or did the LLM guess based on the year?
- **Headers or cookies** that don't actually appear in the response.
- **Function or class names** copy-pasted from generic exploitation guides.

### 2. Theoretical-only impact
- "An attacker **could potentially** gain access to..." — without proof.
- "This **may lead to** RCE" — without showing RCE.
- "An attacker **with the right conditions** could..." — what conditions?
  Are they present here?
- Impact escalations that are not demonstrated in the PoC.
  → If the report claims session hijack, the PoC must show a session
  being hijacked. Not "and therefore session hijack is possible."

### 3. PoC that is not actually a PoC
- Pseudocode instead of real requests.
- `curl https://target.com/vuln?payload=<XSS>` with no real payload.
- "Send this request and you will get..." without the actual response shown.
- Steps that skip over the interesting part with "..." or "as shown above".
- Screenshots described in text but not actually attached.

## Strong red flags (any → NEEDS FIXES)

### 4. AI structural tics
- Excessive H2/H3 sectioning for a simple bug (Introduction / Background /
  Vulnerability Details / Technical Details / Impact / Recommendations /
  Conclusion — for a 1-step XSS).
- "**In conclusion**", "**It is important to note**", "**This vulnerability
  highlights**" phrases.
- Bullet lists where prose would be clearer, especially three-item lists
  that paraphrase each other.
- Closing paragraph that summarizes the whole report ("In summary, this
  report has demonstrated...").
- Excessive bold/italic for emphasis on generic terms.

### 5. OWASP boilerplate
- **Multi-paragraph** generic definitions of the vulnerability class
  copy-pasted from OWASP / PortSwigger / MDN → delete. One short
  sentence of context for non-specialist reviewers (program managers,
  devs reading post-triage) is fine; a 5-line OWASP intro on a 1-step
  bug is slop.
- "Cross-Site Scripting (XSS) is a type of injection attack where..."
  as a standalone paragraph → delete.
- "Mitigation: validate all user input, use parameterized queries..." →
  generic advice not tied to the actual code path of this bug. Either
  give a fix specific to the bug, or omit the section.

### 6. Confidence without evidence
- Severity rated Critical/High with no impact demonstration.
- CVSS vector whose metrics contradict the stated preconditions or
  proven impact. Scoring must be **CVSS 3.1, Base metrics only** —
  flag any Environmental/Temporal metrics the hunter set themselves
  (those are the program's to weigh; setting them inflates the score).
  The classic LLM pattern is the "max-severity" vector
  (`AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` = 9.8) defaulted onto a bug
  it doesn't fit. Common mismatches: `PR:N` claimed when auth has a
  real gate (paid tier, admin invite, org membership) — `PR:N` is
  only correct when self-registration is open; `UI:N` claimed when
  the victim must click / paste / load; `S:C` claimed without an
  actual trust boundary crossing; `AC:L` claimed when the resource ID
  is a non-guessable UUID (that's `AC:H`). Check each metric against
  what was actually verified.
- "Easily exploitable" / "trivially exploitable" without showing the
  exploit working in one step.

### 7. Suspicious neutral tone
- The whole report reads like a textbook, not a hunter's lab notes.
- Zero mention of what the hunter tried, what failed, what they had to
  bypass — only a polished narrative.
- No mistakes, no dead ends. Real exploitation rarely looks this clean.

## Minor red flags (any → MINOR ISSUES)

- Unicode arrows, em dashes used in code blocks, smart quotes in payloads
  (these break copy-paste reproduction).
- Mentioning "the AI assistant" or "as an AI language model".
- Markdown that wasn't rendered (literal `**bold**` in submitted text).

## Output

For each red flag found, output:

```
[CRITICAL|STRONG|MINOR] - <category>: <quote or paraphrase from the report>
→ Fix: <concrete instruction to the hunter>
```

If no red flags found, say so explicitly. Do NOT pad with caveats.

