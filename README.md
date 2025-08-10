

# Debian Minimal GNOME Desktop

A simple way to install a clean, minimal GNOME desktop on Debian without extra software like LibreOffice or games, This method works by editing an installer script and adding a custom tasksel description before tasksel runs.


## For Newer Debian Versions

In newer Debian releases (such as Debian 13  Trixie), tasksel is installed during the "Select and install software" step of the installer itself.  
This means the package `tasksel` is not available before this step, so you cannot edit the tasksel description files prior to that.  

In older Debian versions, the installer installed the `tasksel` package before the "Select and install software" step,  
which allowed users to open a shell and manually edit the `/usr/share/tasksel/descs/debian-tasks.desc` file to customize available tasks.

This guide adapts to the new installer behavior by injecting the custom tasksel descriptions **right before** tasksel runs during the "Select and install software" step, ensuring minimal GNOME desktop options appear correctly in tasksel tasks.


## Debian Installer ISO Download

For netboot install, use this link (amd64 GTK netboot installer):  
[https://d-i.debian.org/daily-images/amd64/daily/netboot/gtk/](https://d-i.debian.org/daily-images/amd64/daily/netboot/gtk/)


## Important: Use Expert Install Mode

Before starting, boot the installer in Expert Install mode:  
At the boot menu, choose **Advanced Options** → **Expert Install**, This lets you open a shell and edit installer scripts during installation.

## Installation Steps

1. Proceed normally until just before **Select and install software**. (don't select it)

2. From the installer menu, choose: **Execute a shell**

3. Edit the file:  
```bash
nano /var/lib/dpkg/info/pkgsel.postinst
```

Around lines **161-162**, find this block:

```bash
if db_get pkgsel/run_tasksel && [ "$RET" = true ]; then
    log "starting tasksel"
    db_progress INFO pkgsel/progress/tasksel
    apt-install tasksel  # ensure tasksel is installed
    DEBIAN_TASKS_ONLY=1 in-target sh -c "tasksel --new-install --debconf-apt-progress='--from $tasksel_start --to $tasksel_end --logstderr'" || aptfailed
fi

```

4.  Between `apt-install tasksel  # ensure tasksel is installed` and the next line, add one of these commands:
-   For minimal GNOME desktop only:
   
  ```bash
  wget -O - https://raw.githubusercontent.com/Sina-Ghaderi/clean-de/refs/heads/master/gome-clean.desc >> /target/usr/share/tasksel/descs/debian-tasks.desc || aptfailed 
  ```      
-   For GNOME + NVIDIA driver:    
  ```bash
  wget -O - https://raw.githubusercontent.com/Sina-Ghaderi/clean-de/refs/heads/master/gome-clean-nvidia.desc >> /target/usr/share/tasksel/descs/debian-tasks.desc || aptfailed
  ```

Code block should be like this:

```bash
if db_get pkgsel/run_tasksel && [ "$RET" = true ]; then
    log "starting tasksel"
    db_progress INFO pkgsel/progress/tasksel
    apt-install tasksel  # ensure tasksel is installed

    wget -O - https://raw.githubusercontent.com/Sina-Ghaderi/clean-de/refs/heads/master/gome-clean.desc >> /target/usr/share/tasksel/descs/debian-tasks.desc || aptfailed

    DEBIAN_TASKS_ONLY=1 in-target sh -c "tasksel --new-install --debconf-apt-progress='--from $tasksel_start --to $tasksel_end --logstderr'" || aptfailed
fi

```
        
5.  Save (`CTRL+s`) and exit (`CTRL+x`) the editor and exit the shell with `exit` and return to the installer menu.
    
6.  Now choose **Select and install software**.  
    You should now see new options like `GNOME Core Desktop` or `GNOME Core Desktop and NVIDIA Driver` in tasksel menu
        
7.  Select your desired option and continue installation.

## License

MIT License
