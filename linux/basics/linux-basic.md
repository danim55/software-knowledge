## Linux Basics Documentation

This directory (`linux/basics`) contains foundational topics on Linux system administration. Each topic is organized into its own Markdown file.

### Sections

1. **Filesystem Management** (`fstab.md`)
   - 1.1 `/etc/fstab` – the filesystem table  
   - 1.2 Mounting filesystems  
   - 1.3 Network File System (NFS)  
   - 1.4 Putting it all together

2. **User and Group Management** (`users.md`)
   - 2.1 User accounts (`useradd`, `usermod`, `userdel`)  
   - 2.2 Group management (`groupadd`, `groupmod`, `groupdel`)

3. **Package Management** (`packages.md`)
   - 3.1 APT/YUM basics  
   - 3.2 Installing and removing packages

---

## 1. `/etc/fstab` – the filesystem table

**What it is**  
A plain-text file that tells Linux which filesystems to mount at boot (or on demand). Each line has six fields:

1. **Device or remote export** (e.g. `10.11.12.20:/path`)  
2. **Mount point** (local directory, e.g. `/mnt/nfs_logs`)  
3. **Filesystem type** (here, `nfs`)  
4. **Mount options** (comma-separated flags, e.g. `defaults,proto=tcp,port=2049`)  
5. **Dump** (usually `0`)  
6. **fsck order** (usually `0` for network filesystems)

**Why it matters**  
When you run `mount -a` or on each boot, Linux reads `/etc/fstab` and automatically mounts everything it finds there.

---

## 2. Mounting – attaching a filesystem

**What it means**  
The `mount` command tells the kernel: “Take that filesystem (local disk, CD, or—here—remote NFS export) and make it available under this directory tree.”

**The mount point**  
A regular directory (empty, or containing a previous mount) where the filesystem’s contents will appear. You create it with:

```bash
mkdir -p /path/to/mountpoint
````

---

## 3. NFS – Network File System

**What NFS is**
A protocol (and set of kernel drivers) that lets one machine share directories over the network, and lets clients mount them as if they were local disks.

**How it works under the hood**

1. **RPC (Remote Procedure Call):** NFS uses RPC (typically via `rpcbind`) to negotiate service ports.
2. **TCP/UDP on port 2049:** Actual file operations—`read`, `write`, `lookup`—happen over TCP or UDP (here forced via `proto=tcp,port=2049`).
3. **Kernel module:** The Linux NFS client module translates local `open()`, `read()`, `write()` calls into network requests.

---

## 4. Putting it all together

1. **Create the mount directory**

   ```bash
   mkdir -p /mnt/nfs_logs
   ```

2. **Add the share to `/etc/fstab`**

   ```fstab
   10.11.12.20:/home/.../log  /mnt/nfs_logs  nfs  defaults,proto=tcp,port=2049  0 0
   ```

3. **Mount all entries**

   ```bash
   mount -a
   ```

   After this, `/mnt/nfs_logs` behaves like a local folder—its contents are served live from the NFS server.

---

### Why this matters

* **Automation & consistency:** New VMs automatically mount the same log share without manual steps.
* **Performance & reliability:** Matching `defaults,proto=tcp,port=2049` ensures consistent transport settings.
* **Seamless tooling:** Standard Linux tools (`ls`, `cp`, `tar`, etc.) operate on remote files as if they were local.

---

**Essence**

* **`fstab`** declares what to mount where.
* **`mount`** performs the action.
* **NFS** provides a networked filesystem interface for remote directories.

