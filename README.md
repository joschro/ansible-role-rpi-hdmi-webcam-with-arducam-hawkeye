rpi-hdmi-webcam
============================

Set up a Raspberry Pi in combination with an Arducam 64MP Arducam Hawkeye camera to act as a HDMI webcam.

Thanks to https://medium.com/javarevisited/using-a-raspberry-pi-as-hdmi-camera-92af84aafee2 for the idea.

Requirements
------------

An RaspiOS installed Raspberry Pi with a Raspberry Pi camera attached to it.

I used
* a Raspberry Pi Zero 2 W: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/boards/raspberry-pi-zero-2-w?c=319
* a Raspberry Pi High Quality camera: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/kameras/raspberry-pi-high-quality-kamera?c=341
  plus a wide-angle lens: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/kameras/6mm-weitwinkelobjektiv-cs-mount?c=341
* a camera mount: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/gehaeuse/gehaeuse-halter-fuer-kameramodul/pro-mounting-plate-f-252-r-high-quality-camera-und-raspberry-pi-zero?c=329
* connector cable: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/kabel-adapter/gpio-csi-dsi-kabel/flexkabel-f-252-r-raspberry-pi-zero-und-kameramodul?number=RPIZ-FLEX-15
* an old 4GB micro SD card
* a gooseneck with table clamp: https://www.berrybase.de/neu/flexibles-schwanenhals-stativ-mit-tischklemme-40cm-schwarz?c=2407
* a Mini-HDMI to HDMI cable

This is how the assembled HDMI webcam looks like:

| ![GOPR2564](https://user-images.githubusercontent.com/12337748/156064537-88d74503-3181-46ec-85c7-d7ed30516b0c.jpg) | ![GOPR2565](https://user-images.githubusercontent.com/12337748/156064572-9f8a8dea-663f-42f4-9fae-ec8e7af7ad8f.jpg) | ![GOPR2568](https://user-images.githubusercontent.com/12337748/156064607-f25f4fa1-00a4-4abb-a4be-72a0b9df3b04.jpg) |
|:-:|:-:|:-:|

The tripod and GoPro adapters in the last picture are only there for making these pictures and not the final setup.

For instructions on creating a micro SD card for the operating system RaspiOS, refer to https://github.com/joschro/setting-up-raspberry-pi .

Dependencies
------------

Out-of-the-box support on RaspiOS "Bullseye". "Buster" does not work out-of-the-box.
Needs Ansible role ```joschro.rpi-hdmi-webcam``` from this repo (from Ansible Galaxy soon).

Example Playbook
----------------
Create a file called ```ansible-playbook-rpi-hdmi-webcam-hawkeye.yml``` in the *pi* user's directory on the Raspberry Pi with following content:
```
---
# Ansible Playbook to set up a Raspberry Pi with Arducam Hawkeye camera as HDMI webcam
- name: Set up a Raspberry Pi HDMI webcam
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Make sure that an empty requirements.yml file exists
      file:
        path: requirements.yml
        state: touch

    - name: Create requirements.yml
      blockinfile:
        path: requirements.yml
        create: yes
        block: |
          # Install a role from the Ansible Galaxy
          #- src: joschro.rpi-hdmi-webcam
          
          # Install a role from GitHub
          - src: https://github.com/joschro/ansible-role-rpi-hdmi-webcam-with-arducam-hawkeye
            name: joschro.rpi-hdmi-webcam-hawkeye

    - name: Install required packages
      become: yes
      package:
        name: git
        state: present

    - name: Source required roles
      command: ansible-galaxy install -r requirements.yml
      # (add parameter --force to update role on succeeding runs of playbook)

    - name: Execute role
      include_role:
        name: joschro.rpi-hdmi-webcam-hawkeye
```

You can now run the following commands on the command line to start the installation:
```
sudo apt update
sudo apt install ansible

ansible-playbook -i localhost ansible-playbook-rpi-hdmi-webcam-hawkeye.yml
```

You may want to update the complete operating system by running
```
sudo apt upgrade
```
and reboot afterwards.

License
-------

GPLv3

Author Information
------------------

joschro
