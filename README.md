# Troubles Encountered and Their Resolution

This document captures incidents and their resolutions during lab development.  
It complements the main **Virtualisation Journey** by focusing on failures, diagnostics, and recovery steps.

---

## GPU Failure Investigation & Infrastructure Hardening

### Incident Sequence
- **Symptom:** Original monitor began arcing/buzzing → later no signal output.  
- **POST Behavior:** Motherboard emitted 6 beeps (GPU fault). 
- **Monitor buzz caused my open circuit on power to backlight circuit unrelated to Motherboard GPU fault**

- **Actions Taken:**  
  - New Monitor obtained to replace damaged one and spare for use on backup PC to avoid sharing.
  - Removed/reinstalled NVIDIA M6000 GPU → no change.  
  - Tested with low‑cost AMD R7 430 GPU → same issue.  
  - CMOS reset (battery replaced after removing for 10 min unpowered, the installed) → system posted.
  - CMOS battery swap was likely cause as it was at least 3 years old when purchased, likely much older.
- This required AHCL configured as HP workstations default to RAID. In this case bios would not detect operating system unless it was all configured in primary RAID configuration, (AHCI more common in domestic PCs).
  - Initially remove the AMD GPU R7 as it wont work unless PCIe training and driver preload delay configured
  - Verified Secureboot and legacy boot options in BIOS configured as previously set.
  - Reinstalled M6000 → fully functional.  
- **Root Cause:** CMOS instability/battery fault, not GPU failure.  
- **Masking Factor:** Monitor failure coincidentally overlapped, misleading diagnosis.

### GPU & Display Strategy
- Xeon workstation has no iGPU → discrete GPU mandatory for POST/RDP.
- **Downgrade to secondary GPU (M6000):** CUDA compute only.  
- **Now Primary GPU (AMD):** Now primary display output + RDP continuity.  

### Monitor & Cable Management
- Cable swapping identified as wear risk and anoying.  
- Extra monitor staged for backup PC (normally RDP‑only).  

### Lessons Learned
- Single‑monitor reliance = complicates setup by, not just by cable swapping, but no spare monitor to test.
- Symptom ≠ cause: validate with CMOS reset before assuming GPU failure.
- GPU was fully functional despite the warnings unique to GPU failure.
- Functional segregation: compute vs display GPUs. AMD R7 GPU now used as main display GPU, M6000 mainly cuda support.  
- Cable swapping might add wear to GPU sockets could eventually lead to concern.  
- Redundancy planning: Monitors for each work station bought with new cables.  
- Operational resilience: document full recovery process, symptoms,resolution approach, detailed list of bios previous configuration with back up on USB.  

---
## BIOS Defaults & Drive Encryption Instability

### Incident Sequence
- **BIOS Update:** Defaulted to RAID mode → drives undetected until reset to AHCI.  
- **Encryption Attempts:**  
  - BitLocker (Windows) and LUKS (Linux) full‑disk encryption tested.  
  - After instability/reboots, recovery passwords failed despite correctness.  
  - Data unrecoverable. Test stage nothing of value lost.

### Lessons Learned
- Enterprise boards often default to **RAID**, unlike consumer boards (AHCI).  
- Always **document baseline BIOS configs** (RAID/AHCI, bifurcation, virtualization flags).  
- **Full‑disk encryption is unforgiving**: a single mismatch or corruption can make valid passwords appear invalid.  
- **Likely root causes of failure:**  
  - **Ubuntu Server (LUKS):** Corruption likely due to improper shutdown/OS instability, especially when running from USB stick during KVM. Had complete USB drive failures before with VMware ESXi without encryption, so complexity of encryption likely to be no surprise in increasing failure likelihood.
  - **A single VM crash in VMWare ESXI can lockup the hypervisor:** Seems to be extremely common when USB stick used to run hypervisor, resulting in complete loss of hypervisor.
  - **Windows BitLocker (HDD):** Should have been more stable on a 256 GB SSD, however instability Windows 10 can result in complete loss. 
  - **Third attempt Bitlocker (M.2):** partition or full M.2 drive, regardless suspect if Windows crashes it can corrupt encryption. 
  - **Second mechanical drive (encrypted):** Likely same corruption cause — improper shutdown or instability.  
- **Safer approach:**  
  - Use encryption only for private data or portable encrypted drives.  
  - Avoid full‑disk encryption on unstable or experimental lab hardware unless it's just a expendable test data.


  - Keep backups on a removeable drives.
  - Back up LUKS headers (`cryptsetup luksHeaderBackup`) and BitLocker recovery keys separately. Improves odd of recovery not eradicate it.

---

## CMOS & Configuration Dependencies

### Incident Sequence
- **Observation:** BIOS resets (after CMOS battery failure) reverted settings to defaults.  
- **Impact:**  
  - RAID vs AHCI mismatch → drives undetected.  
  - Bifurcation disabled → ASUS Hyper card NVMe drives hidden.  
  - Virtualization flags reset → hypervisors failed to start.  
- **Resolution:** Restored documented baseline configs manually.  

### Lessons Learned
- Always maintain a **baseline configuration record** (RAID/AHCI, bifurcation, virtualization flags, boot order).  
- Keep **USB recovery media** staged for quick restoration.  
- Treat CMOS battery replacement as a potential “system‑wide reset” event.  

## AMD GPU Configuration initial use fails to display output
- Requires the legacy configuration to be configured for drivers in boot sequence and preload delay configured
- If for any reason BIOS is reset GPU will fail to work until BIOS reconfigured.
- Seems to be issue with AMD R7 with 2 DP outputs but not DP and VGA.
- **Likely Root Cause:**
  - Initial install HP wont work by default. GPU driver preload delay needs to be configured and so does PCIE training on HP Z640.
- **Resolution:**
  - Reconfigure the BIOS to set PCIE training mode and driver preload before the POST begins.
  - Note this in configuration notes.   
---

## Power supply Issue
- Instability of power, I believe is the area in general, Very short power outages shut down PC.
- Suspect micro fluctuations in power in area usually happen when I am at work regularly find oven and central heating clocks flashing. Rare when at home during day.
- **Resolution:**
  - UPS power supply delivering enough power for 2 PC and 2 monitors and switch obtained
  - Micro outages are logged when the PC that monitors the UPS management is on. Generally not required to run systems all day so this is simply not done.
  - Has highlighted more short less than second outages than expected in area. Area has only had one power outage for longer than 2 hours in times lived here.
  - Requires a few applications replaced that can block PC from shutting down when shutdown process activated by USP, like notepad.exe. 
