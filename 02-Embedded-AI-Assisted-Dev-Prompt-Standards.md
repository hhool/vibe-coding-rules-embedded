# Embedded AI-Assisted Development Prompt Standards

> Version: v1.0 | Maintainer: Hou Yanmeng | Updated: 2025-11  
> Applicable Tools: Cursor IDE / GitHub Copilot / Claude API / ChatGPT / DeepSeek  
> Applicable Domains: Embedded Linux, RTOS, Audio/Video SDK, Driver Development, Cross-platform Porting

---

## 1. General Principles

### 1.1 Objectives

- Standardize AI prompt writing across the team to reduce low-quality generation and rework.
- Define the trust boundary of AI: which generated code can be adopted directly, and which must be manually verified.
- Turn Vibe Coding (intent-driven programming) into an executable engineering workflow.

### 1.2 Core Principles

| Principle | Description |
|-----------|-------------|
| **Context First** | Prompts must include chip model, OS, toolchain, and SDK version as key constraints |
| **Clear Intent** | Describe *what* you want, not *how* to do it — let AI make selections, engineers validate |
| **Minimal & Complete** | One prompt solves one clearly scoped sub-problem; avoid mixing multiple requirements |
| **Verifiable Output** | Always request test cases or a verification method alongside generated code |
| **Safety Red Lines** | Code involving interrupts, memory mapping, or power management must be manually reviewed line-by-line |

---

## 2. Base Prompt Template

### 2.1 Universal Five-Part Structure

```
[Role]        You are an embedded C/C++ engineer experienced with {platform}.
[Context]     Project: {name}, Chip: {model}, OS: {Linux/RTOS}, Toolchain: {GCC version}, SDK: {name & version}.
[Constraints] {Forbidden libraries/functions}; {memory limits}; {real-time requirements}; {coding standard, e.g. MISRA-C / Google Style}.
[Task]        {Specific task, verb-first, single objective}.
[Output]      {Code + comments on key logic + basic test case or verification method}.
```

**Example:**

```
[Role]        You are an embedded Linux C engineer experienced with RK3588 + Debian 12.
[Context]     Project: Outdoor LED display controller; Chip: RK3588; OS: Debian 12; Toolchain: GCC 11.3; SDK: rkmedia 1.7.
[Constraints] No Qt; follow Linux Kernel Coding Style; functions must not exceed 80 lines.
[Task]        Implement a function that reads a smoke sensor register via Modbus RTU, with timeout retry (max 3 attempts).
[Output]      C code + inline comments on each key step + a simple unit test stub.
```

---

## 3. Scene-Specific Prompt Standards

### 3.1 Driver Development

**Use for:** UART/I2C/SPI/GPIO drivers, device tree adaptation, character device drivers

**Required context fields:**
- Kernel version (`uname -r` output)
- Chip vendor BSP version
- Hardware interface spec (pins, frequency, electrical characteristics)
- Whether DMA / interrupt mode is needed

**Template:**

```
Chip: {model}, Kernel: {version}, BSP: {version}.
Peripheral: {name}, Interface: {I2C/SPI/UART}, Address/Rate: {value}.
Requirement: Implement {function}, using {interrupt/polling/DMA} mode, compatible with device tree config.
Output: Driver .c/.h skeleton + sample dts node + annotated probe/remove functions.
```

**⚠️ Safety requirements — manual review mandatory for:**
- `ioremap` / `request_irq` / `dma_alloc_coherent`
- Shared resource access inside interrupt handlers
- Power management callbacks (suspend/resume)

---

### 3.2 Audio/Video SDK Integration

**Use for:** ffmpeg wrappers, rkmedia/MPP calls, WebRTC adaptation, GStreamer pipelines

**Required context fields:**
- ffmpeg / MPP / WebRTC version
- Codec format, resolution, and frame rate requirements
- Whether hardware acceleration is used (VPU/GPU)
- Target latency budget

**Template:**

```
Platform: {RK3588/Hisi/T40}, Library: {ffmpeg 6.x / rkmedia / MPP}.
Requirement: Implement {H264/H265} {encode/decode}, resolution {1080P/4K}, {30/60} fps, {hardware/software}.
Latency target: end-to-end < {X} ms.
Output: Core API call code + parameter config notes + common error handling.
```

---

### 3.3 WebRTC / RTC SDK Trimming & Porting

**Use for:** Embedded WebRTC porting, signaling integration, weak-network optimization

**Required context fields:**
- Target chip compute budget (MIPS/DMIPS or measured idle CPU usage)
- Required audio/video codec formats
- Network conditions (weak network / LAN / public P2P)
- Whether NACK / FEC / BWE are needed

**Template:**

```
Target platform: {chip}, Compute budget: {cores, clock speed}, OS: {Linux/RTOS}.
WebRTC version: M{version}. Modules to strip: {e.g. VP8/VP9/DataChannel}.
Audio: {sample rate, channels, AEC/ANS/AGC on/off}; Video: {codec, resolution}.
Output: Trimmed GN config + key API init code + recommended weak-network parameter settings.
```

---

### 3.4 Cross-Platform Porting

**Use for:** Migrating source code between platforms (e.g., Hisi → RK)

**Template:**

```
Source platform: {Hisi3518ev200}, Target platform: {RK3588}, both on {Linux}.
Module to port: {module name, e.g. H264 encoding wrapper layer}.
Known API differences: {e.g. MPI vs MPP}.
Requirement: Keep the external interface unchanged (header provided below); replace only the underlying implementation.
Output: Adaptation layer code + diff annotations + checklist of items requiring manual verification.
```

---

### 3.5 CMake / Makefile Build Scripts

**Template:**

```
Project structure: {brief directory tree}, Target artifact: {.so / .elf / .a}.
Toolchain: {cross-compiler prefix, e.g. aarch64-linux-gnu-}, sysroot: {path}.
Dependencies: {library names and versions}.
Requirement: Support Debug/Release switch; output compile_commands.json for Clangd.
Output: Complete CMakeLists.txt + annotated key variables.
```

---

### 3.6 Unit Test / Stub Generation

**Template:**

```
Function under test: {signature + brief description} (code below).
Test framework: {Unity / Google Test / custom stub}.
Required coverage: normal flow / boundary values ({list boundary conditions}) / error returns.
Output: Test case code + estimated coverage explanation.
```

---

## 4. Prompt Quality Checklist

Before sending a prompt to AI, self-check the following:

- [ ] Is the chip model and OS version specified?
- [ ] Is the key library / SDK version specified?
- [ ] Are forbidden functions or libraries listed?
- [ ] Does the task have exactly one clear objective (no AND-joined multiple tasks)?
- [ ] Is the output format specified (code + comments + verification method)?
- [ ] Are real-time / memory / power constraints stated?

---

## 5. AI-Generated Code Review Standards

### 5.1 High-Risk Items Requiring Manual Line-by-Line Review

| Category | Typical Scenario | Risk |
|----------|-----------------|------|
| Memory operations | `malloc`/`free`, DMA buffer, ring buffer | Memory leak, UAF |
| Interrupt context | Shared variable access in ISR | Race condition |
| Power management | suspend/resume callbacks | Deadlock, data loss |
| Hardware timing | I2C/SPI delays, power-on sequence | Hardware damage |
| Crypto / Auth | TLS config, device certificates | Security vulnerability |
| Concurrency control | mutex/semaphore/spinlock usage | Deadlock, priority inversion |

### 5.2 Review Workflow

```
AI-Generated Code
       │
       ├─ 1. Static analysis (cppcheck / clang-tidy)         — Automated
       ├─ 2. Manual line-by-line review of high-risk items    — Mandatory
       ├─ 3. Unit test execution (Unity / GTest)              — Automated
       ├─ 4. On-target integration test (functional verify)   — Mandatory
       └─ 5. Long-duration stability test (≥ 24 h)           — Based on module criticality
```

### 5.3 Review Pass Criteria

- No `cppcheck` error-level warnings
- All high-risk items manually confirmed
- Core path unit test coverage ≥ 70%
- Basic functionality runs without crashes on target hardware

---

## 6. Scenarios Where AI-Generated Code Must NOT Be Used Directly

In the following scenarios, AI may **only assist with design ideas — code must be hand-written or deeply rewritten by an engineer:**

1. **Bootloader / U-Boot modifications** — Risk: bricking the device
2. **Mass production flashing scripts** — Risk: bulk device damage
3. **OTA upgrade core logic** — Risk: large-scale bricking
4. **Crypto key management / eFuse operations** — Risk: irreversible security vulnerabilities
5. **Real-time closed-loop control (PID / motor control)** — Risk: hardware damage / personal injury

---

## 7. Recommended Tool Configuration

### 7.1 Cursor IDE

```jsonc
// .cursor/rules (place in project root)
{
  "rules": [
    "Always use C11 standard; VLAs are forbidden",
    "No C++ exceptions in embedded code",
    "Functions exceeding 60 lines must be split",
    "Every malloc must have a corresponding free in the same scope",
    "Interrupt handler functions must be suffixed with _isr",
    "Comments in Chinese; variable/function names in English"
  ]
}
```

### 7.2 GitHub Copilot Project Context File

Place `.github/copilot-instructions.md` in the project root:

```markdown
## Project Context
- Platform: RK3588 / Debian 12
- Toolchain: aarch64-linux-gnu-gcc 11.3
- Coding standard: Linux Kernel Coding Style
- Forbidden: C++ exceptions, RTTI, dynamic libraries (static linking with .a)
- Memory: all heap allocations must have a failure-handling branch
```

---

## 8. Changelog

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2025-11 | Initial release covering driver, audio/video, WebRTC, build, and test scenarios |
