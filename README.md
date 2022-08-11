# ccdc.vagrant-base-box

[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-ccdc.vagrant-base-box-blue.svg)](https://galaxy.ansible.com/ccdc/debloat_windows/)
[![Ansible Lint](https://github.com/ccdc-opensource/ansible-role-vagrant-base-box/actions/workflows/lint-ansible-role.yml/badge.svg)](https://github.com/ccdc-opensource/ansible-role-debloat-windows/actions/workflows/lint-ansible-role.yml)
[![Common CCDC PR Checks](https://github.com/ccdc-opensource/ansible-role-vagrant-base-box/actions/workflows/common_ccdc_status_checks.yml/badge.svg)](https://github.com/ccdc-opensource/ansible-role-debloat-windows/actions/workflows/common_ccdc_status_checks.yml)

An Ansible role to perform some provisioning on virtual machines for exporting as Vagrant base boxes.

## What does this role do?

- [Windows] Rearrange drives so hard drives are clustered from C: onwards and optical drives after them
- [Windows] Install all available system updates
- [Windows] Precompile .NET assemblies
- [Windows] Set PowerShell execution policy to `RemoteSigned`
- [Windows] Disable password expiration for `vagrant` user
- [Windows] Set all network connections to be `Private` by default
- [Windows] Enable access to the VM via RDP
- [Windows] Enable access to the VM via SSH (Windows Server only)
- [Windows] Enable User Account Control
- [Windows] Set power management plan to High Performance

## Requirements

For provisioning Windows VMs, this role requires the
`[community.windows](https://galaxy.ansible.com/community/windows)` collection.

## Role Variables

Most of the variables for this role are simple booleans to determine which customization tasks
should be run; the only task with complex variables controlling its behaviour is the task to
rearrange the system's drives (see below).

| Variable                          | Default value      | Comments                                                                  |
|-----------------------------------|--------------------|---------------------------------------------------------------------------|
| `update_windows`                  | `true`             | Whether or not to install any available operating system updates.         |
| `precompile_dotnet`               | `true`             | Whether or not to precompile .NET assemblies.                             |
| `target_execution_policy`         | `RemoteSigned`     | The execution policy to set. Set this to an empty string to skip.         |
| `disable_vagrant_password_expiry` | `true`             | Whether to disable password expiration for the `vagrant` user.            |
| `disable_network_location_prompt` | `true`             | Whether to disable network type prompt on new network connections.        |
| `set_all_networks_private`        | `true`             | Whether to set all current network connections to be "Private".           |
| `enable_rdp`                      | `true`             | Whether to set the system up to allow access via RDP.                     |
| `enable_ssh`                      | `true`             | Whether to set the system up to allow access via SSH.                     |
| `add_vagrant_ssh_pubkey`          | `true`             | Add the default Vagrant SSH public key to `vagrant`'s `authorized_keys`.  |
| `enable_uac`                      | `true`             | Whether to enable User Account Control.                                   |
| `power_plan`                      | `High performance` | The Windows power management plan to set. Set to an empty string to skip. |

### Rearrange hard drives

This task will rearrange all hard drives as per a given specification and then sort all optical drives to
after the final hard drive.

| Variable              |  Default  | Comments                                                                  |
|-----------------------|-----------|---------------------------------------------------------------------------|
| `rearrange_drives`    | `true`    | Whether or not to rearrange drives. Set to false to skip the entire task. |
| `initialize_disks`    | see below | Disks to bring online/offline on Windows Server.                          |
| `drive_configuration` | see below | A list of mappings of drive labels to their desired letters.              |

#### Default values for `drive_configuration`

At the CCDC, Windows build machines have two drives in addition to the system drive. These are assigned
drive letters as per this specification:

```yaml
drive_configuration:
  - letter: d
    label: scratch
    number: 1
    online: true
    readonly: false
  - letter: e
    label: builds
    number: 2
    online: true
    readonly: false
```

You can similarly assign specific drive letters to any drives you've specified in your VM
configuration and formatted using the `<DiskConfiguration>` section in your Windows
`autounattend.xml` file.

The `number`, `online` and `readonly` fields in the `drive_configuration` section apply only to Windows
Server targets; on these systems the drive configuration task will use the drive number to ensure
the relevant drive is online and read-only as specified here. This is because on e.g. VMWare guests,
disks are often connected via the `pvscsi` driver which Windows Server assumes makes them SAN volumes
and causes it to set them up as offline by default.

Please note that offline drives cannot be reassigned to new drive letters, so if you wish to specify
drive letters on Windows Server you *must* set the drive to be online, not read-only and specify
its drive number.
