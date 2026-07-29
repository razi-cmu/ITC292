# ITC 292 - Final Projects

## Project 5: "The Departing Admin's Scavenger Hunt"

A previous system administrator, R. Kade, left the company abruptly and without a proper handoff. Before leaving, they moved a critical file `final_config_backup.tar.gz` somewhere on this system and left behind a scattered trail of notes, jobs, and logs that only make sense once you start pulling on the right thread. Nobody currently on staff knows where the file ended up.

Your team's job is to act as the incoming sysadmin team: pick up the trail cold, follow it step by step using the diagnostic skills you've built across this course, recover the file, and document your process professionally exactly as you would be expected to on a real support ticket.

Nothing needs to be repaired. You are investigating, not fixing until the very last step, where recovering the file itself does require a corrective action.

This project is worth **20%** of your final course grade.

### Importing the VM

1. You are provided with a `VM_P5.ova` file.
2. Download and unzip the assigned `.ova.zip` file.
3. In VirtualBox, go to **File → Import Appliance** and select the `.ova` file.
4. Confirm the imported VM has two network adapters configured (NAT and Host-only) before powering on. Do not change these settings.
5. Power on the VM.

### Working on the VM

Login is not blocked in this profile, so you can SSH in from the start rather than working through the VirtualBox console window.

1. Check whether a port-forwarding rule already exists: VM **Settings → Network → Adapter 1 → Advanced → Port Forwarding**. If there isn't one, add a rule with Host Port `2222` and Guest Port `22` (leave the IP fields blank).
2. From your host machine's terminal (PowerShell on Windows, Terminal on Mac/Linux):
   ```
   ssh -p 2222 sysadmin@127.0.0.1
   ```
3. Use your assigned account credentials to log in.

### Investigation Briefing

The following was passed along by IT leadership before they handed this system over to your team. This is all the information you get.

> "R. Kade didn't give us any notice, and didn't leave a handoff document. All we know is that there's a configuration backup on this box somewhere that we need back. Their account is still on the system, locked, but whatever they left behind should still be there if you know where to look. Good luck."

### Field Notes

These notes were gathered from various people who worked with Kade or who poked around the system after they left. None of them tell you the destination directly and each one only makes sense once you've found the step before it.

- Kade's old home directory is still on the system, though nobody's found anything obviously useful just by looking inside it.
- Automated jobs on this box haven't been touched in months, but a few of them still carry comments Kade left for themself.
- At least one log file on this system goes back further than anyone currently on staff expected.
- Not every filename on this system can be trusted to describe what's actually inside the file.
- Some notes on this system aren't readable by just anyone who happens to be logged in.
- A couple of old services are still enabled and still logging output on every boot, even though nobody remembers turning them on.
- Once you find the file itself, don't be surprised if it doesn't let you touch it right away.

### Diagnostic Toolkit

You've covered the commands needed to investigate each of these areas across Weeks 1–7. The table below points you to a starting command for each area of investigation. It will help you confirm what you're looking at, but it won't hand you the trail, so treat each as a starting point.

| Area | Starting Command |
|---|---|
| Hidden files in a directory | `ls -la` |
| System-wide scheduled jobs | `cat /etc/crontab` |
| Searching inside a file's contents | `grep "search-term" /path/to/file` |
| Verifying a file's real type | `file /path/to/file` |
| File ownership and permissions | `ls -l`, `id` |
| Service status and logs | `systemctl status <service>`, `journalctl -u <service>` |
| File attributes beyond permissions | `lsattr /path/to/file`, `chattr` |

`chattr`/`lsattr` haven't come up in this course before now. `lsattr` shows extended file attributes that exist alongside normal permissions; `chattr` changes them. An `i` attribute means a file is *immutable*, it can't be modified, renamed, or deleted by anyone, including root, until that attribute is explicitly removed. If you encounter a file that seems to reject even `sudo rm`, this is worth checking.

### Tasks

1. Follow the trail from Kade's home directory through to the final archive. Each step points to the next so work through them in order.
2. For every step in the trail, record: what you found, what command revealed it, and what it told you to look at next.
3. Once you locate the final archive, confirm its contents are intact before doing anything else to it.
4. Fully recover the archive: extract its contents, and restore the file to a normal, unlocked, properly-owned state so it could be handled like any other file going forward.
5. Verify the extracted contents are complete and uncorrupted.

### Rules

- No scripts. Every step must be a single command or interactive `nano` use.
- Do not skip ahead by guessing paths. If you find yourself typing a path you haven't actually been led to yet, back up and find the note that would have told you.
- Work only within your assigned VM. Do not attempt to access another team's VM.
- Document every command as you go as you will not remember the exact sequence by the time you write your report.
- If you get stuck at a step, re-read the note or output from the previous step carefully before asking for help. Every step contains what you need to move forward.

### Deliverables

1. **Written investigation report** for each step in the trail: what you found, the exact command(s) used to find and interpret it, and what it led you to next.
2. **Evidence log** — terminal output or screenshots supporting each step of the trail, including the final recovery.
3. **Recovery summary** — confirmation of the archive's contents, and a description of what you did to fully unlock and restore the file.
4. **Live demonstration** — your team walks through the trail on the VM, from Kade's home directory to the recovered file.

### Grading Rubric

| Category | Weight | What's Assessed |
|---|---|---|
| Diagnosis | 40% | Correctly interpreting each of the 7 steps in the trail and identifying what it points to next |
| Corrective Actions | 30% | Correctly performing the required actions at each gated step (permission/ownership access, accurate file-type identification, full unlock and recovery of the final archive) |
| Documentation | 10% | Clear, accurate written record of the investigation and recovery process |
| Demonstration | 20% | Live walkthrough of the trail and recovery, matching the written report |

### Confirming You've Solved It

Once you believe you've fully recovered the file, check the following before you consider the project complete:

| Check | Command | Expected Result |
|---|---|---|
| Archive contents are readable | `tar -tzf <path-to-archive>` | Lists file contents without error |
| Archive extracts cleanly | `tar -xzf <path-to-archive> -C <destination>` | Extraction completes with no errors |
| No hidden attribute remains | `lsattr <path-to-archive>` | No `i` flag present |
| Ownership is normal | `ls -l <path-to-archive>` | Owned by your working account, not `root` |
| Extracted files are complete | `ls -la <destination>` | All expected files present and non-empty |

If any of these checks fail, you haven't finished the recovery, go back and confirm you addressed both the permission barrier and the file attribute barrier on the final archive.
