# ITC 292 - Final Projects

## Project 4: "The Backup That Isn't There"

You have been handed a Linux virtual machine that a previous system administrator left in a broken, poorly documented state. Your team's job is to act as the incoming sysadmin team: investigate the system, identify every issue, fix it, and document your process professionally, exactly as you would be expected to on the job.

This project is worth **20%** of your final course grade.

### Importing the VM

1. You are provided with a `VM-P4.ova` file.
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

A client reported that "We think our backups have been failing for a while, and now we're noticing other things look off too; we just don't know where to start."

### Additional Field Notes

- Nightly backups were supposedly scheduled, but no one on our team can recall ever actually seeing one run on time.
- When someone manually kicked off a backup to test it, the result didn't seem to include the files it was supposed to.
- The backup status log claims "SUCCESS" every single time, even when other evidence suggests otherwise.
- There's an old backup file sitting in the backup folder, but nobody's been able to confirm whether it's actually usable.
- Attempts to write new backups to the backup folder have been failing quietly, with no clear error shown to the user running them.
- Free disk space has been steadily shrinking, and a log file that's supposed to rotate regularly doesn't seem to be doing so.
- Our security team mentioned the login/authentication log hasn't shown any new entries in some time, even though people have definitely been logging in and using `sudo`.

### Diagnostic Toolkit
You've covered the commands needed to investigate each of these areas across Weeks 1–7. The table below points you to a starting command for each symptom area. It will help you confirm and narrow down an issue, but it won't tell you the cause or the fix. Treat each as a starting point for further investigation, not a final answer.

| Symptom Area | Starting Command |
|---|---|
| Backup schedule | `systemctl list-timers --all` and `systemctl status backup.timer` |
| Backup destination contents | `sudo ls -la /mnt/backups` |
| Backup status log | `sudo cat /var/log/backup-status.log` |
| Running the backup manually | `sudo -u backupsvc /opt/backup-tool/run_backup.sh` |
| Disk usage | `df -h` and `du -sh /var/log/*` |
| Log rotation configuration | `sudo logrotate -d /etc/logrotate.d/appdata` |
| Authentication log | `sudo tail /var/log/auth.log` and `journalctl -u rsyslog` |

### Your Tasks

1. Diagnose: identify every issue preventing normal operation of the system. There is more than one.
2. Fix: resolve each issue so the system returns to full, normal working order.
3. Document: for each issue found, record what was wrong, how you found it, the command(s) used to confirm and fix it, and how you verified the fix worked.
4. Demonstrate: provide evidence (screenshots) showing the system's before-and-after state and how you solved it along with the team member who solved it.

## Rules and Constraints

- **Do not** use AI to generate your diagnosis/documentation.
- Do not restore snapshots to avoid troubleshooting.
- Work only within your assigned VM.
- All team members must contribute.

## Deliverables

- Incident report (symptom, root cause, fix, verification)
- Final working VM (upload on your OneDrive and share the link in Incident Report). Make sure instructor has access to download the VM.
- You'll be required to present the project. Schedule for presentation will be provided on Blackboard.

## Grading Rubric

| Component | Weight |
|---|---:|
| Correct diagnosis | 40% |
| Correct fixes | 30% |
| Documentation quality | 10% |
| Demonstration | 20% |

## Confirming Your Fix

| Check | Expected Result |
|---|---|
| Is the backup timer active? | `systemctl list-timers --all` shows `backup.timer` scheduled with a valid `NEXT` run time |
| Does a manual backup complete successfully? | The status log shows an honest `SUCCESS`, matching what actually happened |
| Does the backup contain the right data? | Listing the newest archive's contents shows the expected files, not an empty or missing archive |
| Is old, bad data cleared out? | No corrupted or clearly stale archives remain in the backup folder |
| Can the backup account write to its destination? | New backups are created without any permission errors |
| Is disk space back to normal? | `df -h` shows a healthy amount of free space |
| Is the authentication log recording new activity? | New entries appear in `/var/log/auth.log` after a fresh login or `sudo` command |

## Getting Started

- Start broad, then narrow your investigation.
- Fix one issue at a time.
- Use the diagnostic commands covered in class.
- Check logs and file timestamps carefully before assuming something is working correctly — a log that says "SUCCESS" isn't always telling the truth.
- Incident report can be in any format you like. Make sure the report is well formatted and easy to read. Add screenshots wherever needed.

---

**Good luck! Treat this as a real production incident on your first day as a system administrator.**
