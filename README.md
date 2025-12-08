
# Windows Live USB Setup <img align="right" src="https://visitor-badge.laobi.icu/badge?page_id=iamthemag.Cisco-Packet-Tracer-Linux" />

Learn how to create a fully functional Live Windows USB that runs Windows directly from an external USB drive, without using third-party tools like WinToUSB. With this method, you can install Windows on a USB drive and seamlessly use it across both PCs and Macs, without altering the internal system. 

This allows you to work on the same projects and files on different machines, eliminating the need for file transfers between devices. Whether you're on a Windows PC or a Mac, your external drive will boot directly into your personalized Windows environment, enabling a smooth, portable computing experience.


## Author

- [@iamthemag](https://www.github.com/iamthemag)


## Appendix
### Windows 7:

- Microsoft officially ended support for Windows 7 on January 14, 2020. This means that the operating system no longer receives security updates, bug fixes, or technical support from Microsoft.

- After this date, PCs running Windows 7 are more vulnerable to security risks, as they will not receive patches for newly discovered vulnerabilities. Microsoft recommends upgrading to a newer version of Windows, like Windows 10 or 11, for continued security and feature updates.

### Windows 8:

- Windows 8 reached the end of mainstream support on January 9, 2018, meaning it stopped receiving new features and non-security updates.

- The end of extended support, which includes security updates, was on January 10, 2023. Like Windows 7, after this date, Windows 8 no longer receives any updates, making systems that still run it more susceptible to security risks. Microsoft encourages upgrading to a newer operating system for improved security and functionality.

### Note:
Both Windows 7 and 8 can still be used, but continued use is not recommended due to the lack of security updates and the increasing risk of vulnerabilities

## ISO Files
- [Windows 7 ISO](https://archive.org/details/windows-7-sp0-sp1-msdn-iso-files-en-de-ru-tr-x86-x64)
- [Windows 8 x64 ISO](https://archive.org/details/windows-8-x-64)
- [Windows 8.1 x64 & x86 ISO](https://archive.org/details/win-8.1-english-x-64_20211019)
 - [Windows 10 ISO](https://www.microsoft.com/en-in/software-download/windows10ISO)
 - [Windows 11 ISO](https://www.microsoft.com/en-in/software-download/windows11)
 




## Installation
First, download your desired ISO file and extract it. For example take Windows 10.

### Step 1: Prepare the Flash Drive

1. Open Run by pressing `Windows key + R`.
2. Open Command Prompt as Administrator by typing `cmd` and pressing `Ctrl + Shift + Enter`.
3. In the Command Prompt, type the following commands to prepare your USB flash drive:
![Dispart Image](https://www.seagate.com/content/dam/seagate/migrated-assets/www-content/support-content/knowledge-base/images/ka03A000000mKf2QAE__5.jpg)
   ```bash
   diskpart
   list disk
   select disk <disk no.>  # Select your flash drive
   clean
   create partition primary
   format fs=exfat quick    # Use 'format fs=ntfs quick' for PC
   assign 
   ```


4. Once the drive is formatted, extract the Windows 10 ISO and copy all files to the flash drive.

### Step 2: Install Windows on External Drive or External USB

1. Connect the flash drive to your PC or Mac.
2. Boot from the USB flash drive by selecting it from the boot menu.
3. On the installation screen, you may encounter errors like:
![IEEE 1394 port Error](https://www.easeus.com/images/en/screenshot/partition-manager/windows-cannot-be-installed-to-this-disk-connected-through-usb-port.png)

4. To bypass these errors, press `Shift + F10` to open a Command Prompt.

### Step 3: Set Up Partitions on the External SSD/HDD or USB

![Windows Partitions](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh8R53zV6HOhwSG6uwb9Hd7w7ASepOs_ACNE32HuACtPzEnntDVJWaZNE917BxXfMwfCsDUeXfg-SNlERqB6A-98_Q3v3frQz9yCUSzJY_4CGgZ3Of5smKRNPCWXnBMlYMj5U03KxpNcDo/s1600/IC577712.jpg)
1. In the Command Prompt, type the following commands to set up the partitions on your external SSD or HDD:
```bash
diskpart
list volume  # Find the letter of your USB flash drive
select disk <disk no.>  # Select your external SSD or HDD
clean
convert gpt

# Create Recovery Partition
create partition primary size=450
format quick fs=ntfs label="Windows RE Tools"
assign letter="T"
set id="de94bba4-06d1-4d40-a16a-bfd50179d6ac"
gpt attributes=0x8000000000000001

# Create EFI Partition
create partition efi size=260
format quick fs=fat32 label="System"
assign letter="S"

# Create MSR Partition
create partition msr size=128

# Create Windows Partition
create partition primary size=372000  # Adjust the size as needed for Windows
format quick fs=ntfs label="Windows"
assign letter="W"

# Create Recovery Image Partition
create partition primary size=4096
format quick fs=ntfs label="Recovery Image"
assign letter="R"
set id="de94bba4-06d1-4d40-a16a-bfd50179d6ac"
gpt attributes=0x8000000000000001

list volume 
```
#### Step 4: Copy Windows Installation Files
Now we need to copy the necessary Windows 10 installation files from the flash drive to different partitions on the SSD. Execute the following commands:
```bash
md R:\RecoveryImage

# Replace 'D' with your USB flash drive letter
copy D:\sources\install.wim 
R:\RecoveryImage\install.wim  
```
#### Note: Change the Index to the Windows 10 version you want to install. 
```bash
# Replace 'D' with your USB flash drive letter
dism /Get-WimInfo /WimFile:D:\sources\install.wim 
```
![Windows Index]( https://howtomanagedevices.com/wp-content/uploads/2021/01/image-112-1024x644.png)
```bash
cd X:\Windows\System32
dism /Apply-Image /ImageFile:R:\RecoveryImage\install.wim /Index:6 /ApplyDir:W:\  # Adjust the index as needed
```
![Dism](https://i0.wp.com/www.kjctech.net/wp-content/uploads/2018/05/image-3.png?fit=774%2C250&ssl=1)

```bash
md T:\Recovery\WindowsRE
copy W:\Windows\System32\Recovery\winre.wim T:\Recovery\WindowsRE\winre.wim

bcdboot W:\Windows /s S: /f UEFI

W:\Windows\System32\reagentc /setosimage /path R:\RecoveryImage /target W:\Windows /index 6
W:\Windows\System32\reagentc /setreimage /path T:\Recovery\WindowsRE /target W:\Windows
```

#### Restart again and choose EFI boot.
![Windows Boot](https://www.niallbrady.com/wp-content/uploads/2017/11/this-might-take-several-minutes.png)

### Follow on-screen instructions to install Windows 10.
### Step 5: Configuring for Mac Compatibility (Optional)

To use the SSD with a Mac, restart your Mac and hold `⌘ + R` to enter recovery mode.

- Allow booting from external media for Macs with Apple's T2 security chips.
- Set the security level to "None".
- Restart again and choose EFI Boot from the boot options.
- Follow the on-screen instructions to install Windows 10.
- Install the Windows Support Software from the USB flash drive, and you're good to go.
