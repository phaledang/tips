# Power Automate Hosted Machine Setup Guideline

## Problem

Setting up Power Automate Desktop to run **unattended flows on a hosted machine** involves many moving parts across multiple Microsoft admin portals (Intune, Entra ID, Power Platform). Misconfiguration at any step — missing licenses, wrong permissions, missing Entra app registrations, or forgetting to log off — can cause the hosted machine provisioning or unattended flow execution to fail.

This guide documents the **end-to-end solution** that was validated on a test environment.

---

## Solution Overview

The setup is broken into the following phases:

1. **[IT Admin]** Infrastructure — image creation, provisioning policies, and Entra ID app registration
2. **[IT Admin]** Licensing — buying and assigning the correct licenses
3. **[IT Admin]** Permissions — assigning admin and environment roles
4. **[Host Creator]** Machine creation — creating and connecting to the hosted machine
5. **[Host Creator]** Machine configuration — browser extensions, desktop flows, Edge profile
6. **[Host Creator]** Unattended flow — creating and testing a cloud flow that triggers a desktop flow in unattended mode

---

## Phase 1: Infrastructure Setup (IT Administrators)

### 1.1 Image creation and provision settings

**Problem:** Without provisioning policies, hosted machines may not join Entra ID or could violate security rules.

**Solution:** Set up provisioning policies to configure machines from the image.

1. Go to <https://intune.microsoft.com/>
2. Click **Device onboarding > Windows 365** to go to policy list
3. Click **Create Policy** to create a new one

![Image creation and provision settings](images/001_image_creation_and_provision_settings.png)

![Image creation and provision settings - policy](images/002_image_creation_and_provision_settings.jpeg)

### 1.2 Entra ID app registration

**Problem:** The Windows 365 service principal may not exist in your tenant, which prevents Cloud PC provisioning.

**Solution:** Verify the app exists and create it if necessary.

1. Go to <https://entra.microsoft.com/>
2. Click **Enterprise apps**
3. Filter application type = Microsoft application, search **Windows 365**

![Entra ID app registration](images/003_entra_id_app_registration.png)

If app does not exist, create the Windows 365 service principal using the [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/):

```bash
az ad sp create --id 0af06dc6-e4b5-4f28-818e-e78e62d137a5
```

Do the same for other required applications.

---

## Phase 2: Licensing (IT Administrators)

### 2.1 Assign license to the selected environment

**Problem:** Hosted machines cannot be provisioned without the correct capacity-based license assigned to the environment.

**Solution:** Assign the **Power Automate Hosted Process** license to the environment.

![Buy and assign license](images/005_buy_and_assign_license_to_the_selected_environment.png)

#### Licenses assigned to the selected environment

- Assign **Power Automate Hosted Process** license (capacity-based)
- Allocate hosted machine capacity in the **Power Platform Admin Center**

#### Host creator licenses

**Problem:** The host creator needs multiple prerequisite licenses or the hosted machine creation will fail.

**Solution:** Ensure the following licenses are assigned to the host creator:

- Windows 365 license
- Microsoft Intune license
- Microsoft Entra ID (Azure AD)

> **Tip:** All of the above are covered under **Microsoft 365 E5**. Additionally, the **Power Automate Premium** user plan or trial must be started for the host creator to run desktop flows.

From [Hosted machines - Power Automate | Microsoft Learn](https://learn.microsoft.com/en-us/power-automate/desktop-flows/hosted-machines).

![Host creator licenses](images/004_host_creator_licenses.png)

Screenshot of the creator's licence on Power Automate (some other trials are turned on but they are not required):

![Host creator licenses screenshot](images/006_host_creator_licenses.png)

---

## Phase 3: Permissions (IT Administrators)

### 3.1 Assign permissions

**Problem:** Without the correct roles, administrators cannot manage devices and the host creator cannot provision machines or run flows.

**Solution:** Assign the following roles:

#### IT Administrator permissions

- **Intune Administrator** (for device management)
- **Entra ID Administrator** (for Entra ID application management)

#### Host creator permissions

- At least **Environment Maker** on the selected environment (Power Platform)
- To verify the settings, the host creator is assigned as **System Administrator** on the Environment

![Host creator permission](images/007_host_creator_permission.png)

### 3.2 Exclude rules for administrator access

**Problem:** IT administrators may have policies that automatically revoke administrator permissions from the hosted machine's owner. This breaks unattended mode because the host creator can no longer control the machine.

**Solution:** Disable those rules and either:
- Add the machine owner to the **Administrators** group, or
- At minimum, add them to the **Remote Desktop Users** group to enable unattended mode

---

## Phase 4: Create and Connect to the Hosted Machine (Host Creator)

### 4.1 Create hosted machine

**Problem:** The hosted machine needs to be provisioned through the Power Automate portal with the correct environment and capacity.

**Solution:**

1. Navigate to **Power Automate portal**
2. Go to: **Monitor → Machines → Hosted Machines**
3. Create new hosted machine:
   - Select environment
   - Assign machine name
   - Allocate capacity
   - Wait for provisioning (Windows 365 backend)

### 4.2 Connect to the hosted machine

**Problem:** Need to verify the hosted machine is accessible and connection quality is acceptable.

**Solution:** Click **Open in browser** to connect to the hosted machine.

- It will ask for connection approval before login
- A new tab `https://windows.cloud.microsoft/webclient/ent/{machine_id}` is opened
- Click on the connection icon to check the status

![Successfully connect to the hosted machine](images/008_successfully_connect_to_the_hosted_machine.png)

![Connection status](images/009_successfully_connect_to_the_hosted_machine.png)

Connection quality details:

| Property | Value |
|---|---|
| Timestamp (UTC) | 2026-04-04T06:53:50.240Z |
| Activity ID | 77e0c0c8-93a5-40bc-94e9-cf37c0750100 |
| **Network** | |
| Transport protocol | TCP |
| Round-trip time | 57 ms |
| Available bandwidth | 0.512 Mbps |
| Frame rate | 32 FPS |
| **Remote computer** | |
| Gateway name | afdfp-rdgateway-r1.wvd.microsoft.com:443 |
| Gateway logon method | Azure Active Directory |

### 4.3 Verify the Intune device

**Problem:** Need to confirm the hosted machine is correctly registered as a managed device.

**Solution:**

1. Go to <https://windows.cloud.microsoft/#/devices>
2. The machine should appear in the device list
3. Click **…** to connect, view details, or check settings

![Intune device list](images/010_the_intune_device_is_added_to_the_host_creators_device.png)

![Intune device details](images/011_the_intune_device_is_added_to_the_host_creators_device.png)

![Intune device settings](images/012_the_intune_device_is_added_to_the_host_creators_device.png)

### 4.4 [IT Admin] Recheck device settings and status

**Problem:** The hosted machine may not be compliant or may be missing required policies.

**Solution:**

1. Go to Microsoft Intune Admin Center: <https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/Cloud%20PC>
2. Navigate: **Devices → Windows 365 → Cloud PCs**
3. Validate:
   - Hosted machine appears as Cloud PC
   - Device is compliant
   - Policies applied:
     - Remote Desktop enabled
     - Device configuration profile assigned

![Device status - recheck settings](images/013_to_recheck_settings_and_status_of_the_devices.jpeg)

![Device compliance](images/014_to_recheck_settings_and_status_of_the_devices.png)

![Device policies](images/015_to_recheck_settings_and_status_of_the_devices.png)

### 4.5 Check admin and remote desktop permission on the hosted machine

**Problem:** The host creator's admin permissions may have been revoked by IT policies. Without proper permissions, the user cannot run commands or trigger unattended flows.

**Solution:**

1. Only the owner can remotely access the machine in the browser
2. If admin permission was revoked: contact IT administrators, log in to the hosted machine and grant control so the host creator can run commands as administrator
3. Add other users to the Remote Desktop group if they also need to run desktop flows

```powershell
net localgroup "Remote Desktop Users" "azuread\the-account" /add
net localgroup administrators "azuread\the-account" /add
```

![Admin and remote desktop permission - local users](images/016_check_admin_and_remote_desktop_permission_on_the_power_automate_hosted_machine.png)

![Admin and remote desktop permission - groups](images/017_check_admin_and_remote_desktop_permission_on_the_power_automate_hosted_machine.png)

![Admin and remote desktop permission - verification](images/018_check_admin_and_remote_desktop_permission_on_the_power_automate_hosted_machine.png)

---

## Phase 5: Configure the Hosted Machine (Host Creator)

### 5.1 Enable Power Automate Desktop browser extensions

**Problem:** Desktop flows that automate browser actions will fail if the Power Automate Desktop extension is not enabled in Edge and Chrome.

**Solution:**

**Edge:** Open Edge → Settings → Extensions → Turn on the Power Automate extension

![Edge extension](images/019_check_if_power_automate_desktop_extension_is_turned_on_edge_and_chrome.png)

**Chrome:** Open Chrome → Settings → Extensions → Turn on the Power Automate extension

![Chrome extension](images/020_check_if_power_automate_desktop_extension_is_turned_on_edge_and_chrome.png)

### 5.2 Create a simple desktop flow

**Problem:** Need to verify that Power Automate Desktop works on the hosted machine before setting up unattended flows.

**Solution:**

1. Go to <https://make.powerautomate.com/>
2. Select the environment
3. Click **New flow > Desktop flow**
4. Power Automate Desktop should already be installed on the hosted machine. If running on a personal computer, click install to download it.

![Create desktop flow](images/021_on_the_hosted_machine_create_a_simple_power_automate_desktop_flow.png)

Sample desktop flow:

![Sample desktop flow](images/022_on_the_hosted_machine_create_a_simple_power_automate_desktop_flow.png)

### 5.3 Update Edge profile for automatic sign-in

**Problem:** Desktop flows that launch Edge fail with the error:

> Action 'Run_a_flow_built_with_Power_Automate_for_desktop' failed: Problem while executing action 'LaunchEdge'. **Failed to assume control of Microsoft Edge**

**Solution:** Update Edge profile settings to allow automatic sign-in:

![Update Edge profile](images/023_update_edge_profile_references_to_automatically_sign_in.png)

### 5.4 Always log off the hosted machine

**Problem:** Unattended mode will **fail** if a user is currently logged in to the machine.

**Solution:** Always run the `logoff` command after connecting to the hosted machine. Never leave a session open.

![Always logoff](images/024_always_logoff_the_hosted_machine.png)

---

## Phase 6: Run Unattended Desktop Flows

### 6.1 Create a test cloud flow

**Problem:** Need to verify end-to-end that a cloud flow can trigger a desktop flow on the hosted machine in unattended mode.

**Solution:**

1. Go to <https://make.powerautomate.com/>
2. Select the environment
3. Click **New flow** → select **Scheduled cloud flow**, enter the name
4. Update the schedule
5. Click **+** to add new action
6. Type "power" in the search box → click action **"Run a flow built with Power Automate"**
7. Create the connection — type username and password of the host creator user
8. Select the desktop flow created in previous step → select **Unattended** mode
9. Click **Save**
10. Click **Test** after the flow is saved successfully
11. Click **Back** to check the running history

![Create test flow - new flow](images/025_create_test_flow_to_test_the_connection.png)

![Create test flow - schedule](images/026_create_test_flow_to_test_the_connection.png)

![Create test flow - add action](images/027_create_test_flow_to_test_the_connection.png)

![Create test flow - search power automate](images/028_create_test_flow_to_test_the_connection.png)

![Create test flow - create connection](images/029_create_test_flow_to_test_the_connection.png)

![Create test flow - select desktop flow](images/030_create_test_flow_to_test_the_connection.png)

![Create test flow - unattended mode](images/031_create_test_flow_to_test_the_connection.png)

![Create test flow - save](images/032_create_test_flow_to_test_the_connection.png)

![Create test flow - test](images/033_create_test_flow_to_test_the_connection.png)

![Create test flow - test running](images/034_create_test_flow_to_test_the_connection.png)

![Create test flow - test result](images/035_create_test_flow_to_test_the_connection.png)

![Create test flow - run history](images/036_create_test_flow_to_test_the_connection.png)

![Create test flow - run details](images/037_create_test_flow_to_test_the_connection.png)

### 6.2 Check machine flow running status

**Problem:** Need to monitor whether the hosted machine is correctly processing flow runs.

**Solution:**

1. Click **Machines**
2. Select the hosted machine
3. Click one request history entry to see details

![Machine flow running status - machines list](images/038_check_machine_flow_running_status.png)

![Machine flow running status - history](images/039_check_machine_flow_running_status.png)

![Machine flow running status - details](images/040_check_machine_flow_running_status.png)

---

## Common Issues & Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Machine doesn't join Entra ID | Missing provisioning policy | Set up provisioning policies in Intune (Phase 1.1) |
| Cloud PC provisioning fails | Windows 365 service principal missing | Create via `az ad sp create` (Phase 1.2) |
| Cannot create hosted machine | Missing Power Automate Hosted Process license | Assign capacity-based license to environment (Phase 2.1) |
| Desktop flow won't run | Missing prerequisite licenses (Windows 365, Intune, Entra ID) | Assign Microsoft 365 E5 + Power Automate Premium trial (Phase 2.1) |
| Cannot provision or manage machines | Insufficient permissions | Assign Environment Maker + System Administrator (Phase 3.1) |
| Admin permissions revoked on hosted machine | IT policy auto-removes local admin | Exclude the machine and re-add user to Administrators group (Phase 3.2) |
| Browser automation fails | Power Automate Desktop extension not enabled | Enable extension in Edge and Chrome (Phase 5.1) |
| "Failed to assume control of Microsoft Edge" | Edge profile not configured for auto sign-in | Update Edge profile settings (Phase 5.3) |
| Unattended mode fails | User is still logged in to the hosted machine | Always run `logoff` after connecting (Phase 5.4) |
| Session creation error | Various | See [troubleshooting guide](https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/desktop-flows/troubleshoot-session-creation-errrors#sessioncreationerror) |

---

## Important Configuration Links

| Name | URL |
|---|---|
| My Sign-Ins \| Recent Activity | <https://mysignins.microsoft.com/> |
| Windows 365 Boot - Microsoft Intune admin center | <https://intune.microsoft.com/> |
| Intune roles - Microsoft Intune admin center | <https://intune.microsoft.com/> |
| Microsoft Remote Desktop - Microsoft Azure | <https://portal.azure.com.mcas.ms/> |
| Licenses - Microsoft Entra admin center | <https://entra.microsoft.com/#view/Microsoft_AAD_IAM/LicensesMenuBlade/~/LicenseUtilization> |
| Licenses - Microsoft 365 admin center | <https://admin.microsoft.com/Adminportal/Home?referrer=entra#/licenses> |
| Capacity \| Power Platform admin center | <https://admin.powerplatform.microsoft.com/resources/capacity> |
| Licenses \| Power Platform admin center | <https://admin.powerplatform.microsoft.com/billing/licenses/dataverse/overview> |
| Power Automate Usage Licenses | <https://admin.powerplatform.microsoft.com/billing/licenses/PowerAutomate/powerAutomateUsage> |
| Licenses - Microsoft 365 admin center (requests) | <https://admin.cloud.microsoft/?source=tcemail#/licenses/requestspage> |
| Troubleshoot session creation errors | <https://learn.microsoft.com/en-us/troubleshoot/power-platform/power-automate/desktop-flows/troubleshoot-session-creation-errrors#sessioncreationerror> |
| Windows Enterprise CPC Image | <https://portal.azure.com/#create/microsoftwindowsdesktop.windows-ent-cpcwin11-25h2-ent-cpc> |

> **Note:** To view licensing information, you need to be a tenant administrator, Power Platform administrator, or Dynamics 365 administrator.

---

## References

- [Hosted machines - Power Automate | Microsoft Learn](https://learn.microsoft.com/en-us/power-automate/desktop-flows/hosted-machines)
- [Run unattended desktop flows - Power Automate | Microsoft Learn](https://learn.microsoft.com/en-us/power-automate/desktop-flows/run-unattended-desktop-flows)
- [Windows 365 Cloud PC image template](https://marketplace.microsoft.com/en-us/product/microsoftwindowsdesktop.windows-ent-cpc)
- [Hosted machine groups - Power Automate | Microsoft Learn](https://learn.microsoft.com/en-us/power-automate/desktop-flows/hosted-machine-groups#licensing-requirements)
- [Start a Free Trial | Microsoft Dynamics 365](https://www.microsoft.com/en/dynamics-365/free-trial)
- [Power Platform—Free Trials | Microsoft Power Platform](https://www.microsoft.com/en-us/power-platform/try-free)
- [Introduction to desktop flows - Power Automate | Microsoft Learn](https://learn.microsoft.com/en-us/power-automate/desktop-flows/introduction)
- [Share/export a desktop flow - Power Automate | Microsoft Learn](https://learn.microsoft.com/en-us/power-automate/desktop-flows/how-to/share-export-desktop-flows)
- [MicrosoftDocs/power-automate-docs: Hosted Machines documentation](https://github.com/MicrosoftDocs/power-automate-docs/blob/main/articles/desktop-flows/hosted-machines.md)
- [Power Automate Pricing | Microsoft Power Platform](https://www.microsoft.com/en/power-platform/products/power-automate/pricing)
