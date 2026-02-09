# Fixing OnVUE “Virtual Machine Detected” Error (When You’re Not Using a VM)

If you’re a developer taking an online proctored exam using Pearson OnVUE, you might hit this terrifying message during the system check:

> ❌ “A virtual machine has been detected. Please close all virtual machine software before continuing.”

The problem?

You’re not using a virtual machine.

That’s exactly what happened to me.

Here’s what I did to fix it.

---

## 🧨 The Issue

OnVUE’s system check failed because it believed my computer was running inside a virtual machine.

Reality:

- No VMware
- No VirtualBox
- No obvious VM running
- Just a normal Windows machine

But I *do* use:

- WSL  
- Docker  
- Visual Studio Code with container extensions  
- Hyper-V (occasionally)

And that’s likely where the problem started.

OnVUE doesn’t care whether you’re actively using a VM.  
If virtualization-related services are running in the background, it can fail the pre-check.

---

## 🛠 Step 1 – Disable Windows Virtualization Features

I opened **Command Prompt as Administrator** and ran:

