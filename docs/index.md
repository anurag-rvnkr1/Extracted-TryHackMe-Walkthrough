---
layout: default
title: "Extracted | TryHackMe Digital Forensics Walkthrough"
description: "Digital forensics and incident response investigation covering PCAP analysis, PowerShell malware, TCP stream reconstruction, Windows memory forensics, KeePass artifact recovery, and defensive detection."
---

<div class="ctf-hero">

  <h1>Extracted</h1>

  <p>
    A Digital Forensics and Incident Response (DFIR) investigation reconstructing
    a credential-theft workflow from captured network traffic, PowerShell
    malware, Windows process memory, and an encrypted KeePass password vault.
  </p>

  <div class="ctf-badges">
    <span class="ctf-badge">TryHackMe</span>
    <span class="ctf-badge">Digital Forensics</span>
    <span class="ctf-badge">Network Analysis</span>
    <span class="ctf-badge">Memory Forensics</span>
    <span class="ctf-badge">Incident Response</span>
  </div>

</div>

## Mission

The objective of this investigation is to reconstruct the documented credential-theft activity by correlating network evidence, reverse-engineering a PowerShell payload, recovering encoded artifacts, examining a Windows MiniDump, and validating access to a recovered KeePass database.

The investigation follows the evidence rather than exploiting a live application. It demonstrates how network forensics, malware analysis, memory analysis, and credential validation can be combined to reconstruct attacker behavior in an authorized TryHackMe laboratory.

## Quick Overview

<div class="ctf-card-grid">

  <div class="ctf-card">
    <div class="ctf-card-title">Platform</div>
    <div class="ctf-card-value">TryHackMe</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Category</div>
    <div class="ctf-card-value">Digital Forensics and Incident Response</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Difficulty</div>
    <div class="ctf-card-value">Medium</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Primary Evidence</div>
    <div class="ctf-card-value">PCAPNG and Windows MiniDump</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Target Application</div>
    <div class="ctf-card-value">KeePass Password Manager</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Investigation Status</div>
    <div class="ctf-card-value">Completed — Secrets Redacted</div>
  </div>

</div>

## Investigation Snapshot

| Investigation Metric | Documented Value |
|---|---|
| Primary evidence | Network packet capture (`.pcapng`) |
| Malware artifact | PowerShell script |
| Network protocols | HTTP and TCP |
| TCP ports investigated | 1337, 1338, and 1339 |
| Memory artifact | Windows MiniDump |
| Credential store | KeePass database (`.kdbx`) |
| Process under investigation | `KeePass.exe` |
| Encoding and obfuscation | Base64 and single-byte XOR |
| Credential analysis | Memory string extraction and candidate validation |
| Final objective | KeePass vault authentication; flag redacted |

## Navigation

<div class="ctf-toc">
<div class="ctf-toc-title">Documentation Map</div>
<ul>
  <li><a href="#mission">Mission</a></li>
  <li><a href="#quick-overview">Quick Overview</a></li>
  <li><a href="#assessment-objectives">Assessment Objectives</a></li>
  <li><a href="#target-and-lab-information">Target and Lab Information</a></li>
  <li><a href="#tools-used">Tools Used</a></li>
  <li><a href="#lab-architecture">Lab Architecture</a></li>
  <li><a href="#reconnaissance">Reconnaissance</a></li>
  <li><a href="#service-enumeration">Service Enumeration</a></li>
  <li><a href="#web-application-enumeration">Web Application Enumeration</a></li>
  <li><a href="#vulnerability-discovery">Vulnerability Discovery</a></li>
  <li><a href="#exploitation">Exploitation</a></li>
  <li><a href="#initial-access">Initial Access</a></li>
  <li><a href="#post-exploitation">Post-Exploitation</a></li>
  <li><a href="#credential-discovery">Credential Discovery</a></li>
  <li><a href="#privilege-escalation">Privilege Escalation</a></li>
  <li><a href="#attack-chain-summary">Attack Chain Summary</a></li>
  <li><a href="#security-findings">Security Findings</a></li>
  <li><a href="#detection-engineering">Detection Engineering</a></li>
  <li><a href="#defensive-recommendations">Defensive Recommendations</a></li>
  <li><a href="#lessons-learned">Lessons Learned</a></li>
  <li><a href="#conclusion">Conclusion</a></li>
</ul>
</div>

## Investigation Objectives

The investigation is organized around two complementary areas.

### Network Forensics and Malware Analysis

- Analyze the packet capture and identify suspicious HTTP activity.
- Investigate the PowerShell payload delivered over HTTP.
- Reconstruct TCP conversations and recover transmitted artifacts.
- Decode Base64-encoded data and reverse the documented XOR transformations.
- Correlate network traffic with the observed credential-theft workflow.

### Memory and Credential Forensics

- Validate the recovered Windows MiniDump.
- Extract UTF-16LE strings from the memory artifact.
- Identify candidate credential material.
- Validate candidate passwords against the recovered KeePass database.
- Confirm vault authentication without publishing sensitive information.
- Document defensive detection opportunities and incident response actions.

## Evidence Correlation and Attack Chain

The original investigation documents a workflow involving PowerShell execution, KeePass process memory acquisition, database collection, artifact obfuscation, and network exfiltration.

<div class="attack-chain">
  <div class="attack-step">PCAPNG Analysis</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">HTTP Payload</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">PowerShell Analysis</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">TCP Stream Recovery</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Base64 + XOR Reversal</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Memory Forensics</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Credential Validation</div>
  <div class="attack-arrow">→</div>
  <div class="attack-step">Vault Authentication</div>
</div>

### Artifact Correlation Diagram

```text
                    Packet Capture (.pcapng)
                              |
                              v
                HTTP PowerShell Payload Delivery
                              |
                              v
                 PowerShell Malware Investigation
                              |
                +-------------+-------------+
                |                           |
                v                           v
       TCP Stream — Port 1337      TCP Stream — Port 1338
       KeePass Process Dump        KeePass Database Vault
                |                           |
                v                           v
           Base64 Decode               Base64 Decode
                |                           |
                v                           v
              XOR 0x41                    XOR 0x42
                |                           |
                v                           v
         Windows MiniDump            KeePass Database
                |                           |
                +-------------+-------------+
                              |
                              v
                   Memory Credential Analysis
                              |
                              v
                 Candidate Password Validation
                              |
                              v
                   KeePass Vault Authentication
                              |
                              v
                   Flag Validation (Redacted)
```

The diagram summarizes the reported investigation flow. The exact packet contents, recovered credentials, and flag remain redacted where indicated in the source documentation.

## Investigation Timeline

| Phase | Investigation Area | Documented Outcome |
|---|---|---|
| 01 | Packet capture analysis | Suspicious HTTP PowerShell delivery identified |
| 02 | PowerShell reverse engineering | Artifact collection and exfiltration behavior investigated |
| 03 | TCP stream reconstruction | Encoded artifacts recovered |
| 04 | Base64 and XOR recovery | KeePass database and process dump reconstructed |
| 05 | Windows memory forensics | Candidate credential material extracted |
| 06 | Credential validation | Candidate password validated against the vault |
| 07 | Vault authentication | KeePass authentication reported successful |
| 08 | Defensive analysis | Indicators, detection opportunities, and response actions documented |

---

## Phase 1 — Network Forensics

> The investigation begins with the available network evidence. The packet capture is used to reconstruct activity without attacking a live target.

### Initial Evidence Acquisition

The primary evidence source is a PCAPNG file captured from a Windows workstation communicating with an attacker-controlled host in the lab scenario.

| Evidence Source | Investigation Goal |
|---|---|
| HTTP traffic | Identify suspicious payload delivery |
| TCP streams | Recover encoded artifacts |
| Host metadata | Correlate PowerShell activity |
| Binary payloads | Reconstruct the process dump and database |

### Investigation Strategy

The initial analysis addresses the following questions:

- Which protocols and services are visible in the capture?
- Is there evidence of PowerShell-related HTTP activity?
- Were files transferred through HTTP?
- Which TCP ports carry suspicious traffic?
- Which streams contain encoded or binary artifacts?

The reported findings include HTTP PowerShell delivery, TCP traffic involving ports 1337 and 1338, and a separate reference to port 1339 for payload delivery infrastructure.

### Evidence — HTTP PowerShell Delivery

![Wireshark evidence of PowerShell payload delivery](assets/01_wireshark_powershell_delivery.png)

*Figure 01 — Wireshark evidence used to investigate the reported HTTP PowerShell payload delivery.*

### Wireshark Evidence Breakdown

| Packet Observation | Investigation Interpretation |
|---|---|
| HTTP GET request | Investigated as a potential script download |
| `.ps1` filename | Indicates a PowerShell script artifact |
| Windows PowerShell User-Agent | Provides context for the reported PowerShell-related network activity |
| HTTP response body | Examined for PowerShell code and behavior |
| Internal lab host | Used to contextualize the payload source |

These observations describe the findings reported in the source documentation. The original packet capture remains the authoritative source for verifying individual packet fields and endpoints.

### Network Traffic Analysis Workflow

```text
PCAPNG Evidence
      |
      v
Inspect HTTP Traffic
      |
      v
Identify PowerShell Request
      |
      v
Examine Response Body
      |
      v
Analyze Payload Behavior
      |
      v
Investigate Related TCP Streams
```

### Network Forensics Findings

| Finding | Security Significance |
|---|---|
| PowerShell script delivered over HTTP | Identifies a suspicious payload-delivery stage |
| Separate TCP channels | Supports investigation of artifact transfer and exfiltration |
| Encoded payloads | Indicates data was transformed before transmission |
| Uncommon TCP ports | Provides pivots for network investigation |

---

## Phase 2 — PowerShell Malware Analysis

> Static analysis of the recovered PowerShell script provides context for the network artifacts and the suspected credential-theft workflow.

### Why Analyze the Payload?

The script is investigated to determine:

- Which process and files were targeted.
- Whether KeePass process memory was collected.
- How the database and memory dump were transformed.
- Which network ports were associated with artifact transmission.

### Documented Malware Behavior

```text
PowerShell Execution
        |
        v
Locate KeePass Process
        |
        +-----------> Create Process Memory Dump
        |
        +-----------> Read KeePass Database
                           |
                           v
                    XOR Transformation
                           |
                           v
                    Base64 Encoding
                           |
                           v
               TCP Ports 1337 / 1338
```

### Malware Capability Matrix

| Capability | Investigation Significance |
|---|---|
| Process enumeration | Identifies the KeePass process |
| Memory dump creation | Produces a Windows process memory artifact |
| File access | Collects the encrypted KeePass database |
| XOR transformation | Obfuscates artifact bytes |
| Base64 encoding | Converts binary data into a text-compatible representation |
| TCP communication | Provides a path for transferring collected artifacts |

### Artifact Obfuscation Workflow

**Process memory dump**

```text
Windows MiniDump
       |
       v
XOR 0x41
       |
       v
Base64
       |
       v
TCP/1337
```

**KeePass database**

```text
Database1337.kdbx
       |
       v
XOR 0x42
       |
       v
Base64
       |
       v
TCP/1338
```

The documented recovery process reverses the transformations to reconstruct the original binary artifacts.

### Malware Analysis Findings

| Finding | DFIR Significance |
|---|---|
| KeePass is the targeted process | Establishes the credential-theft objective described in the scenario |
| XOR keys are documented | Makes byte-level artifact reconstruction possible |
| TCP ports are identified | Narrows the stream investigation |
| Base64 is used for encoding | Allows the encoded data to be decoded before reversing XOR |

---

## Phase 3 — TCP Stream Reconstruction

> TCP stream reconstruction reassembles the captured conversation so that transmitted artifacts can be examined as complete data rather than isolated packets.

### Why Follow TCP Stream?

Wireshark's TCP stream functionality presents the reassembled conversation, helping an investigator:

- Examine the ordered application data.
- Reconstruct the captured conversation.
- Inspect binary or encoded payloads.
- Export a stream for further analysis.

TCP stream reconstruction does not itself reverse application-layer encoding or guarantee that a stream contains a complete file. The recovered output must still be validated.

### Documented Stream Mapping

| TCP Port | Reported Artifact or Role |
|---:|---|
| `1337` | KeePass process dump |
| `1338` | KeePass database |
| `1339` | PowerShell delivery infrastructure, as described in the original documentation |

The source associates the recovered process dump and database with ports 1337 and 1338 respectively. Port 1339 is separately identified in the source as part of the payload-delivery infrastructure.

### Wireshark Recovery Procedure

1. Locate relevant packets using a TCP display filter.
2. Select a packet from the conversation.
3. Right-click and choose **Follow → TCP Stream**.
4. Select the **Raw** representation where appropriate.
5. Save the reconstructed payload for analysis.

The stream should be checked for the expected encoding and artifact boundaries before attempting binary reconstruction.

### CLI Recovery with TShark

The original documentation includes the following command for inspecting TCP payload fields associated with port 1338:

```bash
tshark -r traffic.pcapng \
-Y "tcp.port == 1338" \
-T fields \
-e tcp.payload | tr -d '\n'
```

**Purpose:** Read the capture, filter packets involving TCP port 1338, and output the TCP payload field.

**Important limitation:** This command concatenates displayed payload values; it is not, by itself, a complete TCP reassembly procedure. Packet ordering, retransmissions, stream direction, and payload formatting must be handled correctly. For reliable recovery, use Wireshark's Follow TCP Stream functionality or an appropriate stream-aware extraction method.

The original documentation also provides this hexadecimal conversion command:

```bash
xxd -r -p combined_hex_1338.txt > stream_1338.raw
```

**Purpose:** Convert a text file containing hexadecimal byte values into a binary output file.

**Expected output:** `stream_1338.raw`, provided the input contains the correctly reconstructed hexadecimal stream.

Repeat the corresponding recovery workflow for TCP port 1337.

### Artifact Recovery Summary

| Output Artifact | Reported Purpose |
|---|---|
| `stream_1337.raw` | Encoded data associated with the Windows process dump |
| `stream_1338.raw` | Encoded data associated with the KeePass database |

### Common DFIR Pitfalls

| Mistake | Correct Investigation Practice |
|---|---|
| Exporting individual packets as though each were a complete file | Reconstruct the complete stream |
| Treating hexadecimal output as an already recovered binary | Convert it correctly and validate the result |
| Ignoring retransmissions or packet ordering | Use stream-aware reconstruction |
| Assuming that every TCP payload belongs to one file | Confirm stream direction, boundaries, and application protocol |
| Continuing without validating the output | Check file signatures and artifact-specific structure |

---

## Phase 4 — Artifact Recovery

> The next stage reverses the documented Base64 and XOR transformations to reconstruct the original forensic artifacts.

### Reconstruction Pipeline

```text
Recovered TCP Stream
        |
        v
Encoded Payload
        |
        v
Base64 Decode
        |
        v
Reverse Single-Byte XOR
        |
        v
Recovered Binary Artifact
        |
        v
File-Type and Content Validation
```

### Understanding the XOR Transformation

The source documents two single-byte XOR keys:

- `0x41` for the process memory dump.
- `0x42` for the KeePass database.

XOR is a reversible byte transformation, not encryption that independently provides confidentiality. Applying the same key to the transformed bytes reverses the operation.

### Recovery Workflow

| Input | Transformation | Output |
|---|---|---|
| `stream_1337.raw` | Base64 decode, then XOR `0x41` | `keepprocess.dmp` |
| `stream_1338.raw` | Base64 decode, then XOR `0x42` | `Database1337.kdbx` |

### Python Recovery Logic

The original documentation supplies this conceptual recovery operation:

```python
decoded = base64.b64decode(stream)

recovered = bytes(byte ^ xor_key for byte in decoded)
```

The snippet illustrates the byte transformations. A working implementation also needs to import the appropriate library, load the correct stream, select the documented key, and write the recovered bytes to the intended output file.

### Artifact Validation

| Tool or Method | Purpose |
|---|---|
| `file` | Identify the recovered file format |
| `strings` | Inspect printable strings |
| `strings -el` | Extract UTF-16LE strings commonly found in Windows artifacts |

For example:

```bash
file keepprocess.dmp
```

```bash
strings -el keepprocess.dmp
```

The first command checks the file type. The second extracts UTF-16LE strings that may help identify process metadata and candidate credential material.

### Recovered Investigation Artifacts

| Artifact | Investigation Role |
|---|---|
| `Database1337.kdbx` | Recovered encrypted KeePass credential vault |
| `keepprocess.dmp` | Recovered Windows process memory dump |

The file names are preserved from the source documentation.

---

## KeePass Credential-Theft Architecture

Understanding the difference between an encrypted database and a running process is important to interpreting the recovered evidence.

### Credential Collection Workflow

```text
User Unlocks KeePass
         |
         v
Master Password Processed
         |
         +---------------------------+
         |                           |
         v                           v
Encrypted Database            Runtime Process Memory
(.kdbx on Disk)                (KeePass.exe)
         |                           |
         v                           v
Encrypted Vault Data          Runtime Secret Material
         |                           |
         +-------------+-------------+
                       |
                       v
              Attacker Collects Both
```

### KeePass Security Components

| Component | Purpose |
|---|---|
| KDBX database | Stores the password vault in encrypted form |
| Master password | Primary secret used to unlock the vault |
| Key derivation function | Derives key material from password input |
| Composite key | Represents the combined key material used when additional key components are configured |
| Process memory | Contains runtime state and may expose sensitive material during execution |

The source mentions AES and Argon2 in its discussion of KeePass security. These are examples of encryption and key derivation mechanisms; the actual configuration of the recovered database is not established by the supplied documentation.

### Why Memory Matters

| Encrypted Database Alone | Database Plus Process Memory |
|---|---|
| Vault contents remain encrypted | Memory may contain additional runtime evidence |
| Authentication material is needed to unlock the vault | Recovered strings may provide credential candidates |
| Password verification can require computationally expensive derivation | Memory evidence can provide a different investigative route |

A memory dump does not guarantee that a complete master password or plaintext vault contents will be present. Any recovered candidate must be independently validated.

### Memory Analysis Evidence

![KeePass memory dump extraction evidence](assets/02_keepass_dump_extraction.png)

*Figure 02 — The source documentation identifies this image as evidence associated with KeePass process memory analysis.*

---

## Phase 5 — Windows Memory Forensics

> This phase investigates the recovered MiniDump for runtime evidence associated with the KeePass process.

### Memory Forensics Overview

<div class="ctf-card-grid">

  <div class="ctf-card">
    <div class="ctf-card-title">Memory Artifact</div>
    <div class="ctf-card-value">Windows MiniDump</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Target Process</div>
    <div class="ctf-card-value">KeePass.exe</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">String Encoding</div>
    <div class="ctf-card-value">UTF-16LE</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Objective</div>
    <div class="ctf-card-value">Investigate Runtime Credential Material</div>
  </div>

</div>

### Why Analyze Memory Instead of Only the Database?

An encrypted database and a process memory dump provide different evidence.

The database is intended to protect stored secrets while locked. Runtime memory can contain process state and other sensitive material while the application is running. The contents depend on the process state, application implementation, and the moment of acquisition.

### Memory Acquisition Workflow

The source attributes the MiniDump to the recovered PowerShell payload.

| Investigation Stage | Associated Evidence |
|---|---|
| KeePass running | Runtime process state |
| PowerShell execution | Reported process dump creation |
| XOR transformation | Obfuscated binary artifact |
| Base64 encoding | Transport representation |
| TCP stream reconstruction | Recovered encoded artifact |
| Artifact decoding | Reconstructed MiniDump |
| Memory analysis | Candidate credential material |

### Evidence — Memory Artifact

![KeePass process memory evidence](assets/img/02_keepass_dump_extraction.png)

*Figure 02 — Alternate image path preserved from the source documentation for the memory dump investigation.*

### Memory Validation Pipeline

Validate the reconstructed artifact before interpreting its contents.

```bash
file keepprocess.dmp
```

The command checks whether the output is recognized as a Windows MiniDump or another identifiable file format.

```bash
strings -el keepprocess.dmp
```

The command extracts UTF-16LE strings for inspection.

### Documented Memory Findings

| Recovered Evidence | Investigation Value |
|---|---|
| KeePass executable references | Helps establish relevance to the target process |
| UTF-16LE strings | Provides candidate password fragments for further validation |
| Module metadata | Can help contextualize the process artifact |
| Heap-memory strings | May provide leads for credential analysis |

The supplied material reports recovery of candidate password material. The exact recovered strings are intentionally not reproduced here.

---

## Phase 6 — Credential Recovery and Validation

> Credential candidates recovered from memory are treated as hypotheses until validated against the recovered KeePass database.

### Credential Recovery Workflow

```text
Recovered MiniDump
        |
        v
Unicode String Extraction
        |
        v
Candidate Password Material
        |
        v
Targeted Candidate Generation
        |
        v
Cryptographic Validation
        |
        v
KeePass Password Verification
```

### Candidate Generation

The source describes a targeted candidate-generation approach for cases where memory evidence contains partially reconstructed strings.

The documented methodology is to:

- Preserve the characters recovered from memory.
- Identify unresolved characters.
- Generate a bounded candidate list.
- Validate candidates against the database.
- Avoid treating an unverified memory string as a confirmed password.

### Validation Strategy

```text
Recovered Candidate
        |
        v
Identify Unknown Characters
        |
        v
Generate Candidate List
        |
        v
Convert Database to Verification Format
        |
        v
Validate Candidate Passwords
        |
        v
Confirm Successful Authentication
```

### Password Validation Tools

| Tool | Documented Purpose |
|---|---|
| `keepass2john` | Convert a compatible KeePass database into a John the Ripper verification format |
| John the Ripper (`john`) | Validate password candidates against the converted database representation |
| `keepassxc-cli` | Authenticate against the recovered KeePass vault |

The source reports candidate validation using John the Ripper and successful vault authentication. Exact passwords and secret values remain redacted.

### Public Portfolio Redaction

| Sensitive Item | Public Representation |
|---|---|
| KeePass master password | `[REDACTED]` |
| Recovered flag | `THM{FLAG_REDACTED}` |
| Vault UUID | `[REDACTED]` |

The original documentation intentionally withholds these values. No attempt is made to reconstruct them.

---

## Phase 7 — KeePass Vault Authentication

> The final credential-validation stage confirms whether the recovered candidate permits access to the encrypted vault.

### Authentication Status

<div class="ctf-card-grid">

  <div class="ctf-card">
    <div class="ctf-card-title">Database</div>
    <div class="ctf-card-value">Recovered</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Credential</div>
    <div class="ctf-card-value">Reported as Validated</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Authentication</div>
    <div class="ctf-card-value">Reported Successful</div>
  </div>

  <div class="ctf-card">
    <div class="ctf-card-title">Sensitive Values</div>
    <div class="ctf-card-value">Redacted</div>
  </div>

</div>

### Evidence — Vault Recovery

![KeePass vault authentication evidence](assets/03_keepass_vault_recovery.png)

*Figure 03 — The original documentation identifies this screenshot as evidence of successful KeePassXC CLI authentication.*

### Vault Investigation Summary

| Validation Step | Source-Reported Status |
|---|---|
| Encrypted database recovered | Complete |
| Candidate password validated | Complete |
| Vault authenticated | Complete |
| Challenge entry located | Complete |
| Challenge flag published | Redacted |

### Credential Recovery Architecture

```text
Recovered KeePass Database
          |
          v
Database Validation
          |
          v
Master Password Candidate
          |
          v
Cryptographic Verification
          |
          v
Vault Authentication
          |
          v
Protected Entry Access
```

The reported authentication result is preserved from the original walkthrough. No sensitive vault contents are disclosed.

---

## Recovered Evidence Inventory

The original documentation identifies the following artifacts as part of the investigation.

| Artifact | Description |
|---|---|
| `traffic.pcapng` | Primary packet capture |
| `xxxmmdcclxxxiv.ps1` | Recovered PowerShell payload |
| `stream_1337.raw` | Encoded stream associated with the Windows memory dump |
| `stream_1338.raw` | Encoded stream associated with the KeePass database |
| `keepprocess.dmp` | Recovered Windows MiniDump |
| `Database1337.kdbx` | Recovered KeePass database |

### Investigation Success Matrix

| Objective | Source-Reported Status |
|---|---|
| HTTP payload identified | Complete |
| PowerShell payload analyzed | Complete |
| TCP streams reconstructed | Complete |
| Artifacts decoded | Complete |
| Windows memory dump validated | Complete |
| Credential candidate recovered | Complete |
| KeePass vault authenticated | Complete |

---

## Indicators of Compromise

The following table consolidates the indicators documented in the source. These are investigation pivots, not independently verified production indicators.

### Network Indicators

| Indicator | Reported Significance |
|---|---|
| HTTP PowerShell download | Initial payload-delivery activity |
| PowerShell User-Agent | Context for the observed PowerShell-related network traffic |
| TCP port 1337 | Associated with process dump exfiltration |
| TCP port 1338 | Associated with KeePass database exfiltration |
| TCP port 1339 | Identified in the source as related to payload delivery infrastructure |

### Host Indicators

| Artifact or Behavior | Security Significance |
|---|---|
| `Database1337.kdbx` | Indicates collection of the encrypted password vault |
| `keepprocess.dmp` | Indicates acquisition of process memory |
| PowerShell execution | Associated with the documented malware workflow |
| Base64 and XOR transformations | Provide artifact-recovery and detection pivots |

The source does not provide a verified external IP address, domain name, file hash, or complete endpoint list. None is invented here.

---

## MITRE ATT&CK Coverage

The original walkthrough includes the following ATT&CK mappings. They are retained as the source's proposed coverage; exact technique applicability should be verified against the recovered script and underlying evidence.

| Tactic | Technique | ID | Documented Context |
|---|---|---|---|
| Execution | PowerShell | `T1059.001` | PowerShell payload execution |
| Execution | Command and Scripting Interpreter | `T1059` | Script-based execution |
| Credential Access | OS Credential Dumping | `T1003` | Process memory and credential-material investigation |
| Credential Access | Credentials from Password Stores | `T1555` | KeePass database targeting |
| Defense Evasion | Obfuscated Files or Information | `T1027` | XOR and Base64 transformations |
| Collection | Data from Local System | `T1005` | Collection of local database and process artifacts |
| Exfiltration | Exfiltration Over C2 Channel | `T1041` | Transfer of collected artifacts over TCP |

These mappings are not all equally specific. In particular, generic process dumping does not by itself establish a specific `T1003` sub-technique, and Base64/XOR use does not automatically establish adversarial intent to evade detection. The source evidence should determine the final mapping.

### Documented Attack Lifecycle

```text
PowerShell Execution
        |
        v
Credential Collection
        |
        v
KeePass Process Dump
        |
        v
Base64 + XOR Transformation
        |
        v
Network Artifact Transfer
        |
        v
Credential Validation
```

---

## Detection Engineering and Threat Hunting

The investigation also identifies opportunities for blue-team detection using endpoint telemetry, process-access monitoring, network logs, and SIEM queries.

### Detection Coverage Matrix

| Attack Stage | Detection Opportunity | Visibility Consideration |
|---|---|---|
| PowerShell download | PowerShell logging and proxy telemetry | Depends on logging configuration |
| PowerShell execution | Process-creation telemetry | Requires suitable endpoint audit coverage |
| KeePass process access | EDR process-access monitoring | Depends on available endpoint events |
| Process dump creation | Process-access and dump-creation telemetry | Requires appropriate collection and detection logic |
| Base64-encoded traffic | Proxy, IDS, and network traffic analysis | Encoded traffic alone is not proof of malicious activity |
| TCP ports 1337 and 1338 | Firewall and NetFlow monitoring | Requires contextual analysis of destination and behavior |
| KeePass vault access | Endpoint process and application telemetry | Depends on the available data source |

### Sigma Detection Opportunities

The following rules preserve the original detection concepts. They are starting points, not production-ready detections. Sigma field names and log-source definitions must match the target telemetry schema.

#### Suspicious PowerShell Download

```yaml
title: Suspicious PowerShell HTTP Download
logsource:
  product: windows
  category: process_creation
detection:
  selection:
    Image|endswith: '\powershell.exe'
    CommandLine|contains:
      - 'http'
      - '.ps1'
  condition: selection
level: high
```

**Purpose:** Identify PowerShell process command lines containing HTTP-related and script-file indicators.

**Limitation:** The rule requires both indicators in the same command line and can generate false positives. It does not establish that a script was successfully downloaded or executed.

#### KeePass Process Access

```yaml
title: KeePass Process Memory Access
logsource:
  product: windows
  category: process_access
detection:
  selection:
    TargetImage|contains: 'KeePass.exe'
  condition: selection
level: high
```

**Purpose:** Highlight process-access events involving KeePass.

**Limitation:** Access to a process is not necessarily malicious. The event schema, source process, access rights, and process-access telemetry must be reviewed before deployment.

#### Encoded PowerShell Execution

```yaml
title: Encoded PowerShell Command
logsource:
  product: windows
  category: process_creation
detection:
  selection:
    CommandLine|contains:
      - '-EncodedCommand'
      - 'FromBase64String'
  condition: selection
level: medium
```

**Purpose:** Identify command lines containing common Base64-related PowerShell indicators.

**Limitation:** These strings can occur in legitimate administrative scripts. Validate the rule against the environment's logging format and baseline.

### Microsoft Defender XDR — KQL

The source provides the following hunting examples.

#### Hunt PowerShell Script Downloads

```kusto
DeviceProcessEvents
| where ProcessCommandLine contains ".ps1"
| where ProcessCommandLine contains "http"
```

**Purpose:** Search process-creation telemetry for command lines containing both a PowerShell script extension and an HTTP indicator.

#### Hunt KeePass Process Access

```kusto
DeviceEvents
| where AdditionalFields contains "KeePass"
```

**Purpose:** Search additional event fields for KeePass-related references.

**Limitation:** The broad string search may return unrelated events. The relevant event types and available fields should be confirmed in the tenant.

#### Hunt Suspicious PowerShell

```kusto
DeviceProcessEvents
| where FileName =~ "powershell.exe"
| where ProcessCommandLine contains "Base64"
```

**Purpose:** Identify PowerShell process events whose command lines contain a Base64-related string.

### Splunk Hunting Example

```spl
index=windows EventCode=4688
Image="*powershell.exe"
CommandLine="*.ps1*"
```

**Purpose:** Search Windows process-creation events for PowerShell and script-related command-line indicators.

**Limitation:** Field names depend on the Splunk data model and ingestion configuration. Validate the fields against the actual events.

### Elastic / Sysmon Hunting Example

```kql
process.name : "powershell.exe" and
process.command_line : "*.ps1*"
```

**Purpose:** Search process telemetry for PowerShell executions with script-related command-line content.

**Limitation:** The field mappings and wildcard behavior depend on the configured Elastic data view and query language.

### Network Detection Opportunities

| Indicator | Defensive Investigation |
|---|---|
| HTTP-delivered PowerShell payload | Inspect proxy logs, download records, and script content |
| Large or unusual Base64-like TCP payloads | Examine outbound volume, destination, and surrounding session context |
| TCP ports 1337 and 1338 | Investigate unusual outbound connections and associated processes |
| PowerShell User-Agent | Compare the traffic with legitimate administrative activity |

---

## Incident Response Playbook

The following response workflow is based on the investigation activities and response recommendations in the original documentation.

### Phase 1 — Detection

- Investigate alerts involving suspicious PowerShell network communication.
- Identify the associated host, user, process, and destination.
- Preserve the packet capture and relevant network logs.
- Record timestamps and evidence provenance.

### Phase 2 — Containment

- Isolate the affected workstation when justified by the incident assessment.
- Restrict suspicious outbound communications.
- Preserve volatile memory when feasible and appropriate.
- Collect PowerShell and endpoint telemetry before evidence is lost.

### Phase 3 — Investigation

- Recover and inspect the PowerShell payload.
- Reconstruct relevant TCP streams.
- Decode the documented Base64 and XOR transformations.
- Validate recovered file types.
- Analyze the MiniDump and correlate candidate credentials with the KeePass database.
- Record findings and maintain evidence integrity.

### Phase 4 — Eradication

- Remove confirmed malicious artifacts.
- Rotate credentials that may have been exposed.
- Assess whether the KeePass vault must be replaced or its secrets reissued.
- Investigate potential persistence and other affected systems.

### Phase 5 — Recovery

- Restore systems from trusted sources where necessary.
- Re-establish normal connectivity after containment and eradication checks.
- Validate the integrity of the recovered credential vault.
- Continue monitoring endpoint and network telemetry.

### Incident Response Timeline

```text
Alert Received
      |
      v
Preserve Packet Capture
      |
      v
Investigate PowerShell Payload
      |
      v
Reconstruct TCP Streams
      |
      v
Recover and Validate Artifacts
      |
      v
Analyze Process Memory
      |
      v
Validate Credential Evidence
      |
      v
Containment and Eradication
      |
      v
Recovery and Continued Monitoring
```

This is a recommended response sequence derived from the source's playbook, not a claim that each operational response action was executed during the CTF.

---

## Evidence Correlation and Findings

The investigation correlates several evidence types rather than relying on a single artifact.

| Evidence Source | Documented Finding |
|---|---|
| Packet capture | Suspicious HTTP PowerShell delivery identified |
| PowerShell script | Artifact collection and obfuscation behavior investigated |
| TCP streams | Encoded artifacts reconstructed |
| Windows MiniDump | Candidate credential material extracted |
| KeePass database | Vault recovered and authentication reported successful |
| Detection analysis | Host and network detection opportunities documented |

<div class="key-finding">

<div class="key-finding-title">Key Finding</div>

The source documents a credential-theft workflow in which an encrypted KeePass database and a Windows process memory dump were collected, transformed using Base64 and XOR, recovered from network streams, and used in a subsequent credential-validation workflow. The walkthrough reports successful vault authentication while intentionally redacting sensitive values.

</div>

## Security Recommendations

The recommendations below address the specific behaviors documented in the investigation.

| Observed Behavior | Security Risk | Recommended Control |
|---|---|---|
| HTTP delivery of PowerShell scripts | Unauthorized script execution | Restrict untrusted script downloads, inspect proxy telemetry, and monitor suspicious PowerShell execution |
| Unauthorized access to process memory | Exposure of runtime credential material | Use endpoint protection, restrict unnecessary process access, and investigate anomalous memory-access events |
| Collection of a KeePass database | Exposure of an encrypted credential vault | Protect vault files with appropriate access controls and investigate unexpected access |
| Base64 and XOR transformations | Concealment of data representation during transfer | Correlate process telemetry with outbound network activity instead of relying on encoding indicators alone |
| Suspicious outbound TCP sessions | Potential data exfiltration | Apply egress controls, destination monitoring, and alerting for unusual transfer patterns |
| Potentially exposed master password | Unauthorized vault access | Rotate exposed credentials and assess the need to reissue secrets stored in the vault |

### Credential Protection Considerations

The original documentation also identifies the following KeePass-related controls:

- Use a strong master password.
- Select an appropriate, supported key derivation function configuration.
- Use additional authentication components where supported and appropriate.
- Lock the password manager when idle.
- Review clipboard-history and clipboard-retention settings.
- Limit access to sensitive process memory and credential stores.

The effectiveness of these controls depends on the deployment, application configuration, operating environment, and threat model.

---

## Lessons Learned

### Network Forensics

- Packet captures can expose suspicious payload delivery and communication patterns.
- TCP stream reconstruction provides context that isolated packets may not.
- Network metadata can guide investigation pivots even when payloads are encoded.
- File reconstruction requires correct stream handling and artifact validation.

### Malware Analysis

- Static analysis can reveal collection and exfiltration behavior.
- Reversible XOR transformations should not be mistaken for secure encryption.
- Base64 is an encoding scheme, not a confidentiality mechanism.
- Payload analysis can identify the keys and transformations needed for artifact recovery.

### Memory Forensics

- Memory artifacts can complement encrypted files recovered from disk or network traffic.
- UTF-16LE extraction is useful when investigating Windows memory artifacts.
- A candidate string is not a confirmed credential until validated.
- Process dumps can contain sensitive information and must be handled carefully.

### Incident Response and Detection

- Correlating endpoint and network evidence improves investigative confidence.
- File signatures and content checks should be used to validate reconstructed artifacts.
- Volatile evidence may be lost if collection is delayed.
- Detection rules require schema validation, contextual tuning, and false-positive testing.
- Incident response should include containment, eradication, recovery, and post-incident monitoring.

---

## Skills Demonstrated

| Domain | Skills Practiced |
|---|---|
| Digital Forensics | Evidence analysis, artifact reconstruction, evidence correlation |
| Network Security | Wireshark, TShark, HTTP investigation, TCP stream analysis |
| Malware Analysis | PowerShell script inspection, XOR reversal, Base64 decoding |
| Memory Forensics | Windows MiniDump validation, UTF-16LE string extraction |
| Credential Analysis | Candidate generation, database verification, KeePass authentication |
| Blue Team Operations | IOC analysis, detection logic, SIEM hunting concepts |
| Incident Response | Containment planning, evidence preservation, remediation workflow |
| Technical Documentation | Structured investigation reporting, diagrams, evidence tables, security recommendations |

---

## Tools Used

Only the tools explicitly identified in the source documentation are listed below.

| Tool or Technology | Documented Purpose |
|---|---|
| Wireshark | Packet inspection and TCP stream reconstruction |
| TShark | Command-line packet capture analysis |
| PowerShell | Script behavior investigated in the recovered payload |
| Python 3 | Artifact decoding and byte-level transformation logic |
| `base64` | Decode the documented Base64 representation |
| `xxd` | Convert hexadecimal text to binary data |
| `file` | Validate recovered file types |
| `strings` | Extract printable strings from artifacts |
| `strings -el` | Extract UTF-16LE strings |
| `keepass2john` | Convert a compatible KeePass database for password verification |
| John the Ripper | Validate password candidates |
| `keepassxc-cli` | Authenticate against the recovered vault |
| Sigma | Express detection logic in a portable rule format |
| Microsoft Defender XDR / KQL | Endpoint threat-hunting examples |
| Splunk | SIEM hunting example |
| Elastic / KQL | Process-telemetry hunting example |
| Markdown and Jekyll | Portfolio documentation |

---

## References and Learning Resources

The original documentation identifies the following resources and learning areas. Specific URLs were not supplied for these references in the source material, so no replacement URLs are invented.

### Official Documentation

- Microsoft PowerShell Documentation
- Microsoft Windows MiniDump Documentation
- Wireshark User Guide
- TShark Documentation
- KeePass Documentation
- KeePassXC Documentation
- MITRE ATT&CK Framework

### Educational Resources

- TryHackMe — Digital Forensics learning path
- Windows incident response resources
- Network forensics learning materials

---

## Responsible Use and Redaction

This documentation describes activities performed in an authorized cybersecurity training environment. The techniques discussed should only be used against systems and evidence for which explicit permission has been granted.

The source intentionally redacts sensitive information to preserve the integrity of the challenge and protect credential material.

| Protected Information | Public Representation |
|---|---|
| TryHackMe flag | `THM{FLAG_REDACTED}` |
| KeePass master password | `[REDACTED]` |
| Vault UUID | `[REDACTED]` |
| Recovered credentials | `[REDACTED]` |

```text
THM{FLAG_REDACTED}
Master Password: [REDACTED]
Vault UUID: [REDACTED]
```

No missing credentials, flags, hashes, IP addresses, or hostnames have been reconstructed.

---

## Final Investigation Summary

<div class="key-finding">

<div class="key-finding-title">Investigation Outcome</div>

The original walkthrough reports completion of the artifact-recovery and KeePass authentication workflow. The public portfolio version preserves the methodology, documented findings, and defensive analysis while withholding the challenge flag and sensitive credential values.

</div>

### Investigation Achievements

- Investigated HTTP PowerShell payload delivery.
- Analyzed the documented malware behavior.
- Reconstructed TCP streams associated with the reported artifacts.
- Reversed the documented Base64 and XOR transformations.
- Recovered and validated a Windows MiniDump.
- Examined UTF-16LE strings for candidate credential material.
- Validated a candidate against the recovered KeePass database.
- Documented successful vault authentication as reported by the source.
- Consolidated indicators of compromise and ATT&CK mappings.
- Developed detection and threat-hunting examples.
- Documented incident response and remediation recommendations.

### Defensive Outcomes

- Correlated network, script, memory, and database evidence.
- Identified PowerShell execution and process-access monitoring opportunities.
- Documented network monitoring pivots for suspicious TCP transfers.
- Provided example Sigma, KQL, Splunk, and Elastic detection logic.
- Preserved sensitive challenge values in redacted form.

---

## About This Portfolio Project

This investigation is part of a cybersecurity portfolio documenting practical work in authorized training environments.

It demonstrates hands-on practice in:

- Digital forensics and incident response.
- Network traffic investigation.
- PowerShell malware analysis.
- Windows memory forensics.
- Credential artifact recovery.
- Threat detection and hunting.
- Security documentation and evidence correlation.

The emphasis is on reproducible investigative methodology, technical accuracy, and defensive learning rather than the public disclosure of challenge secrets.

