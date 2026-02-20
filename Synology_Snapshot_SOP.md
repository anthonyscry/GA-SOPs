# STANDARD OPERATING PROCEDURE

## Synology Snapshot Replication for File Versioning

| Field | Value |
|-------|-------|
| **Device** | Synology RS3621xs+ (ioSafe) |
| **Purpose** | Enable Windows Previous Versions functionality for shared files |
| **Date** | February 20, 2026 |

---

# PART 1: ADMINISTRATOR SETUP PROCEDURE

## 1. Prerequisites

### 1.1 Required Software/Packages

| Package | Source | Notes |
|---------|--------|-------|
| **Replication Service** | Synology Package Center | Dependency - install first |
| **Snapshot Replication** | Synology Package Center | Core package - required |
| **Btrfs File System** | Volume configuration | Volume must be Btrfs (not ext4) |

### 1.2 Dependencies

- DSM 6.0 or later (DSM 7.x recommended)
- Storage volume formatted as Btrfs
- SMB service enabled
- Shared folder created on Btrfs volume

> **NOTE:** When updating DSM or firmware, some older packages (including Snapshot Replication and Replication Service) may become incompatible with the new version. Always verify package compatibility before upgrading, and update packages as needed after DSM/firmware updates. Test snapshot functionality in a non-production environment when possible.

---

## 2. Installation

### 2.1 Download Packages (from internet-connected PC)

1. Go to https://www.synology.com/support/download
2. Select product: RS3621xs+
3. Download the following .spk files: Replication Service, Snapshot Replication
4. Transfer .spk files to a USB drive or accessible network location

### 2.2 Install Packages (Manual Install)

1. Log into DSM as administrator
2. Open Package Center
3. Click Manual Install (upper right corner)
4. Click Browse and select the Replication Service .spk file
5. Click Next and accept any terms
6. Wait for installation to complete
7. Repeat steps 3-6 for Snapshot Replication .spk file

### 2.3 Verify Btrfs Volume

1. Open Storage Manager
2. Click on Volume in the left panel
3. Verify the volume shows "Btrfs" as the file system

> **NOTE:** If volume is ext4, snapshots will not be available. Volume must be recreated as Btrfs.

### 2.4 Create Shared Folder on Btrfs Volume

1. Go to Control Panel → Shared Folder
2. Click Create → Create Shared Folder
3. Enter folder name
4. Select the Btrfs volume from the Location dropdown
5. Optional: Add description
6. Click Next
7. Configure encryption if required (or skip)
8. Click Next
9. On Advanced Settings, optionally enable data checksum for integrity
10. Click Next
11. Set folder permissions for users/groups
12. Click Apply to create the shared folder

---

## 3. SMB Configuration

### 3.1 Enable SMB Previous Versions Access

1. Go to Control Panel → File Services → SMB
2. Ensure "Enable SMB service" is checked
3. Verify "Disallow access to Previous Versions" is **UNCHECKED**
4. Click Apply

### 3.2 Advanced SMB Settings (Optional)

1. Click Advanced Settings
2. Recommended settings:
   - Maximum SMB protocol: SMB3
   - Minimum SMB protocol: SMB2
   - Enable Opportunistic Locking: Checked
3. Click Save

---

## 4. Snapshot Configuration

### 4.1 Enable Snapshots on Shared Folder

1. Open Snapshot Replication application
2. Click Snapshots in the left panel
3. Select the target shared folder
4. Click Settings

### 4.2 Configure Snapshot Schedule

1. Check "Enable snapshot schedule"
2. Set schedule based on requirements (see table below)
3. Configure retention policy (recommend: keep latest 500 snapshots)
4. Click OK to save

| Interval | Retention | Storage Impact |
|----------|-----------|----------------|
| Every 15 minutes | ~5 days | Higher |
| Every 30 minutes | ~7 days | Moderate |
| Every 1 hour | ~14 days | Lower |

### 4.3 Enable Visible Snapshot Folder

1. In Snapshot Replication → Snapshots → select folder
2. Click Settings
3. Check "Make snapshot visible"
4. Click OK

> **NOTE:** This creates a #snapshot folder accessible via `\\server\share\#snapshot\`

### 4.4 Take Initial Snapshot

1. Select the shared folder
2. Click Snapshot → Take Snapshot
3. Verify snapshot appears in the list with timestamp

---

## 5. Permissions Configuration

### 5.1 Shared Folder Permissions

1. Go to Control Panel → Shared Folder
2. Select the folder → Edit
3. Go to Permissions tab
4. Set permissions for user groups as appropriate for your organization

### 5.2 Snapshot Access Permissions

By default, users can only access Previous Versions of files they have permission to read. No additional snapshot-specific permissions are required.

> **NOTE:** Users cannot delete snapshots - only administrators can manage snapshot retention through DSM.

---

## 6. Verification

1. From a Windows client, map the shared folder
2. Right-click any file → Properties → Previous Versions
3. Verify snapshots appear in the list
4. Test browsing to `\\[server]\[share]\#snapshot\`
5. Verify timestamped folders are visible

---

# PART 2: END USER PROCEDURE

## 1. Overview

The file server now supports Previous Versions, allowing you to recover earlier versions of files that have been modified or accidentally overwritten. This works automatically - snapshots are taken every 15-30 minutes.

---

## 2. Method 1: Right-Click Previous Versions

*Use this method to quickly restore a specific file.*

1. Navigate to the file in Windows Explorer
2. Right-click the file
3. Select Properties
4. Click the Previous Versions tab
5. Select a version from the list (sorted by date/time)
6. Choose an action:
   - **Open** - View the old version (recommended)
   - **Restore** - Replace current file with selected version

> **NOTE:** "Restore To..." button is not functional with Synology. Use the workaround below.

### 2.1 Workaround: Save Previous Version to Different Location

1. Select the desired version and click Open
2. The file opens in the associated application
3. Click File → Save As
4. Save to your desired location with a new name if needed

*Alternative: Use Method 2 (#snapshot folder) to copy files directly.*

---

## 3. Method 2: Browse #snapshot Folder

*Use this method to browse multiple files or copy entire folders.*

1. Open Windows Explorer
2. Navigate to the shared folder
3. In the address bar, add `\#snapshot` to the path: `\\[server]\[share]\#snapshot\`
4. You will see folders named by timestamp (e.g., GMT-2025.01.09-14.30.00)
5. Open the desired timestamp folder
6. Browse to your files and copy what you need

> **NOTE:** The #snapshot folder is read-only. Copy files out to edit them.

---

## 4. Common Scenarios

### 4.1 "I saved over my file and need the old version"

1. Right-click the file → Properties → Previous Versions
2. Select a version from before your save
3. Click Open - file opens in associated application
4. Click File → Save As and save to your desktop or another location
5. Compare versions and keep what you need

### 4.2 "I deleted a file and need it back"

1. Navigate to the folder where the file was located
2. Right-click the folder → Properties → Previous Versions
3. Select a version from before deletion
4. Click Open
5. Find your file and copy it to the current folder

### 4.3 "I need to compare multiple versions"

1. Browse to `\\[server]\[share]\#snapshot\`
2. Open multiple timestamp folders
3. Copy the different versions to a temp folder
4. Rename them (e.g., file_v1.ext, file_v2.ext) for comparison

---

## 5. Important Notes

- "Restore To..." button does not work - use Open → Save As instead
- Snapshots are taken every 15-30 minutes - very recent changes may not be captured yet
- Snapshots are kept for approximately 5-7 days depending on configuration
- You cannot edit files directly in the #snapshot folder - copy them first
- If Previous Versions shows empty, wait for the next snapshot cycle
- Contact IT if you need a file older than the available snapshots

---

## 6. Quick Reference

| Task | Action |
|------|--------|
| Restore single file | Right-click → Properties → Previous Versions → Open → Save As |
| Copy from snapshot | `\\[server]\[share]\#snapshot\` → Copy file out |
| Recover deleted file | Right-click parent folder → Previous Versions → Open → Copy file |
| Get help | Contact IT / Submit ticket |
