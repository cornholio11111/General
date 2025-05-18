#!/bin/bash

echo "=== Proxmox Full Cluster Nuker ==="
echo "This will hard reset this node outta the cluster."
echo "You can also choose to wipe VM/CT configs and regenerate SSL certs (fixes busted GUI cert errors)."
read -p "You sure you wanna do this? (y/n): " confirm
if [[ $confirm != "y" ]]; then
    echo "Alright, no harm done. Exiting."
    exit 1
fi

echo ""
echo "[1] Stopping cluster services (pve-cluster & corosync)..."
systemctl stop pve-cluster
systemctl stop corosync

echo ""
echo "[2] Starting pve-cluster in LOCAL mode so we can mess with stuff..."
pmxcfs -l

echo ""
echo "[3] Blowing away corosync config files..."
rm -f /etc/pve/corosync.conf
rm -rf /etc/corosync/*

echo ""
echo "[4] Killing pmxcfs local and restarting pve-cluster..."
killall pmxcfs
systemctl start pve-cluster

echo ""
read -p "[5] Wipe all *.conf files from VMs and containers? (y/n): " wipe_configs
if [[ $wipe_configs == "y" ]]; then
    echo "Deleting all VM and LXC config files. Gone like the wind..."
    rm -f /etc/pve/qemu-server/*.conf
    rm -f /etc/pve/lxc/*.conf
    for node_dir in /etc/pve/nodes/*; do
        rm -f "$node_dir/qemu-server/"*.conf 2>/dev/null
        rm -f "$node_dir/lxc/"*.conf 2>/dev/null
    done
else
    echo "Cool, leaving the *.conf files alone."
fi

echo ""
read -p "[6] Wanna blow away SSL certs and regenerate clean ones? (y/n): " fix_ssl
if [[ $fix_ssl == "y" ]]; then
    echo "Yeeting old certs... this should fix the GUI being mad about certs."
    rm -f /etc/pve/priv/pve-root-ca.key /etc/pve/pve-root-ca.pem
    rm -f /etc/pve/local/pve-ssl.pem /etc/pve/local/pve-ssl.key
    rm -f /etc/pve/local/pveproxy-ssl.pem /etc/pve/local/pveproxy-ssl.key

    echo "Rebuilding fresh SSL certs..."
    pvecm updatecerts
    systemctl restart pveproxy
else
    echo "Skipping SSL cleanup. GUI might still squawk if certs are janky."
fi

echo ""
echo "✅ Done. This node is no longer in a cluster."
echo "You can now create a fresh one with 'pvecm create <name>' or just keep it standalone."
echo "If anything feels weird, reboot the box. Proxmox usually sorts itself out after that."
