Creating a USB flashing tool in C that handles different format options, partition tables, and is compatible with live boot disks is a complex task. This requires an understanding of various system-level programming concepts such as raw disk I/O, partition management, and file system creation. Below is a simplified version of how you could start building such a tool. Please note that this example is simplified and would need to be significantly expanded and tested for real-world use.

### High-Level Steps for USB Flashing Tool:

1. **Detect USB Devices**
2. **Partition the USB Drive (MBR/GPT)**
3. **Format the USB Drive (FAT32, NTFS, ext4)**
4. **Flash the ISO Image**
5. **Make the USB Bootable**

### 1. **Setup and Detect USB Devices**

To interact with the USB drive, you will need to access the raw disk I/O. On Linux, you can use the `/dev` directory to access devices.

### 2. **Partition the USB Drive**

You'll need to work with the Master Boot Record (MBR) or GUID Partition Table (GPT) to create partitions. This can be done by writing directly to the raw device, or using system commands like `parted` or `fdisk` through `system()` calls.

### 3. **Format the USB Drive**

Depending on the chosen file system, you would need to call system utilities like `mkfs.vfat`, `mkfs.ntfs`, or `mkfs.ext4`.

### 4. **Flash the ISO Image**

For writing the ISO image, you can use the `dd` command or directly write the file to the device using raw file I/O.

### 5. **Make the USB Bootable**

Ensure the bootloader is correctly placed on the USB drive. For BIOS systems, GRUB can be installed. For UEFI, you might need to create an EFI system partition.

### Example Code

Here’s a highly simplified version that outlines some of the steps in C:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

void execute_command(const char *command) {
    int ret = system(command);
    if (ret == -1) {
        perror("Error executing command");
        exit(EXIT_FAILURE);
    }
}

void detect_usb_devices() {
    printf("Detecting USB devices...\n");
    execute_command("lsblk -o NAME,SIZE,TYPE,MOUNTPOINT | grep disk");
}

void create_partition_table(const char *device, const char *table_type) {
    printf("Creating %s partition table on %s...\n", table_type, device);
    char command[100];
    snprintf(command, sizeof(command), "parted %s mklabel %s", device, table_type);
    execute_command(command);
}

void create_partition(const char *device, const char *start, const char *end, const char *fs_type) {
    printf("Creating partition on %s...\n", device);
    char command[100];
    snprintf(command, sizeof(command), "parted %s mkpart primary %s %s %s", device, fs_type, start, end);
    execute_command(command);
}

void format_partition(const char *partition, const char *fs_type) {
    printf("Formatting %s as %s...\n", partition, fs_type);
    char command[100];
    if (strcmp(fs_type, "fat32") == 0) {
        snprintf(command, sizeof(command), "mkfs.vfat -F 32 %s", partition);
    } else if (strcmp(fs_type, "ntfs") == 0) {
        snprintf(command, sizeof(command), "mkfs.ntfs %s", partition);
    } else if (strcmp(fs_type, "ext4") == 0) {
        snprintf(command, sizeof(command), "mkfs.ext4 %s", partition);
    } else {
        fprintf(stderr, "Unsupported file system type: %s\n", fs_type);
        exit(EXIT_FAILURE);
    }
    execute_command(command);
}

void flash_iso(const char *iso_path, const char *device) {
    printf("Flashing ISO %s to %s...\n", iso_path, device);
    char command[100];
    snprintf(command, sizeof(command), "dd if=%s of=%s bs=4M status=progress", iso_path, device);
    execute_command(command);
}

int main() {
    char device[50], iso_path[100], table_type[10], fs_type[10];
    
    detect_usb_devices();

    printf("Enter device (e.g., /dev/sdb): ");
    scanf("%s", device);

    printf("Enter partition table type (mbr/gpt): ");
    scanf("%s", table_type);
    create_partition_table(device, table_type);

    printf("Enter file system type (fat32/ntfs/ext4): ");
    scanf("%s", fs_type);

    create_partition(device, "0%", "100%", fs_type);
    format_partition(strcat(device, "1"), fs_type); // Assuming single partition

    printf("Enter path to ISO: ");
    scanf("%s", iso_path);
    flash_iso(iso_path, device);

    printf("USB drive is ready!\n");

    return 0;
}
```

### How the Code Works:

1. **Detect USB Devices:**
   - The tool uses `lsblk` to list all block devices and allows the user to choose the correct one.
  
2. **Create Partition Table:**
   - The user chooses MBR or GPT, and the tool creates this partition table using `parted`.

3. **Create and Format Partition:**
   - A single partition is created that spans the entire drive and is formatted according to the user's choice.

4. **Flash the ISO:**
   - The ISO is written to the USB using `dd`.

### Considerations and Expansions:

- **Error Checking:** The code currently does minimal error checking. In a real-world scenario, you would need to handle errors more gracefully.
- **Multiple Partitions:** If you need multiple partitions, you would need to extend the code to handle this.
- **Bootloader Installation:** The current code doesn’t handle installing a bootloader like GRUB, which would be necessary for most OS installations.
- **Compatibility:** This example is Linux-specific. For cross-platform support (e.g., Windows), you'd need to use different system commands and handle disk I/O accordingly.

### Summary:

This is a basic template for a USB flashing tool in C. Building a fully functional tool like Rufus would require more work, especially for handling different file systems, bootloaders, and error conditions. However, this example provides a starting point for understanding how to interact with low-level disk I/O in C.
