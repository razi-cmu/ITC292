# ITC 292 - Final Projects

## Project 3: "The Overloaded Server"

You have been handed a Linux virtual machine that a previous system administrator left in a broken, poorly documented state. Your team's job is to act as the incoming sysadmin team: investigate the system, identify every issue, fix it, and document your process professionally, exactly as you would be expected to on the job.

### Importing the VM

1. You are provided with a `VM_P3.ova` file.
2. Download and unzip the assigned `.ova.zip` file.
3. Open **VirtualBox → File → Import Appliance**.
4. Select the `.ova` file, click **Next**, then **Import**.
5. Start the VM.
6. Immediately create a snapshot before making changes. For more information visit this [link](https://www.virtualbox.org/manual/ch01.html#snapshots).

### Working on the VM

Unlike some past incidents, login itself isn't broken here, so you can SSH in right from the start instead of typing everything into the VirtualBox console window.

1. Check whether a port-forwarding rule already exists: VM **Settings → Network → Adapter 1 → Advanced → Port Forwarding**. If there isn't one, add a rule with Host Port `2222` and Guest Port `22` (leave the IP fields blank).
2. From your host machine's terminal (PowerShell on Windows, Terminal on Mac/Linux):
   ```
   ssh -p 2222 sysadmin@127.0.0.1
   ```

### Symptom Report
The following notes were collected from the client and their team before they handed this system over to you. This is all the information you get. Each note points toward something worth investigating, but none of them tell you the cause or the fix. Treat this the way you would treat a client intake conversation on a real support ticket.

A client reported that "This server hasn't gone down, but something's badly wrong. It's sluggish, unreliable, and our team can't pin down why."

### Additional Field Notes

- Everyday commands and logins feel noticeably slower than they used to, even though nobody changed anything recently.
- An overnight monitoring alert flagged unusually high memory usage, and nobody on our team recalls installing anything new.
- The system does have swap space configured, but it doesn't seem to be helping at all when memory gets tight.
- One of our admins noticed a growing number of strange "defunct" processes piling up when they checked on the system.
- A service that's supposed to generate nightly reports keeps showing as unstable whenever we check its status, and the reports aren't showing up.
- Free disk space has been steadily dropping, and we haven't been able to figure out what's consuming it.
- We do have a health-check log file that's supposed to record system status every minute, but the entries in it look incomplete every time we check.

### Diagnostic Toolkit
You've covered the commands needed to investigate each of these areas across Weeks 1–7. The table below points you to a starting command for each symptom area. It will help you confirm and narrow down an issue, but it won't tell you the cause or the fix. Treat each as a starting point for further investigation, not a final answer.

| Symptom Area | Starting Command |
|---|---|
| CPU usage | `top` or `ps -eo pid,%cpu,cmd --sort=-%cpu \| head` |
| Memory usage | `free -h` |
| Swap configuration | `swapon --show` and `sysctl vm.swappiness` |
| Zombie / defunct processes | `ps aux \| grep Z` |
| Report service status | `systemctl status report-generator.service` and `journalctl -u report-generator.service` |
| Disk usage | `df -h` and `du -sh /var/log/*` |
| Scheduled health-check job | `crontab -l` and `cat /var/log/health-check.log` |

### Your Tasks

1. Diagnose: identify every issue preventing normal operation of the system. There is more than one.
2. Fix: resolve each issue so the system returns to full, normal working order.
3. Document: for each issue found, record what was wrong, how you found it, the command(s) used to confirm and fix it, and how you verified the fix worked.
4. Demonstrate: provide evidence (screenshots) showing the system's before-and-after state and how you solved it along with the team member who solved it.

## Confirming Your Fix

| Check | Expected Result |
|---|---|
| Is CPU usage back to normal? | No single process consuming near 100% of a core |
| Is memory usage back to normal? | `free -h` shows a typical idle usage level |
| Is swap actually usable again? | `sysctl vm.swappiness` shows a normal, non-zero value |
| Are there any zombie processes left? | `ps aux \| grep Z` shows none |
| Is the report service stable? | `systemctl status` shows the service settled, not repeatedly restarting |
| Is disk space back to normal? | `df -h` shows a healthy amount of free space |
| Is the health-check log complete? | New entries contain full output, not cut off partway through |

## Getting Started

- Start broad, then narrow your investigation.
- Fix one issue at a time.
- Use the diagnostic commands covered in class.
- Check running processes and resource usage before assuming a service or script is broken.
- Incident report can be in any format you like. Make sure the report is well formatted and easy to read. Add screenshots wherever needed.

---

