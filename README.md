rpi-hdmi-webcam-with-arducam-hawkeye
====================================

Set up a Raspberry Pi in combination with an Arducam 64MP Arducam Hawkeye camera to act as a HDMI webcam.

Thanks to https://medium.com/javarevisited/using-a-raspberry-pi-as-hdmi-camera-92af84aafee2 for the idea.

Steps to install hawkeye camera taken from https://docs.arducam.com/Raspberry-Pi-Camera/Native-camera/64MP-Hawkeye/

Steps to install pan-tilt demo taken from https://www.arducam.com/downloads/64mp-camera-pan-tilt-kit-manual.pdf and https://github.com/ArduCAM/PCA9685

Requirements
------------

A Raspberry Pi running RaspiOS with an Arducam Hawkeye camera attached to it.

I used
* a Raspberry Pi Zero 2 W: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/boards/raspberry-pi-zero-2-w?c=319
* an Arducam Hawkeye camera with pan-tilt kit: https://www.arducam.com/product/camera-pan-tilt-kit/
* connector cable: https://www.berrybase.de/raspberry-pi/raspberry-pi-computer/kabel-adapter/gpio-csi-dsi-kabel/flexkabel-f-252-r-raspberry-pi-zero-und-kameramodul?number=RPIZ-FLEX-15
* a 32GB micro SD card
* a gooseneck with table clamp: https://www.berrybase.de/neu/flexibles-schwanenhals-stativ-mit-tischklemme-40cm-schwarz?c=2407
* a Mini-HDMI to HDMI cable: https://www.amazon.de/dp/B09CKXL23S

This is how the assembled HDMI webcam looks like:

For instructions on creating a micro SD card for the operating system RaspiOS, refer to https://github.com/joschro/setting-up-raspberry-pi .

Dependencies
------------

Needs Ansible role ```joschro.rpi-hdmi-webcam-with-arducam-hawkeye``` from this repo (from Ansible Galaxy soon).

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

# https://github.com/ArduCAM/Arducam-Pivariety-V4L2-Driver states that "Since 5.15.38, the arducam-pivariety driver has been merged into the Raspberry Pi
# kernel and the name of the device tree is changed to arducam-pivariety, so dtoverlay=arducam-pivariety is required to set the overlay"
# with that, automatically updating to the latest kernel seems to be a bad idea
#    - name: Update packages
#      become: yes
#      package:
#        name: '*'
#        state: latest

    - name: Clone https://github.com/ArduCAM/PCA9685.git demo repository
      ansible.builtin.git:
        repo: 'https://github.com/ArduCAM/PCA9685.git'
        dest: /home/pi/PCA9685

    - name: Build the PCA9685 default target
      make:
        chdir: /home/pi/PCA9685

    - name: Unconditionally reboot the machine with all defaults
      ansible.builtin.reboot:
```

You can now run the following commands on the command line to start the installation:
```
sudo apt update
sudo apt install ansible

ansible-playbook -i localhost ansible-playbook-rpi-hdmi-webcam-hawkeye.yml
```

The system will reboot automatically after updating the operating system.

To test if the camera is detected correctly, run
```
v4l2-ctl --all
```

License
-------

GPLv3

Author Information
------------------

joschro
