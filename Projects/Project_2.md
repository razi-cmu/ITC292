# ITC 292 - Final Projects

## Project 2: "The Automation Breakdown"

You have been handed a Linux virtual machine that a previous system administrator left in a broken, poorly documented state. Your team's job is to act as the incoming sysadmin team: investigate the system, identify every issue, fix it, and document your process professionally, exactly as you would be expected to on the job.

### Importing the VM

1. You are provided with a `VM-P2.ova` file.
2. Download and unzip the assigned `.ova.zip` file.
3. Open **VirtualBox → File → Import Appliance**.
4. Select the `.ova` file, click **Next**, then **Import**.
5. Start the VM.
6. Immediately create a snapshot before making changes. For more information visit this [link](https://www.virtualbox.org/manual/ch01.html#snapshots).

>**Hint**: If normal login fails: Restart the VM. Open the **GRUB** menu (`Shift` or `Esc` during boot). Choose **Advanced options**. Select **Recovery mode**. Drop to a **root shell**. If needed, remount the filesystem as read-write before editing files.

### Working on the VM

You don't have to do everything by typing directly into the VirtualBox console window. Once you've restored basic login access, you can SSH into the VM from your own terminal instead. This makes it much easier to copy-paste commands and edit files.

1. Check whether a port-forwarding rule already exists: VM **Settings → Network → Adapter 1 → Advanced → Port Forwarding**. If there isn't one, add a rule with Host Port `2222` and Guest Port `22` (leave the IP fields blank).
2. From your host machine's terminal (PowerShell on Windows, Terminal on Mac/Linux):
   ```
   ssh -p 2222 sysadmin@127.0.0.1
   ```
3. Note that SSH will only work once the account can log in normally. If login itself is one of the issues you find, you'll need to resolve that first via the console/recovery mode before SSH becomes available.

### Symptom Report
The following notes were collected from the client and their team before they handed this system over to you. This is all the information you get. Each note points toward something worth investigating, but none of them tell you the cause or the fix. Treat this the way you would treat a client intake conversation on a real support ticket.

A client reported that "Nobody can log into this server through their usual account anymore, whether from the console or over SSH."

### Additional Field Notes

- Even though the nightly cleanup job appears to run on schedule, junk files never actually seem to get cleaned up, week after week.
- When one of our admins finally got hold of the script file, it looked normal at a glance, and nothing seemed obviously wrong with it.
- One of our team members tried running the script by hand from the terminal, and nothing appeared to happen.
- A separate maintenance log that this job is supposed to update hasn't received a single new entry in a long time.
- Someone attempted to look inside the automation folder directly and got a permission denied error, even after they were able to log in.
- Our monitoring system has started sending disk space warnings that weren't there before.

### Diagnostic Toolkit
You've covered the commands needed to investigate each of these areas across Weeks 1–7. The table below points you to a starting command for each symptom area. It will help you confirm and narrow down an issue, but it won't tell you the cause or the fix. Treat each as a starting point for further investigation, not a final answer.

| Symptom Area | Starting Command |
|---|---|
| Login/shell status | `getent passwd sysadmin` |
| Scheduled job configuration | `sudo crontab -l` |
| Script file permissions | `ls -la /opt/scripts/system-maintenance.sh` |
| Script logic and behavior | `cat /opt/scripts/system-maintenance.sh` (then try running it manually) |
| Log directory access | `ls -la /var/log/maintenance` |
| Script directory permissions | `ls -la /opt/scripts` and `groups sysadmin` |
| Disk usage | `df -h` |

### Your Tasks

1. Diagnose: identify every issue preventing normal operation of the system. There is more than one.
2. Fix: resolve each issue so the system returns to full, normal working order.
3. Document: for each issue found, record what was wrong, how you found it, the command(s) used to confirm and fix it, and how you verified the fix worked
4. Demonstrate: provide evidence (screenshots) showing the system's before-and-after state and how you solved it along with the team member who solved it.

## Confirming Your Fix

| Check | Expected Result |
|---|---|
| Can you log in normally, both on the console and remotely? | Login succeeds without needing recovery mode |
| Does the maintenance script actually perform its task when run? | The target directory is actually cleared, not just a clean exit code |
| Does the script have execute permission? | `ls -la` shows the execute bit set |
| Does the scheduled job point to the correct script location? | The scheduled job entry matches the script's actual file path |
| Can the primary user access the automation directory normally? | No permission denied errors browsing or reading the script |
| Does the script successfully write to its log file? | New entries appear in the log after running it |
| Is disk space back to a normal level? | `df -h` shows reasonable free space |

## Getting Started 

- Start broad, then narrow your investigation.
- Fix one issue at a time.
- Use the diagnostic commands covered in class.
- Check permissions and file state before assuming a tool or a script is broken.
- Incident report can be in any format you like. Make sure the report is well formatted and easy to read. Add screenshots wherever needed.

---

