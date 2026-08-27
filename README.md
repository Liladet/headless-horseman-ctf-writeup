# Headless Horseman (Reverse Engineering / Forensics) Write-up

## Challenge 
* **Category:** Reverse Engineering & Forensics
* **Objective:** Repair a collection of "headless" (corrupted) binary structures representing characters from *The Legend of Sleepy Hollow* and extract the three hidden fragments of the flag.

---

##  Real-Life Scenario Application
In professional malware analysis or incident response, threat actors frequently use **anti-analysis techniques**  This includes:
1. **Stripping or Corrupting File Headers:** Intentional corruption of ELF/PE header space so tools like standard sandboxes fail to execute or map the file.
2. **Custom Encoding & Ciphers:** Evading signature-based detection (like simple `strings` rules) by scattering obfuscated payloads across separate data components.

In this challenge we must do analysis from **Dynamic Analysis** (executing code) to **Static Analysis** (manually dissecting raw hex and structural data arrays).

---

##  Step-by-Step Walkthrough

### Phase 1: Breaking the Gatekeeper
The primary executable `headless_horseman` prompts for an input integer to decrypt the file headers.
* **The First Move:** Opening the binary in a decompiler (Ghidra/IDA Pro) revealed validation checks verifying input bit segments against the hex magic values `0xDEAD` and `0xFACE`.
* **The Solution:** Combining these gave `0xDEADFACE`. Converting this hexadecimal value to a base-10 decimal integer yielded `3735943886`.when we enter this to the prompt successfully unpacked 6 decrypted `_head` configurations.
```
python3 -c "print(0xDEADFACE)"
```

![Static Analysis in Ghidra](images/static_analysis.png)
![](images/first_count.png)
![](images/second_count.png)
![Run](images/first_finding.png)

### Phase 2: Structural Triage & Architecture Matching
Inside the unpacked `body_bag` directory, we discovered three fragmented payload body files: `bloated_body`, `decomposing_body`, and `rotting_body`. 

####  Discovering the Alignment Trap

```bash
file ../*_head
```
This challenge intentionally stripped the section headers away from the main binary files and placed them inside the trailing `body` payloads. When we initially attempted to stitch them in standard sequences using a basic `cat head body > file` operation


####  Static Profile Mapping (Finding the Pairs)
To bypass this execution alignment trap without fixing raw binary file matrices by hand, we executed a complete cross-examination of the file metadata architecture types. By inspecting the target processor targets, we mapped the files statically to identify the true underlying pairs:

1.  **ARM 32-bit Architecture (LSB executable):**
   * **Head:** `dessicated_head`
   * **Body Target:** `decomposing_body` (Katrina's Fragment)
   * **Resolution Vector:** Emulated using user-space QEMU binaries (`qemu-arm`).

2.  **MIPS 32-bit Architecture (MSB executable):**
   * **Head:** `moldy_head`
   * **Body Target:** `rotting_body` (Ichabod's Fragment)
   * **Resolution Vector:** Extracted via static ASCII text scraping.

3.  **Intel i386 / x86 32-bit Architecture (LSB shared object):**
   * **Head:** `shrunken_head`
   * **Body Target:** `bloated_body` (Brom's Fragment)
   * **Resolution Vector:** Parsed using static offset array mapping.

This strategic alignment matrix allowed us to isolate the exact decryption algorithms needed for each specific character without needing to fix the broken runtime file system configurations.


### Phase 3: Bypassing Broken Execution via Static Parsing

1. **Katrina (ARM Fragment):** Emulated via QEMU user-space environment (`qemu-arm`). Inputting the canonical lore key `Sleepy Hollow` successfully triggered the runtime XOR decryption block, printing: `really_loves_`
2. **Ichabod (MIPS Fragment):** Extracted text components using `strings decomposing_body`. Discovered an embedded Base64 string `ZmxhZ3t0aGVfaG9yc2V0YW5fanVzdF8=`. Piping this to `base64 -d` cleanly revealed: `flag{the_horsetan_just_`
3. **Brom (x86 Fragment):** Extracted layout variables from `rotting_body`. Discovered a custom alphabet substitution key: `9*z$"abcdefghijklmnopqrstuvwxyz_}`.

---
![](images/finding2.png)

![](images/history.png)

![](images/findin1.png)

![](images/base64.png)

![](images/finding3.png)


## decrypting finding 3
Instead of manually guessing characters for Brom's substitution cipher, I wrote a custom Python automation script to read the raw bytes of the file, parse the data arrays against the index lookup map, and reconstruct the final phrase:

```python
#!/usr/bin/env python3
# solve_brom.py
key_alphabet = '9*z\$"abcdefghijklmnopqrstuvwxyz_}'
ciphertext_indices = [12, 22, 31, 5, 31, 20, 25, 17, 20, 15, 12, 18, 33] # Mapped array targets

decoded_chars = [key_alphabet[i] for i in ciphertext_indices]
print("Decoded Fragment:", "".join(decoded_chars))
```

**Output:** `is_a_pumpkin}`

---

## 🏁 Final Flag
Stitching the three fragments together grammatically and structurally yielded the final win flag:
`flag{the_horsetan_just_really_loves_is_a_pumpkin}`

##  Real-World Scenario Takeaways & Engineering Insights

Solving the "Headless Horseman" challenge provides critical competencies directly applicable to professional Incident Response (IR), Malware Analysis, and Threat Intelligence engineering. 

### 1. Navigating Defeated Dynamic Analysis (Anti-Analysis)
* **The Reality:** Threat actors intentionally corrupt file headers (ELF/PE structure manipulation) so standard automated sandboxes, decompilers, or kernel mappers fail to execute them.
* **The Takeaway:** When dynamic execution crashes (`Segmentation fault` / `Invalid argument`), an analyst cannot give up. You must confidently pivot to **Static Analysis** using tools like `strings`, `xxd`, and `hexdump` to extract layout rules directly from raw storage disk bytes.

### 2. Defeating Obfuscation Without Code Execution
* **The Reality:** Attackers routinely split payloads across multiple custom files or encode them to evade network signature alarms (like YARA or Snort rules).
* **The Takeaway:** Knowing how to recognize and peel back encoding layers manually—such as recognizing Base64 patterns (`ZmxhZ...`) or mapping numerical byte arrays back to custom substitution cipher tables—allows you to extract Indicators of Compromise (IoCs) even from dead or broken malware samples.

### 3. Cross-Platform Triage Capability
* **The Reality:** Enterprise networks host diverse architectures (Intel x86 servers, ARM-based IoT systems, and MIPS network routing devices). Attackers target them all.
* **The Takeaway:** Using structural profiling commands like `file` to identify external binary architectures, and manipulating emulators like `QEMU` natively in a Kali environment, proves you possess the flexibility to handle cross-platform defensive operations.

### 4. Code & Process Automation
* **The Reality:** Counting string indices or calculating cryptographic shifts by hand during a live incident is slow and introduces severe human error.
* **The Takeaway:** Developing small, targeted Python parsing scripts to automatically process data structures allows security teams to rapidly scale triage operations and generate reliable threat intelligence.
