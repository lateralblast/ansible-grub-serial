![alt tag](https://raw.githubusercontent.com/lateralblast/ansible-grub-serial/master/images/cat_cable.jpg)

Automating GRUB serial configuration with Ansible
=================================================

Introduction
------------

This is a quick example of how to automate GRUB serial configuration with Ansible

Requirements
------------

Required software:

- Ansible
- setserial packages

Overview
--------

This example does the following:

- Run setserial to get the serial port details
- Update GRUB configuration file if required
- Update GRUB if required

Run setserial to get the serial port details:

```
- name: Check Serial TTY
  shell: |
         set -o pipefail
         /bin/setserial -g /dev/ttyS[0123456] |grep -v unknown |tail -1 |cut -f1 -d, |cut -f3 -d/
  args:
    executable: /bin/bash
  register:  serial_tty

- name: Check Serial Port
  shell: |
         set -o pipefail
         /bin/setserial -g /dev/ttyS[0123456] |grep -v unknown |tail -1 |cut -f3 -d, |awk '{print $2}'
  args:
    executable: /bin/bash
  register: serial_port
```

Update GRUB configuration file if required:

```
- name: Check GRUB
  lineinfile:
    path:   /etc/default/grub
    regex:  "^{{ item.regex }}"
    line:   "{{ item.line }}"
  notify: update_grub
  loop:
    - { regex: "GRUB_DEFAULT",          line: 'GRUB_DEFAULT=0' }
    - { regex: "GRUB_TIMEOUT_STYLE",    line: 'GRUB_TIMEOUT_STYLE=menu' }
    - { regex: "GRUB_TIMEOUT",          line: 'GRUB_TIMEOUT=5' }
    - { regex: "GRUB_DISTRIBUTOR",      line: 'GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`' }
    - { regex: "GRUB_SERIAL_COMMAND",   line: 'GRUB_SERIAL_COMMAND="serial --speed=115200 --port={{ serial_port.stdout }}"' }
    - { regex: "GRUB_CMDLINE_LINUX",    line: 'GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevnames=0 console=tty0 console={{ serial_tty.stdout }},115200 audit=1"' }
    - { regex: "GRUB_TERMINAL_INPUT",   line: 'GRUB_TERMINAL_INPUT="console serial"' }
    - { regex: "GRUB_TERMINAL_OUTPUT",  line: 'GRUB_TERMINAL_OUTPUT="console serial"' }
    - { regex: "GRUB_GFXPAYLOAD_LINUX", line: 'GRUB_GFXPAYLOAD_LINUX=text' }
```

Update GRUB if required:

```
- name: update_grub
  command:     /usr/sbin/update-grub
  become:      yes
  become_user: root
```
