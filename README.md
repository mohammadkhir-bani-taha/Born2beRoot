*This project has been created as part of the 42 curriculum by mobani-t.*


# Born2beRoot

## Project Description


Born2beRoot is a 42 system administration project focused on setting up a secure Linux server inside a virtual machine.


The goal of the project is to understand the fundamentals of system administration, virtualization, disk partitioning, encryption, Linux users and permissions, SSH, firewall configuration, password policies, sudo, security modules, and system monitoring.


For this project, I installed **Debian 13 (Trixie)** as a minimal operating system inside **VirtualBox**, without a graphical interface.


The main features of my setup are:


- Debian 13 (Trixie) running inside VirtualBox.
- Minimal installation without a graphical desktop environment.
- LUKS-encrypted disk with LVM logical volumes.
- Separate logical volumes for the system and different directories.
- Hostname set to `mobani-t42`.
- Regular user `mobani-t`.
- User `mobani-t` belongs to the `sudo` and `user42` groups.
- SSH service running on port `4242` inside the virtual machine.
- Root SSH login disabled.
- UFW firewall enabled, with port `4242` allowed.
- Strong password policy configured.
- Sudo configured with strict security rules and logging.
- AppArmor enabled at startup.
- monitoring.sh Bash script executed every 10 minutes with cron.

---

## Instructions

### Submitted files

The documentation and signature files are located at the root of my GitHub repository (the official 42 submission destination must be confirmed separately):

```text
README.md
signature.txt
```

The virtual machine and its virtual disk file are not included in the Git repository.

### Running the virtual machine

The virtual machine is started using VirtualBox.

During startup, the encrypted disk must be unlocked using the passphrase configured during installation.

The startup process is:

1. Start the virtual machine in VirtualBox.
2. Enter the disk encryption passphrase.
3. Wait for Debian to boot.
4. A terminal login prompt appears.
5. Log in using the regular user account.

There is no graphical desktop environment installed.

### Connecting through SSH

SSH is configured to listen on port `4242` inside the virtual machine.

Example connection from the host:

```bash
ssh -p 44242 mobani-t@127.0.0.1
```

My VirtualBox networking uses NAT port forwarding: Ubuntu host TCP `44242` → Debian guest TCP `4242`. The important settings inside the virtual machine are:

```text
Guest SSH port: 4242
Ubuntu host forwarded port: 44242
PermitRootLogin: no
```
Root login should fail:

```bash
ssh -p 44242 root@127.0.0.1
```

### Generating the signature

The virtual machine must be completely powered off before calculating its disk signature.

On my **Ubuntu host**, the SHA-1 hash was generated from the powered-off VirtualBox disk:

```bash
VDI="/goinfre/mobani-t/Born2beRoot/vm/Born2beRoot/Born2beRoot.vdi"
sha1sum "$VDI" | awk '{print $1}' > signature.txt
```

Verify the saved value against the original virtual disk (with the VM powered off):

```bash
test "$(cat signature.txt)" = "$(sha1sum "$VDI" | awk '{print $1}')" \
  && echo "SIGNATURE MATCH"
```

I confirmed the result `SIGNATURE MATCH` before pushing `signature.txt` to GitHub. The `VDI` path above is specific to the 42 Amman Ubuntu workstation and may differ on another host.

The VM must not be started again after the final signature is generated, because starting the system can modify files, logs, and metadata inside the virtual disk, which can change its SHA1 hash.

### Snapshot rule

The virtual machine must have no snapshots when it is presented at the beginning of the evaluation.

The evaluator may create a temporary snapshot during the evaluation and remove it afterwards, as described in the project subject.

Before generating the final signature, make sure that the VM is powered off and that no snapshots remain.

---

## Operating System Choice

### Why Debian?

I chose Debian because it is stable, well documented, widely used for servers, and recommended for people who are new to system administration.

Debian allowed me to focus on the core concepts of the project: minimal installation, secure configuration, package management, users, permissions, SSH, firewall, sudo, and monitoring.

### Debian vs Rocky Linux

Debian is a community-based Linux distribution. It is developed and maintained by a large open-source community rather than being controlled by a single company. It uses the `apt` package manager.

Rocky Linux is enterprise-oriented and RHEL-compatible. RHEL means Red Hat Enterprise Linux. Rocky Linux follows the Red Hat ecosystem and is commonly used as a free enterprise server distribution. It uses the `dnf` package manager.

**Debian advantages:**

- Stable and well documented.
- Supported by a large open-source community.
- Provides straightforward package management with APT.

**Debian disadvantage:**

- Stable releases may provide older package versions than newer distributions.

**Rocky Linux advantages:**

- Compatible with the RHEL ecosystem.
- Designed for enterprise-oriented server environments.
- Uses the DNF package manager.

**Rocky Linux disadvantage:**

- Uses a different package-management and administration ecosystem from Debian.

Examples:

```bash

# Debian
sudo apt update
sudo apt install package_name

# Rocky Linux
sudo dnf check-update
sudo dnf install package_name
```

Debian commonly uses AppArmor, while Rocky Linux commonly uses SELinux. Debian also commonly uses UFW, while Rocky Linux commonly uses firewalld.

I chose Debian because it is simpler for learning the fundamentals of system administration and because it is highly suitable for this project.

### apt vs aptitude

`apt` is the standard command-line package management tool used on Debian-based systems. It is used to install, update, upgrade, and remove packages.

Examples:
```bash
sudo apt update
sudo apt upgrade
sudo apt install package_name
sudo apt remove package_name
sudo apt purge package_name
```

`aptitude` is another package manager. It also provides an interactive interface and can offer alternative dependency-resolution solutions. In everyday Debian administration, `apt` is commonly used and straightforward.


---

## Virtualization

Virtualization allows multiple operating systems to run as virtual machines on the same physical computer.

A virtual machine has virtual hardware resources such as a CPU, RAM, disk, and network interface.

Using virtualization makes it possible to install and configure a server in an isolated environment without directly modifying the host operating system.

### VirtualBox vs UTM

**VirtualBox**

VirtualBox is a cross-platform virtualization application available for Windows, Linux, and macOS.

It provides:

- Virtual machine creation and management.
- Virtual disk management.
- NAT and bridged networking.
- Port forwarding.
- Snapshot management.

VirtualBox commonly uses the `VDI` virtual disk format.

**UTM**

UTM is primarily designed for Apple platforms, especially macOS. It uses technologies such as Apple's Virtualization framework and QEMU.

It supports virtualization and emulation, including virtual machines on Apple Silicon.

UTM commonly uses `QCOW2` disk images.

**My choice**

I chose VirtualBox on an **Ubuntu 22.04.5 LTS host** because it provides the virtualization and NAT port-forwarding features required for this project.

### Virtual machine configuration

My virtual machine configuration:

```text
Operating system: Debian 13 (Trixie)
Virtualization software: VirtualBox
RAM: 2048 MB allocated
CPU: 2 virtual CPUs
Disk: dynamically allocated VDI (approximately 30 GB configured)
Disk format: VDI
Graphical interface: Not installed
Hostname: mobani-t42
```

---

## No Graphical Environment

Born2beRoot requires a minimal server installation without a graphical interface.

I installed Debian without a desktop environment to keep the system lightweight and focus on command-line administration.

The system boots into a terminal login prompt instead of a graphical desktop.

Useful checks:

```bash
systemctl get-default
```

The expected default target is:

```text
multi-user.target
```

---

## Partitioning, Encryption, and LVM

The disk was configured using encrypted LVM.

### Disk layout

The logical volume sizes below reflect the setup used for this project. Run `lsblk -f` and `lsblk -o NAME,SIZE,TYPE,MOUNTPOINTS` to inspect the exact active layout.

```text
/dev/sda
├─sda1              ext4                         /boot
└─sda5              LUKS
  └─sda5_crypt      LUKS unlocked device
    └─mobani-t42-vg (LVM volume group)
      ├─root          ~7.5G                       /
      ├─var           ~2.3G                       /var
      ├─swap_1        ~1.5G                       [SWAP]
      ├─tmp           ~680M                       /tmp
      ├─home          ~4.7G                       /home
      ├─srv           ~2.8G                       /srv
      └─var-log       ~2.8G                       /var/log
```

These are approximate logical-volume sizes, not an exact copy of `lsblk` output. The partition layout includes directories shown in the subject's bonus partitioning example; this does **not** mean that the bonus services were installed.

### Why is `/boot` not encrypted?

`/boot` is separate and not encrypted because the bootloader needs access to the kernel and initramfs before the encrypted volume can be unlocked.

The encrypted partition is unlocked during boot as `sda5_crypt`.

### Why disk encryption?

Encryption protects the content of the virtual disk. Without the passphrase, the data inside the encrypted partition cannot be accessed directly, even if the virtual disk file is copied.

### Why LVM?

LVM stands for Logical Volume Manager.

It allows a physical storage device to be divided into logical volumes that can be managed separately.

The main advantages are:

- Flexible storage management.
- Ability to organize the system into separate logical volumes.
- Easier management of available space.
- Logical separation of important directories.

In my setup, LVM is inside the encrypted partition.

Useful commands:


```bash
lsblk
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay
```

---

## User and Group Management

The regular user created during installation is:

```text
mobani-t
```

The user belongs to the following groups:

```text
sudo
user42
```

The `sudo` group allows the user to execute administrative commands using sudo.

The `user42` group is required by the Born2beRoot subject.

Useful commands:

```bash
whoami
groups mobani-t
getent group sudo
getent group user42
```


During evaluation, a new user and group can be created to demonstrate user and group management:

```bash
sudo adduser evaluser
sudo groupadd evaluating
sudo usermod -aG evaluating evaluser
groups evaluser
```

The `-aG` options add the user to a supplementary group without removing existing group memberships.

---

## Hostname

The hostname identifies the machine on the system and network.

The project requires the hostname to end with `42`.

My hostname is:

```text
mobani-t42
```

Useful checks:

```bash
hostname
cat /etc/hostname
cat /etc/hosts
```


The hostname can be changed using:

```bash
sudo hostnamectl set-hostname newhostname
sudo nano /etc/hosts
sudo reboot
```


---

## SSH Configuration

SSH stands for Secure Shell.

It is a network protocol used to connect securely to another computer over a network.

SSH encrypts the communication between the client and server, protecting transmitted information such as usernames, passwords, and commands.

In this project:

```text
SSH port inside VM: 4242
Root SSH login: disabled
```

Important configuration values in `/etc/ssh/sshd_config`:

```text
Port 4242
PermitRootLogin no
```

Useful checks:

```bash
sudo service ssh status
sudo ss -tlnp | grep 4242
sudo /usr/sbin/sshd -T | grep -E '^(port|permitrootlogin)'
```

Command explanation:
```text
ss      -> shows socket connections
-t      -> TCP
-l      -> listening ports
-n      -> numeric addresses and ports
-p      -> process information
grep    -> filters the result
```
Root SSH login is disabled because direct remote root access is dangerous. A normal user should connect first and use `sudo` only when administrative privileges are needed.

---

## Firewall Configuration

I chose UFW, which stands for Uncomplicated Firewall.

UFW is a frontend that simplifies the management of firewall rules on Linux systems.

For the mandatory part of the project, the firewall is enabled and only SSH port `4242` is allowed.

Installation and Configuration

```bash
sudo apt install ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 4242/tcp
sudo ufw enable
```

```text
apt install ufw -> Installs UFW.
default deny incoming -> Blocks incoming connections by default.
default allow outgoing -> Allows outgoing connections by default.
allow 4242/tcp -> Allows incoming SSH traffic on TCP port 4242.
enable -> Enables the firewall.
```

Useful Commands:

```bash
sudo ufw status
sudo ufw status numbered
```

These commands display the firewall status and its active rules.

During Evaluation
A temporary rule may be added and removed to demonstrate firewall management:

```bash
sudo ufw allow 8080
sudo ufw status numbered
sudo ufw delete RULE_NUMBER
```

### UFW vs firewalld

UFW is simple and common on Debian and Ubuntu systems. It is easy to use for small server setups.

firewalld is common on Red Hat-based systems such as Rocky Linux. It supports zones and more dynamic firewall management, but it is more complex.

Since this VM uses Debian, I used UFW.

---

## Password Policy

The password policy is designed to improve account security by requiring passwords to meet minimum strength requirements and expire after a specific period.

The policy has two main parts:

1. Password aging.
2. Password complexity.

### Password aging

Password aging is configured in:

```text
/etc/login.defs
```

The required values are:

```text
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

Meaning:

```text
PASS_MAX_DAYS 30  -> Password expires after 30 days.
PASS_MIN_DAYS 2   -> Password cannot be changed again before 2 days.
PASS_WARN_AGE 7   -> Warning appears 7 days before expiration.
```

`/etc/login.defs` provides defaults for new accounts. The aging settings for existing accounts were also applied using `chage`.

Useful checks:

```bash
grep -E "PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE" /etc/login.defs
chage -l mobani-t
sudo chage -l root
sudo chage -M 30 -m 2 -W 7 mobani-t
sudo chage -M 30 -m 2 -W 7 root
```

### Password complexity

The `libpam-pwquality` package provides password quality checking through PAM.

Install it using:

```bash
sudo apt install libpam-pwquality
```

```text
/etc/pam.d/common-password
```

The PAM configuration in `/etc/pam.d/common-password` enables password-quality checks:

```text
password requisite pam_pwquality.so retry=3
```

The main rules are configured in `/etc/security/pwquality.conf`:

```ini
minlen = 10
difok = 7
maxrepeat = 3
dcredit = -1
ucredit = -1
lcredit = -1
usercheck = 1
enforcing = 1
enforce_for_root
```

- `retry=3`: up to three attempts to enter a valid new password.
- `minlen=10`: minimum password length (subject to libpwquality's length accounting).
- `difok=7`: required difference from the previous password when applicable.
- `maxrepeat=3`: restricts repeated consecutive characters.
- `dcredit=-1`, `ucredit=-1`, `lcredit=-1`: require a digit, uppercase letter, and lowercase letter.
- `usercheck=1`: enables a check against the account name.
- `enforcing=1` and `enforce_for_root`: enable enforcement, including quality rules for root.

The `difok` check depends on availability of the previous password; root password changes must still satisfy the other configured quality checks.

After configuring the password policy, the passwords of the existing regular user and root account were changed to apply the new policy.

Useful commands:

```bash
sudo passwd mobani-t
sudo passwd root
sudo passwd newuser
```

### Advantages and disadvantages of password policies

Advantages:

```text
- Reduces the use of weak and predictable passwords.
- Makes dictionary and brute-force attacks more difficult.
- Prevents passwords containing the username.
- Limits consecutive repeated characters.
- Requires regular password changes.
- Establishes a minimum password security standard.
```
Disadvantages:

```text
- Complex passwords can be harder to remember.
- Users may forget their passwords.
- Frequent expiration can encourage users to write passwords down or make predictable changes.
- Password management creates additional administrative work.
```

A password policy improves security, but it should also consider usability.

---

## Sudo Configuration

`sudo` allows an authorized regular user to execute commands with administrative privileges without logging in directly as root.

Examples:

```bash
sudo apt update
sudo service ssh status
sudo ufw status
```

Sudo is useful because it provides controlled access to administrative operations and records important actions.

### Verify sudo installation

```bash
which sudo
sudo --version
dpkg -l | grep sudo
```
Expected idea:

```text
/usr/bin/sudo
Sudo version ...
```
### Sudo rules

The custom sudo settings were stored in:

```text
/etc/sudoers.d/born2beroot
```

The configuration was checked using `sudo visudo -c`.

Configured rules:

```text
Defaults passwd_tries=3
Defaults badpass_message="Incorrect password. Please try again."
Defaults log_input
Defaults log_output
Defaults iolog_dir="/var/log/sudo/"
Defaults requiretty
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

These settings limit password retries, define a custom failure message, enable sudo I/O session recording in `/var/log/sudo/`, require a terminal, and use a restricted command search path.

`secure_path` helps reduce the risk of path hijacking by restricting the directories searched for commands executed through sudo.

Useful checks:

```bash
sudo visudo -c
sudo -l
sudo ls -l /etc/sudoers.d/born2beroot
sudo stat -c "%a %U %G %n" /etc/sudoers.d/born2beroot
sudo ls -la /var/log/sudo
sudo sudoreplay -l -d /var/log/sudo
```

The custom sudoers file was configured with restrictive permissions (`0440`) and root ownership; verify with `stat`.

To verify logging, run a sudo command and then check the log again:

```bash
sudo ls /root
sudo sudoreplay -l -d /var/log/sudo
```

---

## AppArmor

AppArmor is a Linux security module. It restricts programs using path-based security profiles.

It limits what a program can access even if the program is running with high privileges.

Useful commands:

```bash
sudo systemctl status apparmor
sudo aa-status
```

Common AppArmor profile status:

```text
enforce     -> rules are active and violations are blocked
complain    -> violations are logged but not blocked
unconfined  -> profile exists but is not actively restricting the program
```


### AppArmor vs SELinux

AppArmor uses path-based profiles and is common on Debian and Ubuntu systems. It is generally easier to understand.

SELinux uses label-based mandatory access control and is common on Red Hat-based systems such as Rocky Linux. It is powerful but more complex.

Since this VM uses Debian, AppArmor must be running at startup.

---

## Monitoring Script

A Bash monitoring script named `monitoring.sh` was created at:

```text
/usr/local/bin/monitoring.sh
```

It displays system information to all logged-in terminals using `wall`.

The script displays:

```text
- Operating system architecture and kernel version.
- Number of physical processors.
- Number of virtual processors.
- RAM usage and percentage.
- Disk usage and percentage.
- CPU load percentage.
- Last boot time.
- LVM use.
- Number of established TCP connections.
- Number of logged-in user sessions.
- IPv4 address and MAC address.
- Number of sudo commands executed.
```

The script is executable:

```bash
sudo chmod +x /usr/local/bin/monitoring.sh
ls -l /usr/local/bin/monitoring.sh
```

Manual execution:

```bash
sudo /usr/local/bin/monitoring.sh
```

### Script command explanations

**Architecture and kernel**

```bash
uname -a
```

Shows architecture, hostname, kernel version, and system information.

**Physical processors**

```bash
lscpu -p=Socket,Core | grep -v "^#" | sort -u | wc -l
```

Counts unique physical CPU IDs reported by `/proc/cpuinfo`.

**Virtual processors**

```bash
nproc --all
```

Counts processor entries reported by `/proc/cpuinfo`, which represent the virtual CPUs visible to the VM.

**RAM**

```bash
free --mega
```
Shows RAM information in MB.

**Disk usage**

```bash
df -m
```

Shows disk usage in MB.

**CPU Load**

```bash
vmstat 1 2
```

Gets CPU statistics with two measurements. The second line is used to calculate CPU usage as `100 - idle`.

**Last boot**

```bash
who -b
```
Shows the last boot time.

**LVM use**

```bash
lsblk | grep -c "lvm"
```

Checks whether LVM appears in the block device layout.

**TCP connections**

```bash
ss -Htan state established | wc -l
```

Displays TCP sockets. The script counts connections in the `ESTAB` state.

**Logged-in users**

```bash
who | wc -l
```

Counts active logged-in user sessions. It counts sessions, not unique usernames.

**IPv4 address**

```bash
ip -4 -o addr show scope global
```

Displays global IPv4 addresses assigned to network interfaces.

**MAC address**

```bash
ip link
```

Displays network interface information, including MAC addresses.

**Sudo command count**

```bash
journalctl _COMM=sudo --no-pager | grep -c 'COMMAND='
```

Counts sudo commands from the journal logs.

**Broadcast message**

```bash
wall "message"
```

Broadcasts a message to logged-in terminals.

---

## Cron Configuration

Cron is a time-based job scheduler.

Root's crontab runs `monitoring.sh` every 10 minutes.

Open the root crontab:

```bash
sudo crontab -e
```

The following entries are configured:

```cron
@reboot /usr/local/bin/monitoring.sh
*/10 * * * * /usr/local/bin/monitoring.sh
```

Meaning:

```text
@reboot  -> Runs the monitoring script when the system starts.
*/10     -> Every 10 minutes.
*        -> Every hour.
*        -> Every day of the month.
*        -> Every month.
*        -> Every day of the week.
```

The script calls `wall` itself. On this Debian version, broadcasts were verified on the local VirtualBox console; delivery to SSH pseudo-terminals was not reliable during testing.

Check root crontab:

```bash
sudo crontab -l
```

To interrupt the script without modifying it:

```bash
sudo service cron stop
```

To start it again:

```bash
sudo service cron start
```

Check cron status:

```bash
sudo service cron status
```

---
## TTY, PTS, and User Sessions

`tty` means a terminal directly attached to the machine, for example the VirtualBox console.

`pts` means pseudo-terminal, usually created by SSH or a terminal emulator.

Useful commands:

```bash
who
w
tty
users
```

`User log` in the monitoring script means active logged-in sessions, not unique users. If the same user is logged in through `tty1` and SSH, the username may appear more than once.


---

## Resources

The following resources were used to understand Linux system administration, security configuration, networking, and virtualization.

### Official Documentation

- Debian Documentation: https://www.debian.org/doc/
- Debian Wiki: https://wiki.debian.org/
- Debian Administrator's Handbook: https://debian-handbook.info/
- Rocky Linux Documentation: https://docs.rockylinux.org/
- OpenSSH Manual: https://man.openbsd.org/sshd_config
- UFW Manual: https://manpages.debian.org/ufw
- Sudo Documentation: https://www.sudo.ws/docs/
- Linux PAM Documentation: https://www.linux-pam.org/Linux-PAM-html/
- AppArmor Documentation: https://apparmor.net/
- VirtualBox Manual: https://www.virtualbox.org/manual/
- UTM Documentation: https://docs.getutm.app/

### Useful Linux Manual Pages

```bash
man ssh
man sshd_config
man sudo
man sudoers
man ufw
man crontab
man passwd
man chage
man lsblk
man wall
man systemctl
man pam_pwquality
```

### AI Usage

AI was used as a learning assistant during this project.

It was used for:

- Understanding the subject requirements.
- Explaining Linux system administration concepts.
- Clarifying Debian vs Rocky Linux.
- Clarifying AppArmor vs SELinux.
- Clarifying UFW vs firewalld.
- Clarifying VirtualBox vs UTM.
- Understanding LVM, disk encryption, SSH, sudo, cron, PAM, and password policies.
- Reviewing commands and configuration checks.
- Structuring and improving this README.

AI was not used as a replacement for understanding the project. Configuration steps were applied and tested manually inside the virtual machine.

---

## Final Notes

Born2beRoot provided practical experience with Linux system administration and basic server security.

The project helped me understand how different components work together to build a secure and manageable server.

The main parts of my implementation are:

```text
- Minimal Debian 13 (Trixie) installation.
- No graphical desktop environment.
- Encrypted LVM.
- SSH configured on port 4242.
- Root SSH login disabled.
- UFW firewall enabled.
- Password aging and complexity policies.
- Sudo security rules and logging.
- AppArmor enabled at startup.
- Bash monitoring script.
- Cron execution at startup and every 10 minutes.
- Regular user and required group configuration.
- SHA1 virtual disk signature.
```
