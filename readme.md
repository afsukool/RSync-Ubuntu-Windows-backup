# Windows to Ubuntu Backup Project Using rsync

This project details how to set up a regular backup of a Windows machine to an Ubuntu server using `rsync`. The following instructions provide a step-by-step guide to configuring both the Ubuntu server and the Windows client. 

## Getting Started

### Ubuntu Server Setup

1. **Install rsync**:
   ```bash
   sudo apt-get install rsync
   ```

2. **Create `rsyncd.conf`**:
   ```bash
   sudo nano /etc/rsyncd.conf
   ```
   Add the following configuration, replacing `username` with your Ubuntu username:
   ```ini
   [usernamebackup]
       path = /home/username/backup
       comment = Backup
       uid = username
       gid = username
       read only = false
       auth users = username
       secrets file = /etc/rsyncd.secrets
   ```

3. **Set permissions for `rsyncd.conf`**:
   ```bash
   sudo chmod 644 /etc/rsyncd.conf
   ```

4. **Create `rsyncd.secrets`**:
   ```bash
   sudo nano /etc/rsyncd.secrets
   ```
   Add the following line, replacing `username` with your Ubuntu username and `password` with your chosen password:
   ```
   username:password
   ```

5. **Set permissions for `rsyncd.secrets`**:
   ```bash
   sudo chmod 600 /etc/rsyncd.secrets
   ```

6. **Enable and restart `rsync`**:
   ```bash
   sudo nano /etc/default/rsync
   ```
   Set `RSYNC_ENABLE` to `true`:
   ```
   RSYNC_ENABLE=true
   ```
   Restart `rsync`:
   ```bash
   sudo /etc/init.d/rsync restart
   ```

### Windows Client Setup

1. **Install Cygwin**:
   - Ensure `Editors > nano` and `Net > rsync` are selected during installation.

2. **Update Windows PATH**:
   - Add `C:\cygwin\bin;` to the Windows PATH:
     - Right-click on "My Computer" and select "Properties".
     - Switch to the "Advanced" tab and click "Environment Variables".
     - Find the "Path" variable in the "System variables" section and click "Edit".
     - Add `C:\cygwin\bin;` to the beginning of the list.

3. **Create secret file in Cygwin**:
   - Start Cygwin Bash Shell:
     ```bash
     nano /secret
     ```
   - Enter the password from `rsyncd.secrets` and save the file.
   - Set permissions and ownership:
     ```bash
     chmod 600 /secret
     chown Administrator:SYSTEM /secret
     ```

4. **Create a batch file to run rsync**:
   - Open Notepad and enter the following command, replacing placeholders with your information:
     ```cmd
     C:\cygwin\bin\rsync.exe -qrtz --password-file=c:\cygwin\secret --delete "/cygdrive/c/Documents and Settings/User Name" username@ipaddress::usernamebackup
     ```
   - Save the file as `C:\rsync.bat`.

### Schedule the Backup

1. **Create a Scheduled Task**:
   - Go to Start > Programs > Accessories > System Tools > Scheduled Tasks.
   - From the File menu, select New > Scheduled Task and name it "rsync backup".
   - Right-click on the task and select "Properties".
   - Enter `C:\rsync.bat` in the "Run" field.
   - Switch to the "Schedule" tab and select the time for the backup to run daily.

2. **Test the Scheduled Task**:
   - Create a folder `C:\data` and add a few files.
   - Edit `C:\rsync.bat` to use `"/cygdrive/c/data"` instead of `"/cygdrive/c/Documents and Settings/User Name"`.
   - Add a `pause` command at the end of the batch file.
   - Run the scheduled task and troubleshoot any errors.
   - Once confirmed, revert changes in `C:\rsync.bat` and update "Run as:" to `NT AUTHORITY\SYSTEM` in the scheduled task properties.

### Run the First Backup

Execute `C:\rsync.bat` from the command line. Initial backups may take several hours, but subsequent backups will be faster due to `rsync`'s efficiency.

---

This project is designed to ensure a reliable and efficient backup system for your Windows XP machine to an Ubuntu server using `rsync`. For more details, consult the `rsync` man page and online resources.
