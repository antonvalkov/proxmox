#!/bin/bash
# Proxmox Suspend/Shutdown VMs Script
# To be run before rebooting the Proxmox server
# Suspends VMs to disk if possible, shuts down VMs with PCI passthrough devices

########### Variables ###############
SUSPENDED_VMS_FILE="/root/suspended_vms.txt"  # File to store suspended VMIDs
SHUTDOWN_VMS_FILE="/root/shutdown_vms.txt"    # File to store shut-down VMIDs
EXCLUDE_VMIDS=()  # Add any specific VMIDs you want to exclude (optional)

########### End Variables ############

# Clear the suspended and shut-down VMs files
> $SUSPENDED_VMS_FILE
> $SHUTDOWN_VMS_FILE

# Get a list of all running VMs
RUNNING_VMS=$(qm list | grep running | awk '{print $1}')

# Process each running VM
for VMID in $RUNNING_VMS; do
    # Check if VMID is in the exclude list
    if [[ " ${EXCLUDE_VMIDS[@]} " =~ " ${VMID} " ]]; then
        echo "Skipping VM $VMID (excluded)"
        continue
    fi

    echo "Processing VM $VMID..."

    # Attempt to suspend the VM to disk
    if qm suspend $VMID --todisk 2>/dev/null; then
        # VM suspended successfully
        echo "VM $VMID suspended to disk."
        echo $VMID >> $SUSPENDED_VMS_FILE
    else
        # VM has PCI passthrough devices and cannot be suspended to disk
        echo "VM $VMID has PCI passthrough devices and cannot be suspended to disk. Shutting down..."
        qm shutdown $VMID

        # Wait for the VM to shut down
        while [[ ! "$(qm status $VMID)" =~ "stopped" ]]; do
            CURRENT_STATUS=$(qm status $VMID)
            echo "Current status of VM $VMID: $CURRENT_STATUS"
            echo "Waiting for VM $VMID to shut down..."
            sleep 5
        done

        echo $VMID >> $SHUTDOWN_VMS_FILE
    fi
done

echo "All VMs processed. Proceeding with reboot..."