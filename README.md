Basic Enterprise Software Installer

This is a Python-based software installer for Debian/Ubuntu systems. It reads from a `packages.txt` file and installs each package with rollback support, progress bars, and interactive options.

✅ Features

- Bulk install all packages from `packages.txt`
- Interactive install mode (choose packages one by one)
- Rollback feature to uninstall and restore config files
- Automatically runs `apt update && upgrade`
- Progress bars and color output for better experience

🧾 Requirements

- Python 3
- `pip install colorama tqdm`
- Must run as `sudo` (root)

📂 Files

- `installer.py` – Main Python script
- `packages.txt` – List of packages to install
- `logs/install.log` – Log file (auto-created)
- `/tmp/rollback_packages.txt` – Used for rollback tracking
- `/tmp/package_backups/` – Stores backups of config files

▶️ How to Use

1. Clone the repo:

bash

git clone https://github.com/Jurthistar/enterprise-installer.git

cd enterprise-installer

Install dependencies:

bash
pip install colorama tqdm

Run the installer:

bash
sudo python3 installer.py
Choose an option from the menu:

1 – Bulk install

2 – Interactive install

3 – Rollback

4 – Exit

📝 Example packages.txt

curl,
git,
vim,
htop,
nmap,
net-tools,
build-essential,
python3,
python3-pip,
apache2,
nginx,
fail2ban,
docker.io,
docker-compose.
after cloning you can add yours....

📦 Build the .deb Package (Optional)

bash
dpkg-deb --build enterprise-installer-1.0

Then install:

bash
sudo dpkg -i enterprise-installer-1.0.deb
