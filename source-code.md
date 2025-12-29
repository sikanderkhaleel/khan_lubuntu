@echo off

:: ============================================
:: AUTO ELEVATE TO ADMINISTRATOR
:: ============================================
net session >nul 2>&1
if %errorlevel% neq 0 (
    echo Requesting Administrator privileges...
    powershell -Command "Start-Process '%~f0' -Verb RunAs"
    exit /b
)

title Khan Lubuntu - Complete Auto Setup
setlocal enabledelayedexpansion

:: Always work in the folder where .bat exists
cd /d "%~dp0"

:: ============================================
:: PREVENT SLEEP FROM THE START
:: ============================================
echo Preventing system sleep during setup...
powercfg -change -standby-timeout-ac 0
powercfg -change -standby-timeout-dc 0
powercfg -change -hibernate-timeout-ac 0
powercfg -change -hibernate-timeout-dc 0
powercfg -change -monitor-timeout-ac 0
powercfg -change -monitor-timeout-dc 0
echo [OK] Sleep disabled!
echo.

:: ============================================
:: CONFIGURATION
:: ============================================
set "VBOX_VERSION=7.2.4"
set "SAVE_PATH=%USERPROFILE%\Desktop\VBoxSetup.exe"
set "VBOXMANAGE=C:\Program Files\Oracle\VirtualBox\VBoxManage.exe"

:: VM Configuration
set "VM_NAME=KhanLubuntu"
set "VDI_NAME=KhanLubuntu24.04.vdi"
set "VDI_PATH=%~dp0%VDI_NAME%"
set "VDI_URL=https://archive.org/download/khanlubuntu24.04/KhanLubuntu24.04.vdi"
set "VM_FOLDER=%USERPROFILE%\VirtualBox VMs\%VM_NAME%"
set "OS_TYPE=Lubuntu_64"

echo ==========================================
echo   Khan Lubuntu - Complete Auto Setup
echo ==========================================
echo.
echo Working directory: %cd%
echo.

:: ============================================
:: CHECK VIRTUALIZATION SUPPORT (EARLY CHECK)
:: ============================================
echo Checking if virtualization is enabled...
echo.

powershell -Command "if ((Get-WmiObject Win32_Processor).VirtualizationFirmwareEnabled -eq $true) { exit 0 } else { exit 1 }"

if !errorlevel! neq 0 (
    echo.
    echo ==========================================
    echo   [ERROR] Virtualization is DISABLED!
    echo ==========================================
    echo.
    echo Your computer has virtualization turned off in BIOS.
    echo VirtualBox needs this to run virtual machines.
    echo.
    echo Please follow these steps to enable it:
    echo.
    echo   1. Restart your computer
    echo   2. Press F2, F10, F12, DEL or ESC during startup
    echo      ^(depends on your computer brand^)
    echo   3. Find "Virtualization" or "VT-x" or "AMD-V" setting
    echo   4. Enable it
    echo   5. Save and exit BIOS
    echo   6. Run this script again
    echo.
    echo Common BIOS keys by brand:
    echo   HP: F10
    echo   Dell: F2
    echo   Lenovo: F1 or F2
    echo   Asus: F2 or DEL
    echo   Acer: F2 or DEL
    echo.
    goto :restore_sleep
)

echo [OK] Virtualization is enabled!
echo.

:: ============================================
:: PART 1: VDI DOWNLOAD
:: ============================================
echo ==========================================
echo   PART 1: VDI Download
echo ==========================================
echo.

:: Check if VDI already exists
if exist "%VDI_NAME%" (
    echo [FOUND] VDI already exists. Skipping download...
    for %%A in ("%VDI_NAME%") do echo File size: %%~zA bytes
    echo.
    goto :vbox_setup
)

echo VDI not found. Starting download...
echo.

:: ---------------------------------
:: STEP 1: GET ARIA2 (IF NOT EXISTS)
:: ---------------------------------
if exist "aria2c.exe" (
    echo [FOUND] aria2 already present.
) else (
    echo aria2 not found. Downloading...

    powershell -Command ^
    "(New-Object Net.WebClient).DownloadFile('https://github.com/aria2/aria2/releases/download/release-1.37.0/aria2-1.37.0-win-64bit-build1.zip','aria2.zip')"

    if not exist "aria2.zip" (
        echo [ERROR] Failed to download aria2.
        goto :restore_sleep
    )

    powershell -Command "Expand-Archive -Force aria2.zip aria2tmp"

    copy "aria2tmp\aria2-1.37.0-win-64bit-build1\aria2c.exe" "aria2c.exe" >nul

    rd /s /q aria2tmp
    del aria2.zip

    echo [OK] aria2 installed successfully.
)

echo.

:: ---------------------------------
:: STEP 2: DOWNLOAD VDI (FAST MODE)
:: ---------------------------------
echo ==========================================
echo   Downloading VDI from Archive.org
echo ==========================================
echo.
echo Using 16 connections for FAST download!
echo File size: ~8GB
echo.
echo This WILL take time. Do NOT close the window.
echo.

aria2c.exe ^
 --file-allocation=none ^
 --continue=true ^
 -x 16 -s 16 ^
 -k 1M ^
 --max-connection-per-server=16 ^
 --min-split-size=1M ^
 --auto-file-renaming=false ^
 --console-log-level=warn ^
 -o "%VDI_NAME%" ^
 "%VDI_URL%"

echo.

:: ---------------------------------
:: STEP 3: VERIFY VDI FILE
:: ---------------------------------
if exist "%VDI_NAME%" (
    echo ==========================================
    echo   [SUCCESS] VDI Download Complete!
    echo ==========================================
    for %%A in ("%VDI_NAME%") do echo File size: %%~zA bytes
    echo.
) else (
    echo [ERROR] VDI Download failed!
    goto :restore_sleep
)

:: ============================================
:: PART 2: VIRTUALBOX SETUP
:: ============================================
:vbox_setup
echo.
echo ==========================================
echo   PART 2: VirtualBox Setup
echo ==========================================
echo.

:: Check if VirtualBox already installed
echo Checking if VirtualBox is already installed...
echo.

if exist "%VBOXMANAGE%" (
    echo [ALREADY INSTALLED] VirtualBox found!
    echo.
    "%VBOXMANAGE%" --version
    echo.
    goto :verify_vdi
)

echo VirtualBox not found. Proceeding with setup...
echo.

:: Check if installer already downloaded
echo Checking if installer already downloaded...
echo.

if exist "%SAVE_PATH%" (
    echo [FOUND] Installer already exists. Skipping download...
    for %%A in ("%SAVE_PATH%") do echo File size: %%~zA bytes
    echo.
    goto :install_vbox
)

echo Installer not found. Starting download...
echo.

:: ---------------------------------
:: GET DYNAMIC BUILD NUMBER
:: ---------------------------------
echo ==========================================
echo   Finding Latest Build Number
echo ==========================================
echo.

echo Fetching from Oracle servers...

for /f "delims=" %%a in ('powershell -Command "$page = Invoke-WebRequest -Uri 'https://download.virtualbox.org/virtualbox/%VBOX_VERSION%/' -UseBasicParsing; if ($page.Content -match 'VirtualBox-%VBOX_VERSION%-(\d+)-Win\.exe') { $matches[1] }"') do (
    set "BUILD_NUMBER=%%a"
)

if not defined BUILD_NUMBER (
    echo [ERROR] Could not find build number!
    goto :restore_sleep
)

echo [FOUND] Build number: %BUILD_NUMBER%
echo.

set "DOWNLOAD_URL=https://download.virtualbox.org/virtualbox/%VBOX_VERSION%/VirtualBox-%VBOX_VERSION%-%BUILD_NUMBER%-Win.exe"

echo Download URL: %DOWNLOAD_URL%
echo.

:: ---------------------------------
:: DOWNLOAD VIRTUALBOX
:: ---------------------------------
echo ==========================================
echo   Downloading VirtualBox %VBOX_VERSION%
echo ==========================================
echo.
echo Saving to: %SAVE_PATH%
echo Please wait...
echo.

bitsadmin /transfer "VBoxDownload" /download /priority high "%DOWNLOAD_URL%" "%SAVE_PATH%"

echo.
if exist "%SAVE_PATH%" (
    echo ==========================================
    echo   [SUCCESS] Download Complete!
    echo ==========================================
    echo File saved to: %SAVE_PATH%
    for %%A in ("%SAVE_PATH%") do echo File size: %%~zA bytes
) else (
    echo [FAILED] Download failed
    goto :restore_sleep
)

:: ---------------------------------
:: INSTALL VIRTUALBOX
:: ---------------------------------
:install_vbox
echo.
set "INSTALLER=%USERPROFILE%\Desktop\VBoxSetup.exe"

echo ==========================================
echo   Installing VirtualBox Silently
echo ==========================================
echo.

if not exist "%INSTALLER%" (
    echo [ERROR] Installer not found at: %INSTALLER%
    goto :restore_sleep
)

echo Installer found!
echo.
echo Installing to default location...
echo Accepting all prompts automatically...
echo This may take 2-5 minutes...
echo DO NOT close this window!
echo.

"%INSTALLER%" --silent --ignore-reboot --msiparams ALLUSERS=1,REBOOT=ReallySuppress

echo.
echo Waiting for installation to finish...
timeout /t 20 /nobreak

echo.
echo ==========================================
echo   Checking Installation
echo ==========================================
echo.

if exist "%VBOXMANAGE%" (
    echo ==========================================
    echo   [SUCCESS] VirtualBox installed!
    echo ==========================================
    echo.
    echo Location: C:\Program Files\Oracle\VirtualBox
    echo.
    "%VBOXMANAGE%" --version
    echo.

    del "%INSTALLER%" >nul 2>&1
    echo Installer deleted to save space.
) else (
    echo ==========================================
    echo   [FAILED] Installation may have failed
    echo ==========================================
    goto :restore_sleep
)

:: ============================================
:: VERIFY VDI INTEGRITY
:: ============================================
:verify_vdi
echo.
echo ==========================================
echo   Verifying VDI Integrity
echo ==========================================
echo.

echo Checking if VDI file is valid...

"%VBOXMANAGE%" showmediuminfo disk "%VDI_PATH%" >nul 2>&1

if !errorlevel! neq 0 (
    echo.
    echo ==========================================
    echo   [ERROR] VDI file is corrupted!
    echo ==========================================
    echo.
    echo The VDI file appears to be 7-8GB in size but is actually
    echo incomplete or corrupted due to an interrupted download.
    echo.
    echo Deleting corrupted VDI file...
    del "%VDI_PATH%" >nul 2>&1
    del "%VDI_PATH%.aria2" >nul 2>&1
    echo [OK] Corrupted files deleted.
    echo.
    echo Please run this script again to re-download the VDI.
    echo.
    goto :restore_sleep
)

echo [OK] VDI file is valid!
echo.

:: ============================================
:: PART 3: CREATE VIRTUAL MACHINE
:: ============================================
:create_vm
echo.
echo ==========================================
echo   PART 3: Virtual Machine Setup
echo ==========================================
echo.

:: Check if VDI exists
echo Checking for VDI file...
echo.

if not exist "%VDI_PATH%" (
    echo [ERROR] VDI file not found!
    echo.
    echo Please make sure "%VDI_NAME%" is in the same folder as this script.
    echo.
    goto :restore_sleep
)

echo [FOUND] VDI file exists!
for %%A in ("%VDI_PATH%") do echo File size: %%~zA bytes
echo.

:: Check if VM already exists
echo Checking for existing VM...
echo.

"%VBOXMANAGE%" showvminfo "%VM_NAME%" >nul 2>&1
if %errorlevel% equ 0 (
    echo [FOUND] VM "%VM_NAME%" exists.
    echo.
    
    :: Verify VDI exists in VM folder
    echo Verifying VM integrity...
    
    if exist "%VM_FOLDER%\%VDI_NAME%" (
        :: Use PowerShell for large file size comparison (greater than 7GB)
        powershell -Command "if ((Get-Item '%VM_FOLDER%\%VDI_NAME%').Length -gt 7000000000) { exit 0 } else { exit 1 }"
        
        if !errorlevel! equ 0 (
            :: Also verify with VBoxManage
            "%VBOXMANAGE%" showmediuminfo disk "%VM_FOLDER%\%VDI_NAME%" >nul 2>&1
            if !errorlevel! equ 0 (
                echo [OK] VM VDI is valid and complete.
                echo.
                goto :start_vm
            )
        )
        
        echo.
        echo ==========================================
        echo   [ERROR] VM VDI is incomplete or corrupted!
        echo ==========================================
        echo.
        echo The VDI file may appear to be 7-8GB in size but is actually
        echo incomplete or corrupted due to an interrupted process.
        echo.
        echo Please follow these steps:
        echo.
        echo   1. Open VirtualBox
        echo   2. Right-click on "%VM_NAME%" and select "Remove"
        echo   3. Choose "Delete all files"
        echo.
        echo   If folder still exists, manually delete:
        echo   %VM_FOLDER%
        echo.
        echo   Also delete the VDI in the script folder:
        echo   %VDI_PATH%
        echo.
        echo Then run this script again.
        echo.
        goto :restore_sleep
    ) else (
        echo.
        echo ==========================================
        echo   [ERROR] VDI not found in VM folder!
        echo ==========================================
        echo.
        echo The VM exists but VDI file is missing.
        echo.
        echo Please follow these steps:
        echo.
        echo   1. Open VirtualBox
        echo   2. Right-click on "%VM_NAME%" and select "Remove"
        echo   3. Choose "Delete all files"
        echo.
        echo   If folder still exists, manually delete:
        echo   %VM_FOLDER%
        echo.
        echo Then run this script again.
        echo.
        goto :restore_sleep
    )
)

echo VM does not exist. Creating new VM...
echo.

:: ---------------------------------
:: CREATE VM
:: ---------------------------------
echo ==========================================
echo   Creating Virtual Machine
echo ==========================================
echo.

if not exist "%VM_FOLDER%" mkdir "%VM_FOLDER%"

echo Creating VM: %VM_NAME% (%OS_TYPE%)
"%VBOXMANAGE%" createvm --name "%VM_NAME%" --ostype "%OS_TYPE%" --register

if %errorlevel% neq 0 (
    echo.
    echo ==========================================
    echo   [ERROR] Failed to create VM!
    echo ==========================================
    echo.
    echo A previous incomplete installation may exist.
    echo.
    echo Please follow these steps:
    echo.
    echo   1. Open VirtualBox
    echo   2. If "%VM_NAME%" exists, right-click and select "Remove"
    echo   3. Choose "Delete all files"
    echo.
    echo   Also manually delete this folder if it exists:
    echo   %VM_FOLDER%
    echo.
    echo Then run this script again.
    echo.
    goto :restore_sleep
)

echo [OK] VM created!
echo.

:: ---------------------------------
:: CONFIGURE VM
:: ---------------------------------
echo ==========================================
echo   Configuring VM Settings
echo ==========================================
echo.

echo Setting RAM: 2048 MB
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --memory 2048

echo Setting CPU: 2 cores
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --cpus 2

echo Setting Video Memory: 50 MB
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --vram 50

echo Setting Graphics: VMSVGA
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --graphicscontroller vmsvga

echo Setting Network: NAT
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --nic1 nat

echo Setting Boot Order
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --boot1 disk --boot2 dvd --boot3 none --boot4 none

echo Enabling Clipboard: Bidirectional
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --clipboard-mode bidirectional

echo Enabling Drag and Drop: Bidirectional
"%VBOXMANAGE%" modifyvm "%VM_NAME%" --draganddrop bidirectional

echo.
echo [OK] VM configured!
echo.

:: ---------------------------------
:: ATTACH VDI
:: ---------------------------------
echo ==========================================
echo   Attaching VDI to VM
echo ==========================================
echo.

echo Cloning VDI with new UUID...
"%VBOXMANAGE%" clonemedium disk "%VDI_PATH%" "%VM_FOLDER%\%VDI_NAME%" --format VDI

if %errorlevel% neq 0 (
    echo.
    echo ==========================================
    echo   [ERROR] Failed to clone VDI!
    echo ==========================================
    echo.
    echo Cleaning up incomplete files...
    
    :: Delete incomplete VDI in VM folder
    del "%VM_FOLDER%\%VDI_NAME%" >nul 2>&1
    
    :: Remove the VM we just created
    "%VBOXMANAGE%" unregistervm "%VM_NAME%" --delete >nul 2>&1
    
    :: Try to remove folder
    rd /s /q "%VM_FOLDER%" >nul 2>&1
    
    echo.
    echo ==========================================
    echo   IMPORTANT NOTICE
    echo ==========================================
    echo.
    echo If the problem persists, it may be due to an incomplete
    echo download of "KhanLubuntu24.04.vdi".
    echo.
    echo The file size may appear to be 7-8GB, but an error may
    echo have occurred during the download process.
    echo.
    echo Please follow these steps:
    echo.
    echo   1. Navigate to the folder containing this script
    echo   2. Delete the file "KhanLubuntu24.04.vdi"
    echo   3. Also delete "KhanLubuntu24.04.vdi.aria2" if it exists
    echo   4. Run this script again
    echo.
    echo ==========================================
    echo.
    goto :restore_sleep
)

echo [OK] VDI cloned with new UUID!
echo.

echo Adding SATA Controller...
"%VBOXMANAGE%" storagectl "%VM_NAME%" --name "SATA Controller" --add sata --controller IntelAhci

echo Attaching VDI...
"%VBOXMANAGE%" storageattach "%VM_NAME%" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "%VM_FOLDER%\%VDI_NAME%"

if %errorlevel% neq 0 (
    echo.
    echo ==========================================
    echo   [ERROR] Failed to attach VDI!
    echo ==========================================
    echo.
    
    :: Cleanup
    "%VBOXMANAGE%" unregistervm "%VM_NAME%" --delete >nul 2>&1
    rd /s /q "%VM_FOLDER%" >nul 2>&1
    
    echo ==========================================
    echo   IMPORTANT NOTICE
    echo ==========================================
    echo.
    echo If the problem persists, it may be due to an incomplete
    echo download of "KhanLubuntu24.04.vdi".
    echo.
    echo The file size may appear to be 7-8GB, but an error may
    echo have occurred during the download process.
    echo.
    echo Please follow these steps:
    echo.
    echo   1. Navigate to the folder containing this script
    echo   2. Delete the file "KhanLubuntu24.04.vdi"
    echo   3. Also delete "KhanLubuntu24.04.vdi.aria2" if it exists
    echo   4. Run this script again
    echo.
    echo ==========================================
    echo.
    goto :restore_sleep
)

echo [OK] VDI attached!
echo.

:: ============================================
:: PART 4: START VM
:: ============================================
:start_vm
echo.
echo ==========================================
echo   Starting Virtual Machine
echo ==========================================
echo.

echo Launching %VM_NAME%...
echo.

"%VBOXMANAGE%" startvm "%VM_NAME%"

if %errorlevel% equ 0 (
    echo.
    echo ==========================================
    echo ==========================================
    echo   [SUCCESS] ALL DONE!
    echo ==========================================
    echo ==========================================
    echo.
    echo   VM Name: %VM_NAME%
    echo   VDI: %VDI_NAME%
    echo.
    echo   Login Credentials:
    echo   Username: student
    echo   Password: 123
    echo.
    echo   Your VM is now running!
    echo.
) else (
    echo.
    echo ==========================================
    echo   [ERROR] Failed to start VM!
    echo ==========================================
    echo.
    echo This might be because virtualization is disabled.
    echo.
    echo Please follow these steps:
    echo.
    echo   1. Restart your computer
    echo   2. Enter BIOS ^(press F2, F10, DEL during startup^)
    echo   3. Find "Virtualization" or "VT-x" or "AMD-V"
    echo   4. Enable it
    echo   5. Save and exit BIOS
    echo   6. Run this script again
    echo.
    echo Common BIOS keys:
    echo   HP: F10    Dell: F2    Lenovo: F1/F2
    echo   Asus: F2/DEL    Acer: F2/DEL
    echo.
    echo ==========================================
    echo   IMPORTANT NOTICE
    echo ==========================================
    echo.
    echo If the problem persists, it may be due to an incomplete
    echo download of "KhanLubuntu24.04.vdi".
    echo.
    echo The file size may appear to be 7-8GB, but an error may
    echo have occurred during the download process.
    echo.
    echo Please follow these steps:
    echo.
    echo   1. Navigate to the folder containing this script
    echo   2. Delete the file "KhanLubuntu24.04.vdi"
    echo   3. Also delete "KhanLubuntu24.04.vdi.aria2" if it exists
    echo   4. Delete the VM folder:
    echo      %VM_FOLDER%
    echo   5. Run this script again
    echo.
    echo ==========================================
    echo.
)

:: ============================================
:: RESTORE SLEEP SETTINGS
:: ============================================
:restore_sleep
echo.
echo Restoring system sleep settings...
powercfg -change -standby-timeout-ac 15
powercfg -change -standby-timeout-dc 10
powercfg -change -hibernate-timeout-ac 0
powercfg -change -hibernate-timeout-dc 0
powercfg -change -monitor-timeout-ac 10
powercfg -change -monitor-timeout-dc 5
echo [OK] Sleep settings restored!
echo.

pause
