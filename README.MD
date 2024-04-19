# Задание: PAM
Научиться создавать пользователей и добавлять им ограничения.

---
## Решение:

Создадим машину vagrant, при создании которой запускается ansible playbook, который устнанавливает docker, создаёт пользователей, запрещает одному из них (`otus`) подключение по ssh по пятницам и субботам ([скрипт](ansible/login.sh)), а второму (`otusadm`) разрешает запускать и пользоваться docker.

```
[andrej@home-srv OTUS-Task15]$ vagrant up
Bringing machine 'pam' up with 'virtualbox' provider...
==> pam: Importing base box 'ubuntu/jammy64'...
==> pam: Matching MAC address for NAT networking...
==> pam: Checking if box 'ubuntu/jammy64' version '1.0.0' is up to date...
==> pam: Setting the name of the VM: OTUS-Task15_pam_1713564437271_94675
==> pam: Clearing any previously set network interfaces...
==> pam: Preparing network interfaces based on configuration...
    pam: Adapter 1: nat
    pam: Adapter 2: hostonly
==> pam: Forwarding ports...
    pam: 22 (guest) => 2222 (host) (adapter 1)
==> pam: Running 'pre-boot' VM customizations...
==> pam: Booting VM...
==> pam: Waiting for machine to boot. This may take a few minutes...
    pam: SSH address: 127.0.0.1:2222
    pam: SSH username: vagrant
    pam: SSH auth method: private key
    pam: Warning: Connection reset. Retrying...
    pam: Warning: Remote connection disconnect. Retrying...
    pam: 
    pam: Vagrant insecure key detected. Vagrant will automatically replace
    pam: this with a newly generated keypair for better security.
    pam: 
    pam: Inserting generated public key within guest...
    pam: Removing insecure key from the guest if it's present...
    pam: Key inserted! Disconnecting and reconnecting using new SSH key...
==> pam: Machine booted and ready!
==> pam: Checking for guest additions in VM...
    pam: The guest additions on this VM do not match the installed version of
    pam: VirtualBox! In most cases this is fine, but in rare cases it can
    pam: prevent things such as shared folders from working properly. If you see
    pam: shared folder errors, please make sure the guest additions within the
    pam: virtual machine match the version of VirtualBox you have installed on
    pam: your host and reload your VM.
    pam: 
    pam: Guest Additions Version: 6.0.0 r127566
    pam: VirtualBox Version: 6.1
==> pam: Setting hostname...
==> pam: Configuring and enabling network interfaces...
==> pam: Running provisioner: shell...
    pam: Running: inline script
==> pam: Running provisioner: ansible...
    pam: Running ansible-playbook...

PLAY [PAM] *********************************************************************

TASK [Gathering Facts] *********************************************************
ok: [pam]

TASK [install service-packeges] ************************************************
changed: [pam]

TASK [add GPG-key] *************************************************************
changed: [pam]

TASK [add Docker repo] *********************************************************
changed: [pam]

TASK [install docker-ce] *******************************************************
changed: [pam]

TASK [Ensure group "admin" exists] *********************************************
ok: [pam]

TASK [Add the user 'otus'] *****************************************************
[WARNING]: The input password appears not to have been hashed. The 'password'
argument must be encrypted for this module to work properly.
changed: [pam]

TASK [Add the user 'otusadm'] **************************************************
changed: [pam]

TASK [Add the user root to group 'admin'] **************************************
changed: [pam]

TASK [Add the user vagrant to group 'admin'] ***********************************
changed: [pam]

TASK [Add the user otusadm to group 'docker'] **********************************
changed: [pam]

TASK [copy script] *************************************************************
changed: [pam]

TASK [configure PAM] ***********************************************************
changed: [pam]

TASK [reload sshd] *************************************************************
changed: [pam]

TASK [Allow systemctl docker to user 'otusadm'] ********************************
changed: [pam]

PLAY RECAP *********************************************************************
pam                        : ok=15   changed=13   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[andrej@home-srv OTUS-Task15]$ 

```

Подключаемся к ВМ.
Проверяем, что пользователи принадлежат группе `admin`
```bash
[andrej@home-srv OTUS-Task15]$ vagrant ssh
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-71-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Apr 19 22:12:28 UTC 2024

  System load:  0.18603515625     Users logged in:          0
  Usage of /:   5.6% of 38.70GB   IPv4 address for docker0: 172.17.0.1
  Memory usage: 28%               IPv4 address for enp0s3:  10.0.2.15
  Swap usage:   0%                IPv4 address for enp0s8:  192.168.57.10
  Processes:    110


Expanded Security Maintenance for Applications is not enabled.

191 updates can be applied immediately.
116 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Fri Apr 19 22:11:01 2024 from 192.168.57.1
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

vagrant@pam:~$ sudo -i
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@pam:~# cat /etc/group | grep admin
admin:x:118:otus,otusadm,root,vagrant
root@pam:~# 
```

Устанавливаем пароли пользователям.
```bash
admin:x:118:otus,otusadm,root,vagrant
root@pam:~# passwd otus
New password: 
Retype new password: 
passwd: password updated successfully
root@pam:~# passwd otusadm
New password: 
Retype new password: 
passwd: password updated successfully
```

Пробуем подключиться пользователем 'otusadm' и перезапустить docker.
```
[andrej@home-srv OTUS-Task15]$ ssh otusadm@192.168.57.10
The authenticity of host '192.168.57.10 (192.168.57.10)' can't be established.
ED25519 key fingerprint is SHA256:AC2y2krMsAidMEXqvDIgAu26CsJBhybWFVUTJ8zoTnY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.57.10' (ED25519) to the list of known hosts.
otusadm@192.168.57.10's password: 
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-71-generic x86_64)

...

otusadm@pam:~$ systemctl restart docker
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'docker.service'.
Multiple identities can be used for authentication:
 1.  Ubuntu (ubuntu)
 2.  otus
 3.  otusadm
 4.  ,,, (vagrant)
Choose identity to authenticate as (1-4): 3
Password: 
==== AUTHENTICATION COMPLETE ===
otusadm@pam:~$ 
```

Теперь пробуем подключиться пользователем 'otus' и перезапустить docker.
```
[andrej@home-srv OTUS-Task15]$ ssh otus@192.168.57.10
otus@192.168.57.10's password: 
ssh: Permission denied, please try again.
otus@192.168.57.10's password: 
```
