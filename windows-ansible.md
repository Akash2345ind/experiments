To get Ansible talking to a Windows VM from a CentOS 9 machine in a CLI-only environment, you essentially need to build a "bridge" using **WinRM (Windows Remote Management)**. Since you can't use `gcloud` commands, we'll focus entirely on what to run inside the shells of both machines.

---

## Phase 1: Prepare the Windows Target (The Managed Node)

Since you are CLI-only, you are likely using the **Google Cloud Serial Console** or an existing SSH connection to access PowerShell. Ansible needs WinRM enabled to execute commands.

1.  **Open PowerShell** (if not already there).
2.  **Run the Setup Script**: The easiest way is to use the official Ansible configuration script. Run these commands to download and execute it:

```powershell
$url = "https://raw.githubusercontent.com/ansible/ansible-documentation/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file
```

3.  **Verify WinRM is listening**:
```powershell
winrm enumerate winrm/config/Listener
```
*Note: This script enables WinRM on ports 5985 (HTTP) and 5986 (HTTPS). Ensure your GCP VPC firewall allows traffic on these ports between the two VMs.*

---

## Phase 2: Prepare the CentOS 9 VM (The Control Node)

CentOS 9 Stream uses `dnf`. You need to install Ansible and the `pywinrm` library, which allows Python (and thus Ansible) to speak the WinRM protocol.

1.  **Install Python and Pip**:
```bash
sudo dnf install -y python3 python3-pip
```

2.  **Install Ansible and WinRM dependency**:
```bash
sudo pip3 install ansible
sudo pip3 install "pywinrm>=0.3.0"
```

3.  **Install the Windows Collection**: Ansible's Windows modules are bundled in a specific collection.
```bash
ansible-galaxy collection install ansible.windows
```
---
Ubuntu machine commands
```bash
sudo apt update
sudo apt install -y python3 python3-pip
sudo pip3 install ansible --break-system-packages
sudo pip3 install "pywinrm>=0.3.0" --break-system-packages
ansible-galaxy collection install ansible.windows
```
---

## Phase 3: Configure the Ansible Inventory

You need to tell Ansible how to find the Windows machine and what credentials to use. Create a file named `inventory.ini`.



```ini
[windows]
# Replace with the Internal IP of your Windows VM
10.x.x.x 

[windows:vars]
ansible_user=YOUR_WINDOWS_USERNAME
ansible_password=YOUR_WINDOWS_PASSWORD
ansible_connection=winrm
# Use 'ignore' for self-signed certs created by the script in Phase 1
ansible_winrm_server_cert_validation=ignore
```

---

## Phase 4: Create the IIS Playbook

Create a file named `install_iis.yml`. We will use the `win_feature` module to handle the installation.

```yaml
---
- name: Install IIS on Windows
  hosts: windows
  tasks:
    - name: Install IIS (Web-Server)
      ansible.windows.win_feature:
        name: Web-Server
        state: present
        include_management_tools: yes
      register: iis_result

    - name: Reboot if IIS installation requires it
      ansible.windows.win_reboot:
      when: iis_result.reboot_required
```

---

## Phase 5: Run the Playbook

Now, execute the playbook from your CentOS CLI:

```bash
ansible-playbook -i inventory.ini install_iis.yml
```

### Troubleshooting Tips:
* **Connectivity**: From CentOS, try `ping <Windows_Internal_IP>`. If it fails, check your GCP Firewall Rules (you'll need to do this in the GCP Web Console since `gcloud` is restricted).
* **Authentication**: If you get a 401 Unauthorized error, double-check the username and password in the `inventory.ini`. If you are using a local account, the username is usually just the name; for domain accounts, use `user@domain.com`.
* **Port 5986**: If you want to be extra secure using HTTPS, ensure the script in Phase 1 finished successfully and that port 5986 is open.

Since you're working in a restricted CLI environment, are you using local Windows accounts or are these VMs joined to an Active Directory domain?
