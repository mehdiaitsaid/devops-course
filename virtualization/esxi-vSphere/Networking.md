# Lab: Configuring Networking with vSwitch, Port Group, and VMkernel NIC in ESXi

**Note**: If you're running ESXi as a VM (nested virtualization) in VMware Workstation or another hypervisor, you will need to add a new **network adapter** to the ESXi virtual machine. This will allow you to connect to the new vSwitch and port group.

In this lab, you will configure networking by creating a new **vSwitch** (virtual switch), a **port group**, and a **VMkernel NIC** to allow network communication for management and other functions. You will also connect a VM to this network.

---

## Task:

1. **Create a New vSwitch**: Configure a virtual switch to provide network connectivity to the virtual machines.
2. **Create a New Port Group**: Set up a port group on the new vSwitch to allow VMs to be connected to this virtual network.
3. **Configure a VMkernel NIC**: Create and configure a VMkernel NIC for management or other network functions.
4. **Connect a VM to the New Port Group**: Modify the network settings of a VM to connect it to the newly created port group.

---

## Steps:

### 1. Add a New Network Adapter (if running ESXi on a VM)

1. Log in to VMware Workstation or your hypervisor management interface.
2. Select the ESXi virtual machine and ensure it is powered off.
3. Add a new **network adapter** to the VM, which will be used for the new vSwitch. This network adapter will be bridged or connected to your physical network for the vSwitch.
4. Power on the ESXi VM after adding the new adapter.

### 2. Create a New vSwitch

1. Log in to the **vSphere Web Client** using your ESXi host IP address.
2. Navigate to **Networking** under **Host** > **Manage**.
3. Select the **Virtual Switches** tab.
4. Click **Add vSwitch** to create a new virtual switch.
   - **vSwitch Name**: Enter a name for the new switch (e.g., `vSwitch2`).
   - **Network Adapter**: Choose the newly added network adapter for the vSwitch.
   - **MTU**: Set the MTU (Maximum Transmission Unit) value. The default is typically `1500`.
5. Click **OK** to save the new vSwitch.

### 3. Create a New Port Group

1. Navigate to the **Port Groups** tab under **Networking**.
2. Click **Add Port Group** to create a new port group.
3. Provide the following details:
   - **Port Group Name**: Enter a name for the new port group (e.g., `VM_Network`).
   - **VSwitch**: Select the newly created vSwitch (e.g., `vSwitch2`).
   - **VLAN ID**: Set the VLAN ID if needed, or leave it as the default (e.g., `0` for no VLAN).
4. Click **OK** to create the port group.

### 4. Create a New VMkernel NIC

1. Go to the **VMkernel NICs** tab under **Networking**.
2. Click **Add VMkernel NIC** to create a new VMkernel NIC.
3. Provide the following details:
   - **Network Adapter**: Select the network adapter you added earlier.
   - **Port Group**: Select the port group created in the previous step (e.g., `VM_Network`).
   - **VMkernel NIC Settings**:
     - Choose the function for the VMkernel NIC (e.g., **Management Network** for ESXi management, or other functions like **vMotion** or **Storage**).
     - If necessary, set the **IPv4 Address** and **Subnet Mask** for the VMkernel NIC.
4. Click **OK** to create the VMkernel NIC.

### 5. Connect a VM to the New Port Group

1. In the **vSphere Web Client**, navigate to **VMs** and select the VM you want to connect to the new network.
2. Click **Edit Settings** for the selected VM.
3. In the **Network Adapter** section, click **Remove** to remove the current network adapter.
4. Click **Add** to add a new network adapter.
5. In the **Network Connection** dropdown, select the newly created port group (e.g., `VM_Network`).
6. Click **OK** to apply the changes.

### 6. Verify the Network Configuration

1. Power on the VM and check the network configuration inside the guest OS.
2. Verify that the VM receives an IP address from the new network and can communicate with other devices on the network.
3. You can use commands like `ifconfig` (Linux) or `ipconfig` (Windows) to check the network settings inside the VM.

### 7. Verify VMkernel NIC Connectivity

1. On the ESXi host, use the **esxcli** or **vicfg** commands to check the VMkernel NIC settings and verify that the network is properly configured.
2. Check connectivity to the management network or other network functions as required.
