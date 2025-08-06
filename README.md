# Rsync Backup Configuration Guide

This guide provides instructions to set up and automate backups of the `/usr/share/nginx/html` directory from a master server to a remote server using `rsync` over SSH, including passwordless authentication and scheduling via crontab.

## Prerequisites
- Two Linux servers (master and remote) with `dnf` package manager (e.g., CentOS, RHEL, or Fedora).
- Root or sudo privileges on both servers.
- SSH access from the master server to the remote server.
- Nginx installed with the default web directory at `/usr/share/nginx/html`.

## Setup Instructions (On Master Server Only)

### 1. Install rsync
Install the `rsync` package on the master server.

```bash
sudo dnf install rsync -y
```

### 2. Verify Installation
Check if `rsync` is installed.

```bash
rpm -qa | grep rsync
```

### 3. Perform Initial Backup
Backup the contents of `/usr/share/nginx/html` to the remote server.

```bash
rsync -avz -e ssh /usr/share/nginx/html/ root@192.0.2.100:/usr/share/nginx/html
```

*Note*: Replace `192.0.2.100` with the actual IP address of the remote server. You will need to enter the remote server's root password for this step.

### 4. Set Up Passwordless SSH
To eliminate the need for a password during `rsync`, configure SSH key-based authentication.

#### Generate SSH Key
```bash
ssh-keygen
```
Press Enter for all prompts to use default settings (no passphrase).

#### Copy SSH Key to Remote Server
```bash
ssh-copy-id root@192.0.2.100
```
Enter the remote server's root password when prompted. Replace `192.0.2.100` with the actual IP address of the remote server.

### 5. Test Passwordless Backup
Run the `rsync` command again to confirm it works without a password prompt.

```bash
rsync -avz -e ssh /usr/share/nginx/html/ root@192.0.2.100:/usr/share/nginx/html
```

### 6. Automate Backup with Crontab
Schedule the `rsync` command to run automatically using `crontab`.

#### Open Crontab for Editing
```bash
crontab -e
```

#### Add Rsync Command
Add the following line to run the backup every 5 minutes (adjust the schedule as needed, e.g., `*/1 * * * *` for every minute):

```
*/5 * * * * rsync -avz -e ssh /usr/share/nginx/html/ root@192.0.2.100:/usr/share/nginx/html
```

Save and exit the editor.

## Notes
- Replace `192.0.2.100` with the actual IP address of the remote server.
- Ensure the remote server has sufficient storage and appropriate permissions for `/usr/share/nginx/html` (e.g., owned by `root` or the target user).
- Verify SSH connectivity and key-based authentication before automating (`ssh root@192.0.2.100` should connect without a password).
- Check the crontab schedule with `crontab -l` to confirm the job is added.
- Monitor the backup process using logs (e.g., `/var/log/cron` or rsync output redirection).
- For enhanced security, consider using a non-root user for SSH and rsync operations.
