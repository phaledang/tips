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
- Hyper-V 

And that’s likely where the problem started.
If virtualization-related services are running in the background, it can fail the pre-check.

---
Note: I’m not 100% sure every step was required, but I’m documenting everything that I did.

## 🛠 Step 1 – Disable Windows Virtualization Features

I opened **Command Prompt as Administrator** and ran:
```
wsl --shutdown
DISM /Online /Disable-Feature /FeatureName:VirtualMachinePlatform
dism /online /disable-feature /featurename:Microsoft-Hyper-V-All
fsutil devdrv disable

```

After disabling these features, I **restarted the computer**.

> Restarting after turning off Windows features is important.

---

## 🔎 Step 2 – Check Task Manager (Very Important)

After rebooting, I opened **Task Manager**.

Even though WSL wasn’t in Startup apps, I still saw related background processes.

That means something was triggering it automatically.

So I:

- Looked for any WSL-related processes  
- Looked for Docker services  
- Ended those tasks manually  
---

## 🧩 Step 3 – Investigate Visual Studio Code Extensions

My suspicion: a VS Code extension was starting WSL or container services in the background.

So I:

1. Opened **Visual Studio Code**
2. Went to **Extensions**
3. Searched for:
   - `docker`
   - `container`
   - `wsl`
4. Disabled extensions like:
   - Remote – WSL  
   - Dev Containers  
   - Docker  

I **did restart after disabling the Windows features**, but I honestly don’t remember whether I restarted again after disabling the VS Code extensions.

---

## 🌐 Bonus Issue – Edge Appearing as “Still Open”

Another strange behavior:

OnVUE sometimes told us to close Microsoft Edge. 
But Edge was already closed.

Solution:

1. Open Task Manager  
2. Find all **Microsoft Edge** processes  
3. Click **End Task**  

Sometimes Edge runs background processes even when no window is open.

After force-ending it, the pre-check moved forward.

---

## ✅ The Final Sequence I Recommend

If you want a clean and stress-free setup before your exam, do this:

1. Disable Windows virtualization features  
2. Restart the computer  
3. Open Task Manager and end:
   - WSL processes  
   - Docker services  
   - Edge background processes  
4. Disable VS Code Docker/WSL/Container extensions  
5. Restart again (recommended)  
6. Run the OnVUE system test  

---

## 🧠 Why This Happens

Tools like:

- WSL2  
- Docker Desktop  
- Hyper-V  
- Dev Containers  

Use virtualization under the hood.

Even if you are not “running a virtual machine,” Windows may still:

- Keep virtualization services active  
- Automatically start WSL  
- Enable background container services 

And OnVUE detects that.

---

## The key takeaway:
Background services can cause OnVUE to fail.
So before your exam:
- Disable virtualization features
- Disable VS Code Docker/WSL/Container extensions  
- Restart  
- Double-check Task Manager  




