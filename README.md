#TEST 1 
# Intel Container Experience Kits Setup Scripts

Intel Container Experience Kits Setup Scripts provide a simplified mechanism for installing and configuring Kubernetes clusters on Intel Architecture using Ansible.

The software provided here is for reference only and not intended for production environments.

## Quickstart guide

1. Initialize git submodules to download Kubespray code.

    ```bash
    git submodule update --init
    ```

2. Decide which configuration profile you want to use and export environmental variable.
   For VM case is PROFILE environment variable mandatory.
   > **_NOTE:_** For non-VM case it will be used only to ease execution of the steps listed below.
    - For **Kubernetes Basic Infrastructure** deployment:

        ```bash
        export PROFILE=basic
        ```

    - For **Kubernetes Access Edge Infrastructure** deployment:

        ```bash
        export PROFILE=access
        ```

    - For **Kubernetes Regional Data Center Infrastructure** deployment:

        ```bash
        export PROFILE=regional_dc
        ```

    - For **Kubernetes Remote Forwarding Platform Infrastructure** deployment:

        ```bash
        export PROFILE=remote_fp
        ```

    - For **Kubernetes Infrastructure On Customer Premises** deployment:

        ```bash
        export PROFILE=on_prem
        ```

    - For **Kubernetes Full NFV Infrastructure** deployment:

        ```bash
        export PROFILE=full_nfv
        ```

    - For **Kubernetes Storage Infrastructure** deployment:

        ```bash
        export PROFILE=storage
        ```

    - For **Kubernetes Build-Your-Own Infrastructure** deployment:

        ```bash
        export PROFILE=build_your_own
        ```

3. Install dependencies

   ```bash
   pip3 install -r requirements.txt
   ```

4. Generate example host_vars, group_vars and inventory files for Intel Container Experience Kits profiles.

   > **_NOTE:_** It is **highly recommended** to read [this](docs/generate_profiles.md) file before profiles generation.

    ```bash
    make examples ARCH=<skl,clx,**icx**,spr> NIC=<fvl,**cvl**>
    ```

5. Copy example inventory file to the project root dir.

    ```bash
    cp examples/k8s/${PROFILE}/inventory.ini .
    ```

    or, for VM case:

    ```bash
    cp examples/vm/${PROFILE}/inventory.ini .
    ```

6. Update inventory file with your environment details.

    For VM case: update details relevant for vm_host

    > **_NOTE:_** At this stage you can inspect your target environment by running:

    ```bash
    ansible -i inventory.ini -m setup all > all_system_facts.txt
    ```

    In `all_system_facts.txt` file you will find details about your hardware, operating system and network interfaces, which will help to properly configure Ansible variables in the next steps.

7. Copy group_vars and host_vars directories to the project root dir.

    ```bash
    cp -r examples/k8s/${PROFILE}/group_vars examples/k8s/${PROFILE}/host_vars .

    or

    For VM case:
    cp -r examples/vm/${PROFILE}/group_vars examples/vm/${PROFILE}/host_vars .
    ```

8. Update group and host vars to match your desired configuration. Refer to [this section](#configuration) for more details.

    > **_NOTE:_** Please pay special attention to the `http_proxy`, `https_proxy` and `additional_no_proxy` vars if you're behind proxy.

    For VM case:
    - update details relevant for vm_host (e.g.: datalane_interfaces, ...)
    - update VMs definition in host_vars/host-for-vms-1.yml
    - update/create host_vars for all defined VMs (e.g.: host_vars/vm-ctrl-1.yml and host_vars/vm-work-1.yml)
      Needed details are at least dataplane_interfaces
      For more details see [VM case configuration guide](docs/vm_config_guide.md)

9. **Recommended:** Apply bug fix patch for Kubespray submodule (Required for RHEL 8+ and Ubuntu 22.04).

    ```bash
    ansible-playbook -i inventory.ini playbooks/k8s/patch_kubespray.yml
    ```

10. Execute `ansible-playbook`.

    ```bash
    ansible-playbook -i inventory.ini playbooks/${PROFILE}.yml
    ```

    or, for VM case:

    ```bash
    ansible-playbook -i inventory.ini playbooks/vm.yml
    ```

    > **_NOTE:_** VMs are accessible from ansible host via ssh vm-ctrl-1 or ssh vm-work-1

## Configuration

Refer to the documentation linked below to see configuration details for selected capabilities and deployment profiles.

- [SRIOV Network Device Plugin and SRIOV CNI plugin](docs/sriov.md)
- [MinIO Operator](docs/storage.md)
- [VM case configuration guide](docs/vm_config_guide.md)
## Prerequisites and Requirements

- Required packages on the target servers: **Python3**.
- Required packages on the ansible host (where ansible playbooks are run): **Python3 and Pip3**.
- Required python packages on the ansible host.  **See requirements.txt**.

- SSH keys copied to all Kubernetes cluster nodes (`ssh-copy-id <user>@<host>` command can be used for that).
- For VM case SSH keys copied to all VM hosts (`ssh-copy-id <user>@<host>` command can be used for that).
- Internet access on all target servers is mandatory. Proxy is supported.
- At least 8GB of RAM on the target servers/VMs for minimal number of functions (some Docker image builds are memory-hungry and may cause OOM kills of Docker registry - observed with 4GB of RAM), more if you plan to run heavy workloads such as NFV applications.

- For the `RHEL`-like OSes `SELinux` must be configured prior to the CEK deployment and required `SELinux`-related packages should be installed.
  `CEK` itself is keeping initial `SELinux` state but `SELinux`-related packages might be installed during `k8s` cluster deployment as a dependency, for `Docker` engine e.g.,
  causing OS boot failure or other inconsistencies if `SELinux` is not configured properly.
  Preferable `SELinux` state is `permissive`.

  For more details, please, refer to the respective OS documentation.
