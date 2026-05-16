# Vibe Coding — Top-Level Principles

> Author: seaman.player | Version: v1.0 | 2025-11  
> Vibe Coding: Describe intent in natural language, let AI generate code, while engineers focus on architecture, validation, and quality.

---

## Principle 1: Intent Over Instructions

**Don't tell AI how to write it. Tell AI what you need.**

Wrong:
> "Use a for loop to iterate the array, then compare each element with strcmp, and return the index when a match is found."

Right:
> "Find the first matching entry in a string array and return its index; return -1 if not found. Must be C11-compatible with no dependencies beyond the standard library."

An engineer's core value lies in **defining the problem boundary**, not describing implementation steps.

---

## Principle 2: Context Determines Code Quality

**The quality of AI-generated code equals the quality of context you provide.**

Every prompt must specify:
- **Hardware constraints**: chip model, memory size, presence of MMU/FPU
- **Software environment**: OS version, compiler version, SDK version
- **Quality constraints**: real-time requirements, power limits, coding standards
- **Forbidden boundaries**: prohibited libraries, patterns, or code-size limits

The more precise the context, the less rework.

---

## Principle 3: Single-Responsibility Prompts

**One prompt. One problem.**

Break large tasks into independent focused prompts:

```
❌ "Implement camera driver, H264 encoding, RTC signaling, and weak-network reconnect"

✓  Prompt 1: Implement a V4L2 camera frame capture wrapper
✓  Prompt 2: Feed captured YUV frames into the MPP H264 encoder
✓  Prompt 3: Implement a WebRTC signaling handshake state machine
✓  Prompt 4: Implement disconnect detection with exponential backoff reconnect
```

Decomposition is not a loss of efficiency — it is a guarantee of quality.

---

## Principle 4: Validation Is Part of Generation

**Every time you request code, also request a verification method.**

Append to every prompt:
> "Also provide: ① Verification steps or unit test stubs for the key logic ② The place you think is most error-prone and why."

The risks AI identifies itself are usually exactly where human review should focus.

---

## Principle 5: AI Generates, Engineers Decide

**Generation ≠ Adoption. The engineer is the final accountable party.**

| AI is good at | Engineer owns |
|---------------|---------------|
| Boilerplate, repetitive structures, format conversion | Architecture decisions, algorithm selection |
| Docs, comments, initial test case drafts | Security boundaries, performance bottleneck judgment |
| Cross-platform adaptation layer scaffolding | Hardware timing, interrupt safety |
| Error handling patterns | System-level stability validation |

Never let AI-generated code bypass Code Review and go directly to production.

---

## Principle 6: Dialogue Is Iteration, Not a One-Shot Request

**The first generation is just the starting point. Iterative dialogue converges on the best solution.**

Typical iteration flow:

```
Round 1: Generate initial code framework
Round 2: "What problems would the above solution have in a weak-network environment? How to improve?"
Round 3: "Extract the error handling section into a separate function and add logging"
Round 4: "Generate unit tests that cover boundary values for this code"
Round 5: "Which parts of this code violate MISRA-C? List and fix them."
```

The accumulated context from multi-round dialogue is worth more than a single perfect prompt.

---

## Principle 7: Build a Team Prompt Asset Library

**Turn individual experience into team-reusable assets.**

- Every effective prompt template goes into the team **prompt library** (organized by scenario)
- Every typical AI-generated bug discovered goes into the **anti-pattern library**
- Onboarding must-reads: prompt standards + anti-pattern library + high-risk scenario checklist
- The prompt library is versioned and continuously updated with each project iteration

A team's Vibe Coding maturity = the depth of its accumulated prompt assets.

---

## Principle 8: Safety Red Lines Are Non-Negotiable

**In the following scenarios, no matter how perfect the AI output looks, code must be hand-written or deeply rewritten.**

| Red-Line Scenario | Reason |
|-------------------|--------|
| Bootloader / flashing scripts | Failures are irreversible and affect all devices |
| OTA upgrade core logic | Risk of large-scale device bricking in production |
| Crypto key management / eFuse operations | Security vulnerabilities are unrecoverable |
| Real-time closed-loop control (PID / motor) | Hardware damage / personal injury risk |
| Core paths of interrupt handlers | Race conditions are difficult to cover in testing |

Red lines are engineering non-negotiables. They do not flex under delivery pressure.

---

## Principle 9: Vibe Coding Is Not Lazy Programming

**Vibe Coding doesn't release responsibility — it releases repetitive labor.**

- Release: boilerplate code, documentation, format conversion, initial test case drafts
- Retain: architecture judgment, performance tuning, security review, system stability verification

Use the time saved to think more deeply about architecture — not to pile on more features.

---

## Principle 10: Keep Tracking, Stay at the Frontier

**AI tools go through generational leaps every 3–6 months. Stopping to learn means falling behind.**

- Read at least 1 article on AI programming advances each month (model capability updates, tool releases)
- Re-evaluate your toolstack every quarter for better alternatives
- Update the team's prompt standards and tool configuration every six months

Tools change. Principles don't. Principles guide you on how to get the most out of every new generation of tools.

---

## Summary: Ten Principles at a Glance

| # | Principle | One-Liner |
|---|-----------|-----------|
| 1 | Intent over instructions | Say what, not how |
| 2 | Context determines quality | Tighter constraints, better code |
| 3 | Single-responsibility prompts | One problem per prompt |
| 4 | Validation with generation | Ask for code and a way to verify it |
| 5 | AI generates, engineers decide | Generation ≠ adoption |
| 6 | Dialogue is iteration | Multi-round beats one perfect shot |
| 7 | Build prompt assets | Turn personal insight into team wealth |
| 8 | Safety red lines hold | Five scenarios require hand-written code |
| 9 | Not lazy programming | Release repetition, retain judgment |
| 10 | Track the frontier | Stopping learning means falling behind |
