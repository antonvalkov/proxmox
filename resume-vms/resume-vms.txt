#!/bin/bash
# Proxmox Resume/Start VMs Script
# To be run after rebooting the Proxmox server
# Resumes suspended VMs and starts shut-down VMs

########### Variables ###############
SUSPENDED_VMS_FILE="/root/suspended_vms.txt"  # File containing suspended VMIDs
SHUTDOWN_VMS_FILE="/root/shutdown_vms.txt"    # File containing shut-down VMIDs

########### End Variables ############

# Resume suspended VMs
if [[ -f $SUSPENDED_VMS_FILE ]]; then
    while read -r VMID; do
        echo "Resuming VM $VMID from disk..."
        qm resume $VMID
    done < $SUSPENDED_VMS_FILE
else
    echo "No suspended VMs file found."
fi

# Start shut-down VMs
if [[ -f $SHUTDOWN_VMS_FILE ]]; then
    while read -r VMID; do
        echo "Starting VM $VMID..."
        qm start $VMID
    done < $SHUTDOWN_VMS_FILE
else
    echo "No shut-down VMs file found."
fi

# Clear the suspended and shut-down VMs files
> $SUSPENDED_VMS_FILE
> $SHUTDOWN_VMS_FILE

echo "All VMs processed."