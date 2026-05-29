## <div align="center"><strong>⚙️</strong></div> <div align="center"><strong>МОДУЛЬ 1</strong></div>

<br/>

<p align="center">
  <a href="https://github.com/kaktotad/demo-sys-2026/blob/main/topologia.png" target="_blank">
    <img src="https://github.com/crowcorpse/demo/blob/5a2dc610bab7da6d103fd0377c3c7c788dc3db87/images/topologia.png?raw=true" alt="Топология" width="50%"/>
  </a>
</p>

## 📚 Навигация по методичке

* [Задание №12 Samba](#samba)
* [Задание №13 RAID](#raid)
* [Задание №14 NFS](#nfs)
* [Задание №15 Ansible](#ansible)
* [Задание №16 Docker](#docker)
* [Задание №17 Web-приложение](#web)
* [Задание №18 Проброс портов](#проброс)
* [Задание №19 Обратный прокси](#прокси)

---

<a id="samba"></a>
## ✔️ 12. Samba

<details>
<summary>Решение</summary>

<h2><strong>BR-SRV</strong></h2>
Сначала:
```bash
apt install samba winbind -y
```

Затем:
```bash
rm -rf /etc/samba/smb.conf
samba-tool domain provision --use-rfc2307 --interactive
```

Теперь:
```bash
systemctl disable --now smbd winbind
systemctl mask smbd winbind
systemctl start samba
```

Группа и 5 пользователей:
```bash
samba-tool group create hq
for i in {1..5}; do
samba-tool user create hquser$i "P@ssw0rd"
samba-tool group addmembers hq hquser$i
done
```

<h2><strong>HQ-CLI</strong></h2>

```bash
apt install krb5-user -y
```

В качестве управляющего сервера необходимо добавить <code>br-srv.au-team.irpo</code>.

Далее:
```bash
realm join -U Administrator br-srv
```

Теперь надо получить билет:
```bash
kinit -k 'HQ-CLI$@AU-TEAM.IRPO'
```

Далее для группы прописывается правило:
```bash
echo "%hq@au-team.irpo ALL=(ALL) /usr/bin/cat, /usr/bin/grep, /usr/bin/id" > /etc/sudoers.d/hq
chmod 440 /etc/sudoers.d/hq
visudo -c
```

Обязательно попробовать войти под пользователем:
```bash
sudo pam-auth-update --enable mkhomedir
su - hquser1@au-team.irpo
```

<h2>Решение проблем</h2>

Если пишет <code>su: Системная ошибка:<code>

Необходимо сделать на BR-SRV:
```bash
samba-tool ntacl sysvolreset
```

Попробовать подключиться через smbclient напрямую для просмотра политик:
```bash
apt install smbclient -y
```
```bash
smbclient //br-srv/sysvol -k -c 'ls au-team.irpo/Policies'
```

Если ничего не выдало, попробовать:
```bash
kdestroy
kinit -k 'HQ-CLI$@AU-TEAM.IRPO'
```

И посмотреть тикет, в нем должны быть доменные правила:
```bash
klist -k /etc/krb5.keytab
```

Попробовать снова <code>smbclient</code>. Если получается - сделать первый шаг.

Если вообще ничего не выходит - перейти в файл /etc/sssd/sssd.conf и добавить следующую строку:
```bash
ad_gpo_access_mode = disabled
```

Это отключит проверку групповых политик, поэтому должно пустить.
</details>
