# Common-error
# Kali Linux - Root Login Fails on GDM3 ("Password Authentication Did Not Work")

## Error

After upgrading Kali Linux, attempting to log in as the `root` user from the graphical login screen (GDM3) fails with the message:

```text
Password authentication did not work
```

Even though:

* The root password is correct.
* `su -` works from the terminal.
* `sudo -i` works.
* The root account is not locked.

---

## Environment

* Operating System: Kali Linux
* Display Manager: GDM3
* Login Method: Graphical Login Screen (GUI)
* Root Account: Enabled

---

## Symptoms

### GUI Login Fails

When attempting to log in as:

```text
Username: root
Password: <correct password>
```

GDM3 displays:

```text
Password authentication did not work
```

### Terminal Login Works

The following command successfully switches to the root user:

```bash
su -
```

or

```bash
sudo -i
```

This indicates that the root account and password are valid.

---

## Root Cause

GDM3 uses PAM (Pluggable Authentication Modules) for authentication.

The file:

```bash
/etc/pam.d/gdm-password
```

contains the following rule:

```text
auth required pam_succeed_if.so user != root quiet_success
```

This rule explicitly prevents the `root` user from authenticating through the GDM3 graphical login screen.

### Explanation

```text
user != root
```

means:

> Allow all users except root.

As a result, root authentication is denied before password verification is completed.

---

## Verify the Issue

Check whether the blocking rule exists:

```bash
grep -n root /etc/pam.d/gdm-password
```

Expected output:

```text
3:auth required pam_succeed_if.so user != root quiet_success
```

Verify the root account is not locked:

```bash
passwd -S root
```

Expected output:

```text
root P YYYY-MM-DD 0 99999 7 -1
```

Where:

```text
P = Password Set
L = Locked
```

---

## Solution

### Step 1: Create a Backup

Always back up the PAM configuration before making changes.

```bash
cp /etc/pam.d/gdm-password /etc/pam.d/gdm-password.bak
```

---

### Step 2: Edit the PAM Configuration

Open the file:

```bash
nano /etc/pam.d/gdm-password
```

Find:

```text
auth required pam_succeed_if.so user != root quiet_success
```

Comment it out:

```text
# auth required pam_succeed_if.so user != root quiet_success
```

Save and exit.

---

### Step 3: Restart GDM3

Restart the display manager:

```bash
systemctl restart gdm3
```

or reboot the system:

```bash
reboot
```

---

## Verification

After reboot:

1. Go to the login screen.
2. Select "Not Listed?" if required.
3. Enter:

```text
Username: root
Password: <your root password>
```

Root login should now work.

---

## Rollback

If the system behaves unexpectedly after modification:

Restore the original configuration:

```bash
cp /etc/pam.d/gdm-password.bak /etc/pam.d/gdm-password
```

Restart GDM3:

```bash
systemctl restart gdm3
```

or reboot:

```bash
reboot
```

---

## Security Warning

Allowing direct graphical root login is generally discouraged because:

* A compromised GUI session provides immediate root access.
* Malware or malicious applications gain full system privileges.
* Mistakes made while running as root can damage the operating system.

Recommended practice:

```bash
sudo -i
```

or

```bash
su -
```

from a standard user account.

---

## Quick Fix Summary

```bash
cp /etc/pam.d/gdm-password /etc/pam.d/gdm-password.bak

nano /etc/pam.d/gdm-password

# Comment:
# auth required pam_succeed_if.so user != root quiet_success

systemctl restart gdm3
```

---

## Tags

```text
kali-linux
gdm3
root-login
linux
pam
authentication
password-authentication-did-not-work
root-user
troubleshooting
vmware
```
