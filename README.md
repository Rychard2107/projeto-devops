# Projeto 01 - DevOps com Vagrant e Ansible

- **Instituição:** IFPB - Campus João Pessoa
- **Disciplina:** Administração de Sistemas Abertos
- **Professor:** Leonidas Lima
- **Grupo:** rychardson.zafler (domínio `rychardson.zafler.devops`)
- **Integrante:** Rychardson Zafler (matrícula 20251380026)

## Máquinas

| VM  | IP                          | Serviços                                  |
|-----|-----------------------------|-------------------------------------------|
| arq | 192.168.56.126 (fixo)       | DHCP, DNS (bind9), LVM, NFS               |
| db  | 192.168.56.127 (DHCP por MAC) | mariadb, autofs                         |
| app | 192.168.56.128 (DHCP por MAC) | apache2, autofs                         |
| cli | DHCP (faixa 50 a 100)       | firefox-esr, xauth, X11 via ssh, autofs   |

Todas recebem `playbooks/comum.yml` (update/upgrade, chrony, timezone, grupo `ifpb`, usuários, chaves ssh, sudo, sshd restrito).

## Execução

```
vagrant up          # arq sobe primeiro e passa a atender DHCP e DNS
```

O gatilho do Vagrantfile desliga o servidor DHCP do VirtualBox. As chaves dos usuários são geradas em `keys/`.
Ao terminar: `vagrant halt` e `VBoxManage dhcpserver start --interface vboxnet0`.

## Ajustes na máquina real

```
sudo resolvectl dns vboxnet0 192.168.56.126
sudo resolvectl domain vboxnet0 '~rychardson.zafler.devops'
echo '192.168.56.50 cli.rychardson.zafler.devops' | sudo tee -a /etc/hosts
```

## Teste

```
python3 teste.py rychardson.zafler
```
