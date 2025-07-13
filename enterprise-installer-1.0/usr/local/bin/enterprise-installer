#!/usr/bin/env python3

import os
import subprocess
import datetime
import shutil
import threading
from concurrent.futures import ThreadPoolExecutor
from colorama import Fore, Style, init
from tqdm import tqdm

init(autoreset=True)  # Enable colorama for Windows compatibility

PACKAGE_LIST = "packages.txt"
ROLLBACK_FILE = "/tmp/rollback_packages.txt"
BACKUP_DIR = "/tmp/package_backups"
LOG_FILE = "logs/install.log"

os.makedirs("logs", exist_ok=True)
os.makedirs(BACKUP_DIR, exist_ok=True)

MAX_PARALLEL = 4  # Number of parallel installs

def log(message):
    with open(LOG_FILE, "a") as f:
        f.write(f"{datetime.datetime.now()} | {message}\n")
    print(message)

def is_root():
    return os.geteuid() == 0

def update_system():
    print(Fore.BLUE + "Updating system (apt update && upgrade)...")
    subprocess.run(["apt", "update", "-y"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    subprocess.run(["apt", "upgrade", "-y"], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    print(Fore.GREEN + "System updated.\n")

def package_installed(pkg):
    result = subprocess.run(["dpkg", "-s", pkg], stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    return result.returncode == 0

def backup_config(pkg):
    etc_path = f"/etc/{pkg}"
    if os.path.exists(etc_path):
        backup_path = os.path.join(BACKUP_DIR, f"{pkg}.bak")
        shutil.copytree(etc_path, backup_path, dirs_exist_ok=True)
        log(Fore.YELLOW + f"Backed up config for {pkg}")

def restore_config(pkg):
    backup_path = os.path.join(BACKUP_DIR, f"{pkg}.bak")
    etc_path = f"/etc/{pkg}"
    if os.path.exists(backup_path):
        shutil.copytree(backup_path, etc_path, dirs_exist_ok=True)
        log(Fore.YELLOW + f"Restored config for {pkg}")

def install_package(pkg):
    if package_installed(pkg):
        log(Fore.CYAN + f"[✓] Already installed: {pkg}")
        return True

    backup_config(pkg)
    result = subprocess.run(["apt", "install", "-y", pkg], capture_output=True, text=True)
    if result.returncode == 0:
        log(Fore.GREEN + f"[✓] Installed: {pkg}")
        with open(ROLLBACK_FILE, "a") as f:
            f.write(f"{pkg}\n")
        return True
    else:
        log(Fore.RED + f"[✗] Failed: {pkg} — {result.stderr.strip()}")
        return False

def uninstall_packages():
    print(Fore.YELLOW + "Rolling back previously installed packages...")
    if not os.path.exists(ROLLBACK_FILE):
        print(Fore.RED + "No rollback file found.")
        return

    with open(ROLLBACK_FILE) as f:
        for pkg in f:
            pkg = pkg.strip()
            if not pkg:
                continue
            subprocess.run(["apt", "remove", "-y", pkg], stdout=subprocess.DEVNULL)
            restore_config(pkg)
            log(Fore.YELLOW + f"Rolled back: {pkg}")

def read_packages():
    packages = []
    try:
        with open(PACKAGE_LIST, "r") as f:
            for line in f:
                line = line.strip()
                if line and not line.startswith("#"):
                    packages.append(line)
    except FileNotFoundError:
        log(Fore.RED + "packages.txt not found!")
        exit(1)
    return packages

def bulk_install():
    update_system()
    packages = read_packages()
    print(Fore.BLUE + f"\nInstalling {len(packages)} packages in parallel...\n")
    with tqdm(total=len(packages), desc="Installing", colour="green") as pbar:
        def task(pkg):
            install_package(pkg)
            pbar.update(1)

        with ThreadPoolExecutor(max_workers=MAX_PARALLEL) as executor:
            for pkg in packages:
                executor.submit(task, pkg)

def interactive_install():
    update_system()
    for pkg in read_packages():
        if package_installed(pkg):
            print(Fore.CYAN + f"{pkg} is already installed.")
            continue
        choice = input(Fore.YELLOW + f"Install {pkg}? (y/n): ").strip().lower()
        if choice == "y":
            success = install_package(pkg)
            if not success:
                rollback = input(Fore.RED + "Install failed. Rollback previous installs? (y/n): ").strip().lower()
                if rollback == "y":
                    uninstall_packages()
                break

def menu():
    print(Fore.YELLOW + """
=== Enterprise Software Installer ===
1. Bulk install (auto install all)
2. Interactive install (choose one-by-one)
3. Rollback (uninstall installed + restore configs)
4. Exit
""")
    return input("Select an option [1-4]: ")

def main():
    if not is_root():
        print(Fore.RED + "Please run as root (sudo).")
        exit(1)

    while True:
        choice = menu()
        if choice == "1":
            bulk_install()
        elif choice == "2":
            interactive_install()
        elif choice == "3":
            uninstall_packages()
        elif choice == "4":
            print("Exiting...")
            break
        else:
            print("Invalid choice.")

if __name__ == "__main__":
    main()
