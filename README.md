# Raspberry Pi PTP Server with Ansible

This repository contains an Ansible control node configuration that provisions a Raspberry Pi as a Precision Time Protocol (PTP) grandmaster using the `linuxptp` suite (`ptp4l` and optionally `phc2sys`).

## Repository structure

```
ansible.cfg                  # Ansible configuration pointing at the sample inventory
inventories/
  production/
    hosts                    # Example inventory with a Raspberry Pi host
    group_vars/
      ptp_servers.yml        # Default variable values for hosts in the ptp_servers group
playbooks/
  deploy_ptp_server.yml      # Main playbook that applies the ptp_server role
roles/
  ptp_server/                # Role that installs and configures linuxptp services
```

## Requirements

* A control node with Ansible 2.12+ and Python 3.8+
* SSH access to the Raspberry Pi with sudo privileges (the playbook assumes the default `pi` user but you can change it in the inventory)
* The managed node must be running a Debian-based operating system (Raspberry Pi OS or Ubuntu Server) with `apt`

## Usage

1. Update `inventories/production/hosts` with the address and SSH details of your Raspberry Pi.
2. Adjust any settings in `inventories/production/group_vars/ptp_servers.yml` to match your desired PTP parameters. Key options include the network interface name, PTP priorities, announce/sync intervals, and whether to run the optional `phc2sys` service.
3. Run the playbook:

   ```bash
   ansible-playbook playbooks/deploy_ptp_server.yml
   ```

   The playbook installs the `linuxptp` package, deploys a deterministic `ptp4l.conf`, configures `/etc/default/ptp4l`, and ensures the `ptp4l` systemd service is enabled and running. If `phc2sys_enable` is set to `true`, the `phc2sys` service is also configured and started.

## Customisation

* **Interface specific options** – Set `ptp4l_extra_interface_options` (a dictionary) to append options to the interface section of `ptp4l.conf`.
* **Global options** – Use `ptp4l_extra_global_options` to add additional key/value pairs under the global section of the configuration file.
* **Service options** – Add extra command-line flags to the `ptp4l` systemd service by populating the `ptp4l_service_extra_options` list.
* **PHC synchronisation** – Enable and customise the `phc2sys` daemon by setting `phc2sys_enable: true` and adjusting the `phc2sys_options` list to suit your hardware capabilities.

## Verifying the deployment

After running the playbook, SSH into the Raspberry Pi and check the status of the services:

```bash
sudo systemctl status ptp4l
sudo journalctl -u ptp4l -f
```

You should see log messages indicating that the node is advertising itself as the PTP master on the configured interface.

## Troubleshooting

* Ensure that the Raspberry Pi and downstream PTP clients are connected to the same layer-2 network (PTP requires hardware timestamping support for best accuracy).
* If you need software timestamping (the default), keep `ptp4l_hardware_timestamping` set to `false`. Set it to `true` only when the NIC and driver provide a hardware clock.
* Adjust PTP priority values (`ptp4l_priority1`, `ptp4l_priority2`) to control which node becomes the grandmaster when multiple masters are present.
