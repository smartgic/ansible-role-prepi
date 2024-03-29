[![Ansible Galaxy](https://img.shields.io/badge/galaxy-smartgic.prepi-green.svg?style=flat)](https://galaxy.ansible.com/smartgic/prepi)
[![Discord](https://img.shields.io/discord/809074036733902888)](https://discord.gg/Vu7Wmd9j)

# Ansible Role: Prepi

This Ansible role will apply some configurations to obtain the best performances from a Raspberry Pi 4B board and later.

`prepi` stands for "Prepare Raspberry Pi".

**First, a major, MAJOR caveat**: You are responsible of the choices made with this role, you decide which firmware[1], which EEPROM[2] you want to use. The same rule applies to the overclocking[3] feature. **Remember that overclocking requires a proper active cooling system.**

That being said, this role performs the following tasks _(depending your wishes)_:

- Update Raspberry Pi OS to the latest version
- Add Debian backports repository _(customizable)_
- Update firmware using the `next` branch which provide kernel 5.10 and edge firmware _(customizable)_
- Update EEPROM using the `beta` version which provide edge features _(customizable)_
- Setup `initial_turbo` to speedup the boot process
- Overclock the Raspberry Pi to 2Ghz _(customizable)_
- Mount `/tmp` on a RAMDisk
- Optimize `/` partition mount options to improve SDcard read/write
- Manage I2C, SPI & UART interfaces _(customizable)_
- Set CPU governor to `performance` to avoid context switching between the `idle*` kernel functions _(customizable)_
- Install and configure PulseAudio (non-system wide) _(customizable)_
- Reboot the Raspberry Pi when new firmware or EEPROM are installed _(customizable)_

This role has been developed on a Raspberry Pi 4B with [Raspberry Pi OS 64-bit](https://downloads.raspberrypi.org/raspios_arm64/images).

Please read the documentation links below before taking any decisions.

- _[1]_ https://www.raspberrypi.com/documentation/computers/os.html#rpi-update
- _[2]_ https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#raspberry-pi-4-boot-eeprom
- _[3]_ https://www.raspberrypi.com/documentation/computers/config_txt.html#overclocking-options

## Requirements

- Raspberry Pi 4B board and later
- [Raspberry Pi OS 64-bit](https://downloads.raspberrypi.org/raspios_arm64/images) _(required for board with more than 4GB of RAM)_

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
---
# System user to add to system group such as gpio, audio, etc...
prepi_pi_user: pi

# Set the Raspberry Pi hostname
prepi_hostname: smartgic-pi4b8-13

# Set the CPU governor
#  - ondemand (less power consumption but less performances)
#  - performance (more power consumption but more performances)
prepi_cpu_governor: performance

# Use fast DNS nameservers (CloudFlare and Google)
# Keep in mind that if you are using a DHCP server these changes will be override
prepi_nameservers:
  - 1.1.1.1
  - 8.8.8.8

# Update the firmware
prepi_firmware_update: yes

# Which firmware branch to use
#  - master (stable)
#  - next (edge firmware and kernel)
prepi_firmware_branch: next

# Which EEPROM release to use
#  - critical (rarely updated)
#  - stable (updated when new/advanced features have been successfully beta tested)
#  - beta (new or experimental features are tested here first)
prepi_eeprom: beta

# Overclock CPU/GPU
prepi_overclock: yes

# Use with caution, this parameter allows to use an higher voltage
prepi_force_turbo: no

# CPU frequence to overclock
# If the Raspberry Pi is unstable try to reduce the clock
prepi_cpu_freq: 2000

# GPU frequence to overclock
prepi_gpu_freq: 650

# Voltage to use, the higher the CPU clock will be the higher the voltage should be
# 6 is the limit when prepi_force_turbo is set to no, higher voltage could damage your board
prepi_voltage: 6

# Enable I2C interface
prepi_i2c_enable: yes

# Enable SPI interface
prepi_spi_enable: yes

# Disable UART interface
prepi_uart_enable: no

# Install and configure PulseAudio
prepi_pulseaudio: yes

# PulseAudio persistent when user disconnect and start at boot time.
prepi_pulseaudio_daemon: yes

# Enable Debian backports repository to get the latest packages
prepi_backports_enable: yes

# Reboot the Raspberry Pi when new firmware or EEPROM are installed
prepi_reboot: yes

# Disable the onboarding screen configuration (when using the Raspberry Pi OS desktop version)
prepi_disable_onboarding_screen: yes
```

## Dependencies

Same as for the requirements section.

## Example Playbook

Inventory file with `rpi` group which has one host named `rpi4b01` with the IP address `192.168.1.97`.

```ini
[rpi]
rpi4b01 ansible_host=192.168.1.97 ansible_user=pi
```

Basic playbook running on `rpi` group using the `pi` user to connect via SSH _(based on the inventory)_ with some custom variables.

```yaml
---
- hosts: rpi
  gather_facts: no
  become: yes

  # Install Python using raw module to make sure requirements are installed
  pre_tasks:
    - name: Install Python 3.x Ansible requirement
      raw: apt-get install -y python3
      changed_when: no

- hosts: rpi
  become: yes

  vars:
    prepi_hostname: smartgic-pi4b8-13
    prepi_cpu_freq: 2000
    prepi_cpu_governor: performance
    prepi_force_turbo: yes

  tasks:
    - import_role:
        name: smartgic.prepi
      tags:
        - prepi
        - raspberry
```

## License

MIT

## Author Information

I'm [Gaëtan Trellu (goldyfruit)](https://smartgic.io/), let's discuss :) - 2021
