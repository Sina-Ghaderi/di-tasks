# Minimal GNOME Installation on Debian 13 (Trixie)
This is a method for installing a minimal and clean GNOME desktop on Debian 13 Stable that excludes unnecessary packages.  
It is suitable for systems with hardware limitations or for users who want a fully functional desktop without many additional software and packages.

### Background

In the Debian installer, the "Select and Install Software" section installs software (such as various desktops, additional software like OpenSSH, web servers, etc.) using the **tasksel** package.

In previous Debian installer versions, **tasksel** was installed before reaching the “Select and Install Software” step.  
This allowed editing the `/target/usr/share/tasksel/descs/debian-tasks.desc` file and creating a custom task for **gnome-core**, which could then be installed via the installer’s software selection menu.

However, in Debian 13, **tasksel** itself is installed during the “Select and Install Software” step, this means we can no longer modify its task list before it runs.

Additionally, in Debian 13, the **gnome-core** package is no longer as clean as it used to be.  
It’s essentially a meta-package (macro) that pulls in many dependencies, and in this release, even more packages have been added — which, personally, I still find excessive.

While **gnome-core** is much more minimal than the full **gnome** package that Debian’s installer selects in **tasksel** by default, it’s still far from minimal.  
For reference: installing GNOME via **tasksel** brings in a lot of extras such as LibreOffice, 10–15 games, and various other non-essential packages.  

Even **gnome-core** now includes additional software that may not be strictly necessary for a functional desktop.
Another limitation of installing via **tasksel** is that the `APT::Install-Recommends` option is always enabled, so all recommended packages get installed automatically.

---

### Our Approach

In this guide:

- We skip **tasksel** entirely and install packages directly with `apt` using the `--no-install-recommends` flag inside the target system.  
- We skip **gnome-core** as well, since it’s no longer minimal enough. Instead, we manually install only the essential packages required for a fully functional but ultra-minimal GNOME desktop.

---

### Step 1 — Start the Installer in Expert Mode

To follow this method, you **must** run the Debian installer in **Expert Mode**:

1. Boot the Debian installer.  
2. Go to **Advanced Options**.  
3. Select **Expert Mode** or **Expert Install**.

---

### Step 2 — Proceed Until “Select and Install Software”

Continue through the installer until you reach the **Select and Install Software** step, **Do not** select this option

Instead:
1. Choose **Execute a shell** from the installer menu.  
2. You will now have shell access to the installer environment.

---

### Step 3 — Replace Tasksel with Minimal GNOME Install

Inside the installer shell edit this file `/var/lib/dpkg/info/pkgsel.postinst`:

```bash
nano /var/lib/dpkg/info/pkgsel.postinst
```
Find this block (around line 161):

```bash
if db_get pkgsel/run_tasksel && [ "$RET" = true ]; then
    log "starting tasksel"
    db_progress INFO pkgsel/progress/tasksel
    apt-install tasksel  # ensure tasksel is installed
    DEBIAN_TASKS_ONLY=1 in-target sh -c "tasksel --new-install --debconf-apt-progress='--from $tasksel_start --to $tasksel_end --logstderr'" || aptfailed
fi
```
Comment out lines `apt-install tasksel ...` and `DEBIAN_TASKS_ONLY=1 in-target sh -c...` then add following lines below them:

```bash
pkg_list=$(wget -O - https://raw.githubusercontent.com/Sina-Ghaderi/di-tasks/refs/heads/master/gnome-clean.list) || aptfailed
in-target sh -c "debconf-apt-progress --from $tasksel_start --to $tasksel_end --logstderr -- apt-get -q -y install --no-install-recommends -- $pkg_list" || aptfailed
```
**Note:** if you want to install nvidia driver too use `gnome-clean-nvidia.list` instead of `gnome-clean.list`  
finally, entire block should be like this: 

```bash
if db_get pkgsel/run_tasksel && [ "$RET" = true ]; then
    log "starting tasksel"
    db_progress INFO pkgsel/progress/tasksel
    # apt-install tasksel  # ensure tasksel is installed
    # DEBIAN_TASKS_ONLY=1 in-target sh -c "tasksel --new-install --debconf-apt-progress='--from $tasksel_start --to $tasksel_end --logstderr'" || aptfailed

    # clean-gnome without nvidia driver, if you want nvidia driver to be installed too use gnome-clean-nvidia.list in link below
    pkg_list=$(wget -O - https://raw.githubusercontent.com/Sina-Ghaderi/di-tasks/refs/heads/master/gnome-clean.list) || aptfailed
    in-target sh -c "debconf-apt-progress --from $tasksel_start --to $tasksel_end --logstderr -- apt-get -q -y install --no-install-recommends -- $pkg_list" || aptfailed

fi
```

After making these changes, save the file (Ctrl+O), exit nano (Ctrl+X), then exit the shell. now, choose **Select and Install Software** from the installer menu and continue the installation.

Because of the modifications, the tasksel menu will no longer appear during the **Select and Install Software** step, and the minimal GNOME packages will be installed automatically.

### Step 4 — Finish Installation
After the installation completes, continue with the rest of the Debian installer steps as usual.

### Results
Tasksel + gnome-core installation: ~1100+ packages  
This method: ~750 packages

The resulting GNOME desktop is clean, lightweight, and fully functional.
However, since some optional dependencies and libraries are omitted, you may need to install additional packages later based on your needs.

### Package Lists
We provide two package lists for your convenience:

gnome-clean.list — Minimal GNOME without NVIDIA drivers
gnome-clean-nvidia.list — Minimal GNOME with NVIDIA driver to be installed

You can fork the repository and add your own packages to the lists so that your custom packages will be installed automatically during the Debian installation.
