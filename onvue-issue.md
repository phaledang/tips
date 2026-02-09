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

## 🛠 Step 1 – Shutdown wsl, disable Windows Virtualization Features, disable dev drive

I opened **Command Prompt as Administrator** and ran:
```
wsl --shutdown
DISM /Online /Disable-Feature /FeatureName:VirtualMachinePlatform
dism /online /disable-feature /featurename:Microsoft-Hyper-V-All
fsutil devdrv disable

```
Alternative: using "turn Windows features on or off"

<img width="326" height="201" alt="image" src="https://github.com/user-attachments/assets/b1e63cef-00fd-4c50-8203-65ef4b19c110" />


After disabling these features, I **restarted the computer**.

> Restarting after turning off Windows features is important.

### Verify dev drive and ubuntu (not 100% certain this was required)
- No Ubuntu distribution visible
- No mounted WSL filesystem
- No Dev Drive attached (click detach if needed)

<img width="158" height="83" alt="image" src="https://github.com/user-attachments/assets/45d77de8-610e-43a1-9285-566183976938" />

---

## 🔎 Step 2 – Check Task Manager (Very Important)

After rebooting, I opened **Task Manager**.

Even though WSL wasn’t in Startup apps, I still saw related background processes.

That means something was triggering it automatically.

So I:

- Looked for any WSL-related processes  
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
4. Disabled extensions:
   - Container Tools
   - Dev Containers
   - Renote Development
   - Docker
   - WSL  
<img width="725" height="233" alt="image" src="https://github.com/user-attachments/assets/63f1de50-7dec-4b24-b807-e4e9056bb7f8" />
<img width="706" height="323" alt="image" src="https://github.com/user-attachments/assets/7c6e142a-e6be-4128-94f6-3b4c3d0671d7" />
<img width="751" height="457" alt="image" src="https://github.com/user-attachments/assets/4fe640a3-baed-4824-9564-11062e32276f" />

5. Restarted the computer

---

## Run the OnVUE system test again.

If it mentions that a specific process is still running (for example, a browser, background service, or unknown app):

- Open Task Manager
- Locate the exact process name shown in the OnVUE warning
- Click End Task
- Retry the system check

OnVUE may detect:

- Background browser processes
- Hidden services
- Dev-related tooling
- Container or virtualization services

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
1. Disable Windows virtualization features and restart the computer
2. Disable VS Code Docker/WSL/Container extensions and restart the computer 
3. Open Task Manager and end:
   - WSL processes  
   - Edge background processes 
4. Run the OnVUE system test. If it mentions that a specific process is still running, use task manager to end it 



