# Linux: Interview Study Guide

All 50 Linux topics from the study list: what each command does, the flags that matter, and a typical example.

## Part 1: Navigation and File Basics

### 1. ls: List files and directories

Shows directory contents. Key flags: `-l` (long format with permissions, owner, size, date), `-a` (include hidden dotfiles), `-h` (human-readable sizes), `-t` (sort by time), `-R` (recursive). Example: `ls -lah`.

**Examples**

- `ls`: lists the names of files and folders in the current directory.
- `ls -l`: long format, one item per line with permissions, owner, size and date, e.g. `-rw-r--r-- 1 sam sam 220 Jan 5 notes.txt`.
- `ls -a`: also shows hidden files such as `.bashrc` and `.git`.
- `ls -lah /var/log`: lists a different folder, hidden files included, with sizes like `4.0K` or `1.2M`.
- `ls -lt | head -5`: the five most recently modified files.

### 2. cd: Change directories

Moves you to another directory. `cd ~` or bare `cd` goes home, `cd -` returns to the previous directory, `cd ..` goes up one level. `cd` is a shell builtin, not a separate program.

**Examples**

- `cd /var/log`: jump to an absolute path (starts with `/`).
- `cd projects`: go into a folder relative to where you are now.
- `cd ..`: go up one level; `cd ../..` goes up two.
- `cd ~` or just `cd`: go to your home directory.
- `cd -`: switch back to the directory you were in before.

### 3. pwd: Print the current directory

Prints the absolute path of your working directory. Useful in scripts and when you are lost in a deep tree. `pwd -P` resolves symlinks to the physical path.

**Examples**

- `pwd`: prints where you are, e.g. `/home/sam/projects`.
- `cd .. && pwd`: move up a level, then confirm where you landed.
- `pwd -P`: if `/home/sam/app` is a symlink to `/srv/app`, this prints the real path `/srv/app`.

### 4. mkdir: Create directories

`mkdir dir` makes one directory; `-p` creates parent directories as needed and does not error if the directory exists: `mkdir -p app/src/utils`.

**Examples**

- `mkdir photos`: creates one folder called `photos`.
- `mkdir docs images videos`: creates three folders at once.
- `mkdir -p app/src/utils`: creates `app`, `src` and `utils` in one go. Without `-p` it fails with "No such file or directory" if `app/src` does not exist.
- `mkdir -p logs`: safe to run repeatedly; no error if `logs` already exists.

### 5. touch: Create files or update timestamps

Creates an empty file if it does not exist; otherwise updates its access and modification times. Often used to create placeholder files or to trigger build tools that watch timestamps.

**Examples**

- `touch notes.txt`: creates an empty file named `notes.txt`.
- `touch a.txt b.txt c.txt`: creates three files at once.
- `touch notes.txt` (again): the file already exists, so only its modified time changes; check with `ls -l`.
- `touch -d "2024-01-01" old.txt`: sets a specific timestamp, handy for testing.

### 6. cp: Copy files and directories

`cp src dest`. Use `-r` for directories, `-p` to preserve mode and timestamps, `-a` (archive) to preserve everything, `-i` to prompt before overwriting.

**Examples**

- `cp report.txt backup.txt`: copies a file under a new name.
- `cp report.txt ~/Documents/`: copies into another folder, keeping the name.
- `cp -r project project_backup`: copies a whole folder (`-r` is required for directories).
- `cp -i a.txt b.txt`: asks "overwrite b.txt?" before replacing an existing file.
- `cp *.jpg photos/`: copies every `.jpg` in the current folder into `photos/`.

### 7. mv: Move or rename files

The same command moves and renames: `mv old.txt new.txt` renames, `mv file dir/` moves. Within one filesystem it is instant because only the directory entry changes. `-i` prompts before overwrite, `-n` never overwrites.

**Examples**

- `mv old.txt new.txt`: renames a file.
- `mv report.txt ~/Documents/`: moves a file into another folder.
- `mv report.txt ~/Documents/final.txt`: moves and renames in one step.
- `mv *.log logs/`: moves every `.log` file into `logs/`.
- `mv -i a.txt b.txt`: asks before overwriting `b.txt`.

### 8. rm: Remove files and directories

Deletes permanently (no recycle bin). `-r` removes directories recursively, `-f` forces without prompts, `-i` asks first. `rm -rf /` style mistakes are infamous; double-check paths and variables.

**Examples** (there is no undo, so check before you press Enter)

- `rm temp.txt`: deletes one file.
- `rm -i temp.txt`: asks "remove regular file 'temp.txt'?" first.
- `rm *.tmp`: deletes every `.tmp` file in the current folder.
- `rm -r old_folder`: deletes a folder and everything inside it.
- `ls build/` then `rm -rf build/`: preview what is in the folder first, then force-delete it without prompts.

### 9. cat: Display file contents

Prints files to standard output and concatenates multiple files: `cat a.txt b.txt > all.txt`. `-n` numbers lines. Best for small files; use `less` for large ones.

**Examples**

- `cat notes.txt`: prints the whole file to the screen.
- `cat part1.txt part2.txt > full.txt`: joins two files into a new one.
- `cat -n script.sh`: prints the file with line numbers.
- `cat > hello.txt`: type some text, then press `Ctrl+D` to save it as a new file.

### 10. less: Read files page by page

A pager that lets you scroll, search and jump without loading the whole file. Keys: space/`f` forward, `b` back, `/pattern` search, `n` next match, `G` end, `g` start, `q` quit, `F` follow like `tail -f`.

**Examples**

- `less /var/log/syslog`: opens a big log without loading it all at once.
- Inside `less`, type `/error` and press Enter to search, then `n` for the next match and `N` for the previous one.
- `G` jumps to the end, `g` to the start, `q` quits.
- `ls -l /etc | less`: page through long command output.

## Part 2: Searching, Permissions and Processes

### 11. grep: Search text using patterns

Prints lines matching a pattern. Flags: `-i` ignore case, `-r` recursive, `-n` line numbers, `-v` invert match, `-c` count, `-E` extended regex, `-l` list matching filenames, `-A/-B/-C` context lines. Example: `grep -rn "TODO" src/`.

**Examples**

- `grep "error" app.log`: prints every line of `app.log` containing "error".
- `grep -i "error" app.log`: same, ignoring case (matches `Error`, `ERROR`).
- `grep -rn "TODO" src/`: searches all files under `src/` and shows file name and line number, e.g. `src/main.py:42: # TODO fix`.
- `grep -v "DEBUG" app.log`: shows lines that do **not** contain "DEBUG".
- `grep -c "404" access.log`: prints how many lines contain "404".
- `grep -C 2 "crash" app.log`: shows each match with 2 lines of context above and below.

### 12. find: Find files and directories

Searches the filesystem by attributes: `find /var/log -name "*.log" -mtime -1 -size +10M`. Can run actions: `-exec rm {} \;` or pipe to `xargs`. Slower than `locate` but always current and far more flexible.

**Examples**

- `find . -name "notes.txt"`: looks for a file by exact name starting from the current folder (`.`).
- `find /var/log -name "*.log"`: finds all `.log` files under `/var/log`.
- `find . -type d -name "node_modules"`: finds directories only (`-type d`; use `-type f` for files).
- `find . -mtime -1`: files modified in the last day.
- `find . -size +100M`: files larger than 100 MB.
- `find . -name "*.tmp" -exec rm {} \;`: runs `rm` on every match (`{}` stands for the found file).

### 13. chmod: Change permissions

Sets read (r=4), write (w=2), execute (x=1) for user, group and others. Numeric: `chmod 755 script.sh`. Symbolic: `chmod u+x file`, `chmod go-w file`. `-R` applies recursively.

**Examples**

- `chmod +x deploy.sh`: makes a script executable so you can run `./deploy.sh`.
- `chmod 644 notes.txt`: owner can read and write; everyone else can only read (`rw-r--r--`).
- `chmod 755 script.sh`: owner can do everything; others can read and execute (`rwxr-xr-x`).
- `chmod 600 ~/.ssh/id_rsa`: private key readable only by you (SSH insists on this).
- `chmod go-w file.txt`: removes write permission from group and others.
- `chmod -R 755 public/`: applies the permission to a folder and everything in it.

### 14. chown: Change ownership

Changes owner and group: `chown user:group file`. `-R` for recursion. Usually requires root. Common when fixing permission issues on web or app directories.

**Examples**

- `sudo chown alice file.txt`: makes `alice` the owner of the file.
- `sudo chown alice:developers file.txt`: sets owner `alice` and group `developers`.
- `sudo chown -R www-data:www-data /var/www/site`: gives a web server user ownership of a whole site folder.
- `ls -l file.txt`: shows the owner and group columns so you can verify the change.

### 15. ps: Display process information

Snapshot of running processes. `ps aux` lists all processes with user, PID, CPU, memory and command; `ps -ef` is the System V style; `ps -p PID` inspects one. Combine with grep: `ps aux | grep nginx`.

**Examples**

- `ps`: shows only processes started from your current terminal.
- `ps aux`: every process on the system with user, PID, %CPU, %MEM and command.
- `ps aux | grep nginx`: finds the nginx processes and their PIDs (the `grep` line itself also appears).
- `ps -p 1234`: details for the single process with PID 1234.
- `ps aux --sort=-%mem | head -5`: the five processes using the most memory.

### 16. top: Monitor running processes

Live view of CPU, memory, load average and processes. Interactive keys: `P` sort by CPU, `M` by memory, `k` kill, `q` quit. The three load averages are 1, 5 and 15-minute run-queue lengths; compare them to the CPU count.

**Examples**

- `top`: opens a live view that refreshes every few seconds.
- Inside `top`, press `P` to sort by CPU, `M` to sort by memory, `q` to quit.
- Header line `load average: 0.50, 0.70, 0.90` means the average number of busy processes over the last 1, 5 and 15 minutes. On a 4-core machine, a load near 4 means the CPUs are fully busy.
- `top -u alice`: shows only processes owned by `alice`.

### 17. kill: Send signals to processes

Sends a signal to a PID. `kill PID` sends SIGTERM (15, graceful shutdown); `kill -9 PID` sends SIGKILL (cannot be caught, no cleanup); `kill -HUP` often reloads config. Try SIGTERM first. `pkill name` and `killall` target by name.

**Examples**

- `kill 1234`: politely asks process 1234 to stop (SIGTERM), letting it clean up.
- `kill -9 1234`: force-kills it immediately (SIGKILL); use only if the polite request is ignored.
- `kill -HUP 1234`: many daemons reload their configuration on this signal.
- `pkill firefox`: stops processes by name instead of PID.
- Typical workflow: `ps aux | grep myapp` to find the PID, then `kill <PID>`.

### 18. df: Check filesystem disk space

Reports used and free space per mounted filesystem. `df -h` for readable units, `df -i` to check inode usage (a disk can be "full" with free space if inodes run out).

**Examples**

- `df -h`: disk space per filesystem in readable units.
- Sample output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   32G   16G  67% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
```

- `df -h /home`: shows only the filesystem that holds `/home`.
- `df -i`: shows inode usage. If `IUse%` is 100% you cannot create files even with free space.

## Part 3: Text Processing and Utilities

### 19. head / tail: View the start or end of a file

`head -n 20 file` shows the first lines; `tail -n 50 file` the last. `tail -f app.log` follows new lines live, the standard way to watch logs (`tail -F` survives log rotation).

**Examples**

- `head -n 5 data.csv`: first 5 lines, a quick peek at a big file's header and format.
- `tail -n 20 app.log`: last 20 lines, usually the newest log entries.
- `tail -f app.log`: keeps the file open and prints new lines as they are written. Press `Ctrl+C` to stop.
- `head -n 20 app.log | tail -n 5`: lines 16 to 20 by combining both.

### 20. wc: Count lines, words, characters

`wc -l` counts lines, `-w` words, `-c` bytes. Example: `grep ERROR app.log | wc -l` counts error lines.

**Examples**

- `wc -l notes.txt`: number of lines, e.g. `42 notes.txt`.
- `wc -w essay.txt`: number of words.
- `ls | wc -l`: how many items are in the current folder.
- `grep ERROR app.log | wc -l`: how many lines contain `ERROR`.

### 21. sort / uniq: Sort and deduplicate output

`sort` orders lines (`-n` numeric, `-r` reverse, `-k` by field, `-u` unique). `uniq` collapses *adjacent* duplicates, so sort first; `uniq -c` counts. Classic: `sort | uniq -c | sort -nr` for frequency counts.

**Examples**

- `sort names.txt`: alphabetical order.
- `sort -n numbers.txt`: numeric order (otherwise `10` sorts before `9`).
- `sort -r names.txt`: reverse order.
- `sort names.txt | uniq`: removes duplicate lines (`uniq` only collapses *adjacent* duplicates, so sort first).
- `sort names.txt | uniq -c`: counts how many times each line appears, e.g. `3 alice`.
- `cut -d' ' -f1 access.log | sort | uniq -c | sort -nr | head`: top IP addresses in a web log.

### 22. cut / awk / sed: Extract and transform text

`cut -d, -f1,3` extracts columns. `awk` is a mini language for field-based processing: `awk '{print $1, $3}'`, or `awk -F: '$3 > 1000' /etc/passwd`. `sed` edits streams: `sed 's/old/new/g'`, `sed -i` edits in place, `sed -n '5,10p'` prints lines.

**Examples** (given a file `users.csv` with lines like `alice,30,Paris`)

- `cut -d, -f1 users.csv`: takes field 1 using comma as the delimiter, giving `alice`.
- `cut -d, -f1,3 users.csv`: fields 1 and 3, giving `alice,Paris`.
- `awk -F, '{print $1, $3}' users.csv`: prints name and city separated by a space.
- `awk -F, '$2 > 25' users.csv`: prints only rows where the age column is above 25.
- `sed 's/Paris/London/g' users.csv`: replaces every `Paris` with `London` in the output (original file unchanged).
- `sed -i 's/Paris/London/g' users.csv`: same, but edits the file in place.
- `sed -n '2,4p' users.csv`: prints only lines 2 to 4.

### 23. echo: Print text or variables

Prints its arguments: `echo "Hello $USER"`. `-e` interprets escapes. Often used with redirection to write files: `echo "data" >> file`. Print the last exit code with `echo $?`.

**Examples**

- `echo "Hello World"`: prints `Hello World`.
- `echo "Hi $USER"`: variables expand inside double quotes, e.g. `Hi sam`.
- `echo "line one" > notes.txt`: writes text into a file (overwrites).
- `echo "line two" >> notes.txt`: appends to the file.
- `ls /nonexistent; echo $?`: prints the exit code of the last command (`2` here; `0` means success).

### 24. du: Check directory size

Disk usage of files and directories. `du -sh dir` gives a summary in readable form; `du -h --max-depth=1 | sort -h` finds what is eating space. Differs from `df`, which reports filesystem-level usage.

**Examples**

- `du -sh projects/`: one summary line for the folder, e.g. `1.4G projects/`.
- `du -h --max-depth=1 /var`: size of each direct subfolder of `/var`.
- `du -h --max-depth=1 . | sort -h`: lists subfolders smallest to largest, so the biggest space hog is last.
- `du -sh *`: size of everything in the current folder.

### 25. free: Check memory usage

Shows total, used, free, shared, buff/cache and available RAM and swap. `free -h` is readable. Look at **available**, not free: Linux uses spare memory for cache and releases it when needed.

**Examples**

- `free -h`: memory in readable units. Sample output:

```
              total   used   free  shared  buff/cache  available
Mem:           15Gi   4.2Gi  1.1Gi   300Mi       10Gi       10Gi
Swap:         2.0Gi      0B  2.0Gi
```

- Here only 1.1 GiB is "free", but **available** is 10 GiB because the cache is released whenever programs need it. The machine is healthy.
- `free -h -s 2`: refreshes every 2 seconds.

### 26. tar / gzip / zip: Archive and compress files

`tar` bundles files; compression is separate. Create: `tar -czf out.tar.gz dir/`. Extract: `tar -xzf out.tar.gz`. List: `tar -tf`. `gzip` compresses a single file, `zip`/`unzip` handle the zip format. Remember: c=create, x=extract, z=gzip, f=file.

**Examples**

- `tar -czf backup.tar.gz myproject/`: **c**reate a g**z**ip-compressed archive, into **f**ile `backup.tar.gz`.
- `tar -tf backup.tar.gz`: lists what is inside without extracting.
- `tar -xzf backup.tar.gz`: **x**tract into the current folder.
- `tar -xzf backup.tar.gz -C /tmp/restore`: extract into a chosen folder.
- `gzip bigfile.log`: compresses to `bigfile.log.gz` (replacing the original); `gunzip bigfile.log.gz` reverses it.
- `zip -r site.zip site/` and `unzip site.zip`: the zip equivalents.

### 27. ln: Create hard and symbolic links

**Hard link** (`ln a b`): another name for the same inode; cannot cross filesystems or link directories; data survives until all links are gone. **Symbolic link** (`ln -s target link`): a pointer to a path; can cross filesystems and dangles if the target is removed.

**Examples**

- `ln -s /var/www/site-v2 /var/www/current`: creates a symbolic link `current` pointing to `site-v2`. Switching versions is just repointing the link.
- `ls -l current`: shows `current -> /var/www/site-v2`.
- `ln original.txt copy.txt`: creates a hard link. Both names refer to the same data, so editing one changes the other, and deleting one leaves the other intact.
- `ln -sf /opt/app/bin/app /usr/local/bin/app`: replaces an existing link (`-f`) to put a program on your PATH.

### 28. sudo: Run commands as superuser

Executes a command with another user's privileges (root by default), governed by `/etc/sudoers` (edit with `visudo`). Prefer targeted `sudo cmd` over a root shell. `sudo -u user cmd` runs as another user.

**Examples**

- `sudo apt update`: runs a package update as root.
- `sudo nano /etc/hosts`: edit a protected system file.
- `sudo -u postgres psql`: run a command as another user (`postgres`).
- `sudo !!`: repeats your last command with sudo, handy after "Permission denied".
- `sudo visudo`: safely edits the sudoers file (checks syntax before saving).

### 29. man / --help: Read documentation

`man command` opens the manual page (sections: 1 commands, 5 files, 8 admin). `/term` searches inside, `man -k keyword` searches descriptions. `command --help` gives a quick usage summary.

**Examples**

- `man ls`: opens the manual for `ls`. Press `q` to quit, `/keyword` to search.
- `man -k "copy files"`: searches manual descriptions when you do not know the command name.
- `ls --help`: a short usage summary printed straight to the terminal.
- `man 5 passwd`: the manual for the `/etc/passwd` file format (section 5), not the `passwd` command (section 1).

### 30. which / whereis: Locate commands

`which cmd` shows which executable would run (first match in `$PATH`). `whereis cmd` also shows source and man page locations. `type cmd` reveals aliases and builtins.

**Examples**

- `which python3`: prints the path that will run, e.g. `/usr/bin/python3`.
- `which node`: prints nothing if Node is not installed or not on your `PATH`.
- `whereis ls`: shows the binary and its man page, e.g. `ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz`.
- `type ll`: reveals that `ll` is an alias for `ls -alF`.

### 31. history: View previous commands

Lists your command history (`~/.bash_history`). `!!` repeats the last command, `!n` runs entry n, `Ctrl+R` does reverse incremental search. `sudo !!` reruns the last command with sudo.

**Examples**

- `history | tail -5`: your last five commands with their numbers.
- `history | grep ssh`: finds past commands that used `ssh`.
- `!!`: re-runs the previous command.
- `!42`: re-runs command number 42 from the history list.
- `Ctrl+R` then type `docker`: searches backwards through history; press Enter to run the match.

## Part 4: System Administration and Networking

### 32. useradd / passwd / usermod: Manage users

`useradd -m -s /bin/bash alice` creates a user with a home directory; `passwd alice` sets the password; `usermod -aG docker alice` appends a user to a group (always use `-a` with `-G` or you remove existing groups). Account data lives in `/etc/passwd`, `/etc/shadow`, `/etc/group`.

**Examples**

- `sudo useradd -m -s /bin/bash alice`: creates user `alice` with a home folder (`-m`) and bash as her shell (`-s`).
- `sudo passwd alice`: prompts you to set her password.
- `sudo usermod -aG docker alice`: adds her to the `docker` group. Keep `-a` (append); without it her other groups are removed.
- `id alice`: shows her user ID and groups so you can verify.
- `sudo userdel -r alice`: deletes the user and the home folder.

### 33. systemctl / journalctl: Manage services and read logs

On systemd systems: `systemctl status|start|stop|restart|enable|disable nginx`. `enable` makes it start at boot. `journalctl -u nginx -f` follows a service's logs; `--since "1 hour ago"` and `-p err` filter.

**Examples**

- `systemctl status nginx`: shows whether nginx is running, plus its latest log lines.
- `sudo systemctl start nginx` / `stop` / `restart`: control the service right now.
- `sudo systemctl enable nginx`: start it automatically at boot (`enable --now` also starts it immediately).
- `journalctl -u nginx -n 50`: last 50 log lines for the nginx service.
- `journalctl -u nginx -f`: follow its logs live.
- `journalctl -p err --since "1 hour ago"`: only errors from the past hour.

### 34. ssh / scp: Remote login and file copy

`ssh user@host` opens a secure shell; use key pairs (`ssh-keygen`, `ssh-copy-id`) rather than passwords. `scp file user@host:/path` copies over SSH (`rsync` is better for large or repeated transfers). Config shortcuts live in `~/.ssh/config`; `-L` creates port forwards.

**Examples**

- `ssh alice@192.168.1.10`: opens a remote shell on that server as `alice`.
- `ssh -p 2222 alice@example.com`: connects to a non-default port.
- `ssh-keygen -t ed25519` then `ssh-copy-id alice@example.com`: creates a key pair and installs the public key so you can log in without a password.
- `scp report.pdf alice@example.com:/home/alice/`: copies a local file to the server.
- `scp alice@example.com:/var/log/app.log .`: copies a remote file to the current folder.
- `rsync -avz project/ alice@example.com:/srv/project/`: syncs a folder, sending only changes.

### 35. curl / wget: Make web requests and download files

`curl` is the Swiss-army HTTP client: `curl -X POST -H "Content-Type: application/json" -d '{...}' url`, `-I` headers only, `-o` save, `-L` follow redirects, `-s` silent. `wget` is simpler for downloading files and mirroring.

**Examples**

- `curl https://example.com`: prints the page's HTML to the terminal.
- `curl -I https://example.com`: shows only response headers (status code, content type).
- `curl -L -o page.html https://example.com`: follows redirects (`-L`) and saves to a file (`-o`).
- `curl -X POST -H "Content-Type: application/json" -d '{"name":"sam"}' https://api.example.com/users`: sends JSON to an API.
- `curl -s https://api.example.com/status | head`: silent mode, without the progress meter.
- `wget https://example.com/file.zip`: downloads a file into the current folder.

### 36. ping / traceroute: Test network connectivity

`ping host` sends ICMP echo requests and reports latency and loss; it tests reachability. `traceroute host` shows each hop along the path to find where delay or loss occurs. Some networks block ICMP, so failure does not always mean down.

**Examples**

- `ping -c 4 google.com`: sends 4 echo requests and reports each round-trip time, e.g. `time=12.3 ms`, plus a packet-loss summary.
- `ping 192.168.1.1`: tests whether your router is reachable (stop with `Ctrl+C`).
- `traceroute google.com`: lists each router hop on the way, with delays. A hop where delays jump or replies stop shows where the problem is.
- Troubleshooting order: ping your router, then ping `8.8.8.8` (internet works?), then ping a domain name (DNS works?).

### 37. ss / netstat / ip: Inspect network connections and interfaces

`ss -tulpn` shows listening TCP/UDP sockets with owning processes (modern replacement for `netstat`). `ip addr` shows addresses, `ip route` the routing table, `ip link` the interfaces.

**Examples**

- `ss -tulpn`: lists listening TCP (`t`) and UDP (`u`) ports with the owning program (`p`) and numeric ports (`n`). You may need `sudo` to see process names.
- `ss -tulpn | grep :80`: what is listening on port 80?
- `ip addr` (or `ip a`): shows your network interfaces and IP addresses.
- `ip route`: shows the routing table, including the default gateway (`default via 192.168.1.1`).
- `ip link show`: lists interfaces and whether they are up.

### 38. crontab: Schedule recurring jobs

`crontab -e` edits your schedule. Format: `minute hour day-of-month month day-of-week command`, e.g. `0 2 * * * /backup.sh` runs daily at 2:00. Cron has a minimal environment: use absolute paths and redirect output to a log.

**Examples** (fields are: minute, hour, day of month, month, day of week)

- `crontab -e`: opens your schedule for editing; `crontab -l` lists it.
- `0 2 * * * /home/sam/backup.sh`: every day at 02:00.
- `*/15 * * * * /home/sam/check.sh`: every 15 minutes.
- `30 8 * * 1-5 /home/sam/report.sh`: 08:30 on weekdays (Mon to Fri).
- `0 3 * * 0 /home/sam/weekly.sh >> /home/sam/weekly.log 2>&1`: Sundays at 03:00, with output saved to a log. Use absolute paths because cron has a minimal environment.

### 39. jobs / bg / fg / nohup: Manage background processes

`cmd &` runs in the background; `Ctrl+Z` suspends; `bg` resumes it in the background, `fg` brings it to the foreground, `jobs` lists them. `nohup cmd &` keeps it running after you log out (output goes to `nohup.out`). For long-lived sessions use `tmux` or `screen`.

**Examples**

- `sleep 300 &`: starts the command in the background and returns your prompt immediately.
- `jobs`: lists background and suspended jobs, e.g. `[1]+ Running sleep 300 &`.
- Start `vim notes.txt`, press `Ctrl+Z` to suspend it, run other commands, then `fg` to return to it.
- `bg %1`: resumes suspended job 1 in the background; `fg %1` brings it to the foreground.
- `nohup ./long_job.sh &`: keeps running after you close the terminal; output goes to `nohup.out`.
- For long sessions, `tmux` lets you detach and reattach later.

### 40. export / env: Set and view environment variables

`VAR=value` sets a shell variable; `export VAR=value` makes it available to child processes. `env` lists the environment (or runs a command with a modified one). Persist settings in `~/.bashrc` or `~/.profile`.

**Examples**

- `NAME=sam` then `echo $NAME`: a plain shell variable, visible only in this shell.
- `export API_URL=https://api.example.com`: also visible to programs you launch from this shell.
- `bash -c 'echo $API_URL'`: a child process, which prints the value because it was exported.
- `env | grep API`: lists environment variables containing "API".
- `echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc`: makes a setting permanent by putting it in your shell startup file (open a new terminal or run `source ~/.bashrc`).

## Part 5: Power Tools and Scripting

### 41. Pipes and redirection (|, >, >>, 2>, tee)

`|` feeds one command's output into the next. `>` overwrites a file, `>>` appends, `<` reads input, `2>` redirects stderr, `2>&1` merges stderr into stdout, `&> file` captures both. `tee` writes to a file *and* passes output on. File descriptors: 0 stdin, 1 stdout, 2 stderr.

**Examples**

- `ls | wc -l`: the pipe `|` sends the output of `ls` into `wc -l`, which counts the items.
- `ps aux | grep nginx | head -3`: chain three commands.
- `ls > files.txt`: writes the output to a file, overwriting it.
- `echo "done" >> log.txt`: appends instead of overwriting.
- `sort < names.txt`: feeds a file into a command's input.
- `ls /nonexistent 2> errors.txt`: sends only error messages to a file (stream 2).
- `make > build.log 2>&1`: sends normal output and errors to the same file.
- `ls | tee list.txt | wc -l`: saves the listing to `list.txt` *and* counts it.

### 42. xargs: Build commands from input

Converts lines from stdin into arguments for a command: `find . -name "*.tmp" | xargs rm`. Use `-0` with `find -print0` to handle spaces in filenames; `-n 1` runs one argument at a time; `-P` runs in parallel.

**Examples**

- `echo "a b c" | xargs mkdir`: turns the words into arguments, running `mkdir a b c`.
- `find . -name "*.tmp" | xargs rm`: deletes every `.tmp` file found.
- `find . -name "*.txt" -print0 | xargs -0 wc -l`: the `-print0` / `-0` pair handles file names containing spaces.
- `cat urls.txt | xargs -n 1 curl -O`: runs `curl` once per URL (`-n 1`).
- `ls *.jpg | xargs -P 4 -I {} convert {} {}.png`: converts four images at a time in parallel.

### 43. diff: Compare files

Shows line differences. `diff -u a b` produces unified format (what patches and Git use); `diff -r` compares directories; `-q` reports only whether files differ. `sdiff` and `vimdiff` show side by side.

**Examples** (with `a.txt` containing `apple, banana` and `b.txt` containing `apple, cherry`)

- `diff a.txt b.txt`: prints `2c2`, `< banana`, `> cherry`, meaning line 2 differs: left has banana, right has cherry.
- `diff -u a.txt b.txt`: unified format with `-banana` and `+cherry`, the same style Git uses.
- `diff -r dir1 dir2`: compares two folders recursively.
- `diff -q a.txt b.txt`: just reports `Files a.txt and b.txt differ`.
- `diff a.txt b.txt > changes.patch`: saves the differences to a file.

### 44. lsof: List open files and ports

In Unix everything is a file, so this shows files, sockets and pipes opened by processes. `lsof -i :8080` finds what is using a port; `lsof -p PID` lists a process's files; `lsof +L1` finds deleted-but-still-open files that consume disk.

**Examples**

- `sudo lsof -i :8080`: which process is using port 8080? Output includes the command name and PID.
- `sudo kill $(sudo lsof -t -i :8080)`: `-t` prints only the PID, so this frees the port.
- `lsof -p 1234`: all files and sockets opened by process 1234.
- `lsof /var/log/syslog`: which processes have this file open.
- `sudo lsof +L1`: finds deleted files still held open, a common reason disk space is not freed after `rm`.

### 45. lsblk / mount: Disks and mounting

`lsblk` shows block devices and partitions as a tree. `mount /dev/sdb1 /mnt/data` attaches a filesystem to a directory; `umount` detaches it. Persist mounts in `/etc/fstab` (prefer UUIDs). `findmnt` and `blkid` help inspect.

**Examples**

- `lsblk`: shows disks and partitions as a tree, e.g. `sda` with `sda1` mounted at `/`.
- `sudo mount /dev/sdb1 /mnt/usb`: attaches a USB partition to the folder `/mnt/usb` (create the folder first).
- `ls /mnt/usb`: now shows the USB's files.
- `sudo umount /mnt/usb`: safely detaches it before unplugging.
- `blkid`: shows each partition's UUID, which you put in `/etc/fstab`, e.g. `UUID=abcd-1234 /mnt/data ext4 defaults 0 2`, to mount at boot.

### 46. apt / yum / dnf: Package management

Install and update software from repositories. Debian/Ubuntu: `apt update && apt install nginx`, `apt remove`, `apt upgrade`. RHEL/CentOS/Fedora: `yum` (older) and `dnf` (newer) with `install`, `remove`, `update`. Dependencies are resolved automatically.

**Examples**

- Debian/Ubuntu: `sudo apt update` refreshes the package list, then `sudo apt install nginx` installs nginx and its dependencies.
- `sudo apt remove nginx`: uninstalls it; `sudo apt upgrade`: updates all installed packages.
- `apt search image editor`: searches available packages.
- RHEL/Fedora: `sudo dnf install httpd` and `sudo dnf update` do the same jobs (`yum` is the older name).
- `dpkg -l | grep nginx`: checks whether a package is installed (Debian family).

### 47. htop: Interactive process viewer

A friendlier `top` with colors, per-core CPU bars, mouse support, tree view (`F5`), search, and kill/renice from the UI. Usually needs installing separately.

**Examples**

- `htop`: opens a coloured live view with one bar per CPU core and a memory bar.
- Press `F6` to choose the sort column (e.g. `PERCENT_MEM`), or click the column header.
- Press `F3` and type a name to search, then `F9` to kill the highlighted process.
- Press `F5` for a tree view showing which process started which.
- `sudo apt install htop`: installs it if the command is not found.

### 48. vim / nano: Terminal text editors

`nano` is beginner-friendly (shortcuts shown at the bottom, `Ctrl+O` save, `Ctrl+X` exit). `vim` is modal: `i` insert, `Esc` normal, `:w` save, `:q` quit, `:wq` both, `:q!` discard, `/text` search, `dd` delete line, `yy` copy line, `p` paste, `u` undo. Knowing how to exit vim is the classic joke and a real skill.

**Examples**

- `nano notes.txt`: opens a simple editor. Type normally, press `Ctrl+O` then Enter to save, `Ctrl+X` to exit.
- `vim notes.txt`: opens vim in *normal* mode, where keys are commands, not text.
- In vim: press `i`, type your text, press `Esc` to leave insert mode, then `:wq` and Enter to save and quit.
- In vim: `:q!` quits and throws away changes; `/word` searches; `dd` deletes the current line; `u` undoes.
- Stuck in vim? Press `Esc` a couple of times, then type `:q!` and Enter.

### 49. umask and permission numbers (755, 644)

Permissions are three digits (owner, group, others), each a sum of r=4, w=2, x=1. **755** = rwxr-xr-x (executables, directories), **644** = rw-r--r-- (normal files), **600** = private files like SSH keys. **umask** subtracts bits from defaults: default 666 for files and 777 for directories, so umask `022` yields 644 and 755. Also know setuid, setgid and the sticky bit (`/tmp`).

**Examples**

- `ls -l script.sh` shows `-rwxr-xr-x`: owner `rwx` = 7, group `r-x` = 5, others `r-x` = 5, so the mode is **755**.
- `chmod 644 notes.txt` gives `-rw-r--r--` (6 = read+write, 4 = read).
- `chmod 600 secret.key` gives `-rw-------`, readable only by you.
- `umask`: prints the current mask, usually `0022`.
- With umask `022`: new files start at 666 minus 022 = **644**; new folders at 777 minus 022 = **755**.
- `umask 077`: new files are `600` and folders `700`, private to you.

### 50. Shell scripting basics (variables, loops, conditionals, exit codes)

```
#!/bin/bash
set -euo pipefail          # fail fast on errors/undefined vars
name="world"
if [ -f "$1" ]; then echo "exists"; else echo "missing"; fi
for f in *.log; do gzip "$f"; done
while read -r line; do echo "$line"; done < input.txt
```

Rules: no spaces around `=`, quote variables (`"$var"`), `$1`/`$@` are arguments, `$?` is the last exit code (0 means success, nonzero is failure), `&&` runs the next command only on success and `||` only on failure. Make scripts executable with `chmod +x`.

**Worked example: a small backup script**

Save this as `backup.sh`, run `chmod +x backup.sh`, then `./backup.sh ~/projects`:

```
#!/bin/bash
set -euo pipefail

source_dir="$1"                      # first argument
if [ ! -d "$source_dir" ]; then       # conditional: is it a directory?
  echo "Error: $source_dir is not a folder" >&2
  exit 1                              # non-zero exit code means failure
fi

stamp=$(date +%F)                     # command substitution, e.g. 2026-10-31
tar -czf "backup-$stamp.tar.gz" "$source_dir"

for f in backup-*.tar.gz; do          # loop over matching files
  echo "Found backup: $f"
done
echo "Backup complete"
```

- `./backup.sh ~/projects`: creates `backup-2026-10-31.tar.gz`, lists backups and ends with exit code 0.
- `./backup.sh nothere`: prints the error and exits with code 1.
- `echo $?` right after: shows the exit code (`0` success, non-zero failure).
- `./backup.sh ~/projects && echo "ok" || echo "failed"`: `&&` runs only on success, `||` only on failure.

---

## Practice Prompts

1. Find the 5 largest files under `/var`.
2. Count requests per IP in an access log and show the top 10.
3. A port is in use: find the process and stop it.
4. The disk shows full but `du` disagrees. What could be happening? (Hint: deleted-but-open files, inodes.)
5. Write a cron entry that runs a backup every Sunday at 3 AM.
