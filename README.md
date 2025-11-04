# 🔧 Troubles Encountered and Their Resolution

This document captures incidents and their resolutions during lab development.  
It complements the main **Virtualization Journey** by focusing on failures, diagnostics, and recovery steps.

---

## GPU Failure Investigation & Infrastructure Hardening

### 🛑 Incident Sequence
- **Symptom:** Original monitor began arcing/buzzing → later no signal output.  
- **POST Behavior:** Motherboard emitted 6 beeps (GPU fault).  
- **Actions Taken:**  
  - Removed/reinstalled M6000 GPU → no change.  
  - Tested with low‑cost AMD GPU → same issue.  
  - CMOS reset (battery replaced, 10 min unpowered) → system posted.
  - CMOS battery swap means BIOS configuration likely resets, use backup or notes, workstations default to RAID and likely wont detect operating system unless it was all configured in primary RAID configuration, (AHCI more common in domestic PCs).
  - Reinstalled M6000 → fully functional.  
- **Root Cause:** CMOS instability/battery fault, not GPU failure.  
- **Masking Factor:** Monitor failure coincidentally overlapped, misleading diagnosis.

### 🖥 GPU & Display Strategy
- Xeon workstation has no iGPU → discrete GPU mandatory for POST/RDP.  
- **Primary GPU (M6000):** CUDA compute only.  
- **Secondary GPU (AMD):** Display output + RDP continuity.  

### 🖧 Monitor & Cable Management
- Cable swapping identified as wear risk.  
- Extra monitor staged for backup PC (normally RDP‑only).  

### 📌 Lessons Learned
- Single‑monitor reliance = risk.  
- Symptom ≠ cause: validate with CMOS reset before assuming GPU failure.  
- Functional segregation: compute vs display GPUs.  
- Cable wear is a hidden failure vector.  
- Redundancy planning: spares for monitors and cables.  
- Operational resilience: document lineage, role separation, and hardware adaptation.  

---

## BIOS Defaults & Drive Encryption Instability

### 🛑 Incident Sequence
- **BIOS Update:** Defaulted to RAID mode → drives undetected until reset to AHCI.  
- **Encryption Attempts:**  
  - BitLocker (Windows) and LUKS (Linux) full‑disk encryption tested.  
  - After instability/reboots, recovery passwords failed despite correctness.  
  - Data unrecoverable.  

### 📌 Lessons Learned
- Enterprise boards often default to **RAID**, unlike consumer boards (AHCI).  
- Always **document baseline BIOS configs** (RAID/AHCI, bifurcation, virtualization flags).  
- **Full‑disk encryption is unforgiving**: a single mismatch or corruption can make valid passwords appear invalid.  
- **Likely root causes of failure:**  
  - **Ubuntu Server (LUKS):** Corruption likely due to improper shutdown/OS instability, especially when running from USB stick during KVM installation. Had complete USB drive failures before with VMware ESXi without encryption, so complexity of encryption likely to be no surprise in increasing failure likelihood.
  - **Windows BitLocker (HDD):** Should have been more stable on a 256 GB SSD, but instability likely due to Windows 10 version at time.
  - **Third attempt Bitlocker (M.2):** So far so good partition holding so issues, not sure if this is due to partition only instead of full second drive or M.2 drive used and not HHD.  
  - **Second mechanical drive (encrypted):** Likely same corruption cause — improper shutdown or instability.  
- **Safer approach:**  
  - Use encryption only for private data or portable encrypted drives.  
  - Avoid full‑disk encryption on unstable or experimental lab hardware.  
  - Back up LUKS headers (`cryptsetup luksHeaderBackup`) and BitLocker recovery keys separately.  

---

## CMOS & Configuration Dependencies

### 🛑 Incident Sequence
- **Observation:** BIOS resets (after CMOS battery failure) reverted settings to defaults.  
- **Impact:**  
  - RAID vs AHCI mismatch → drives undetected.  
  - Bifurcation disabled → ASUS Hyper card NVMe drives hidden.  
  - Virtualization flags reset → hypervisors failed to start.  
- **Resolution:** Restored documented baseline configs manually.  

### 📌 Lessons Learned
- Always maintain a **baseline configuration record** (RAID/AHCI, bifurcation, virtualization flags, boot order).  
- Keep **USB recovery media** staged for quick restoration.  
- Treat CMOS battery replacement as a potential “system‑wide reset” event.  

---
