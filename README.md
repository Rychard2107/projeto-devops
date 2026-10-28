# Projeto 01 - DevOps com Vagrant e Ansible

- Instituição: IFPB - Campus João Pessoa
- Disciplina: Administração de Sistemas Abertos
- Professor: Leonidas Lima
- Integrante: Rychardson Zafler (matrícula 20251380026)

## Descrição
Quatro VMs Debian 12 (arq, db, app, cli) criadas com Vagrant/VirtualBox e
configuradas com Ansible. A arq oferece DHCP, DNS, LVM e NFS; db tem MariaDB;
app tem Apache; cli tem Firefox e X11. db, app e cli montam o NFS via autofs.

## Execução
1. Instalar VirtualBox, Vagrant e Ansible
2. `ansible-galaxy collection install -r requirements.yml`
3. `vagrant up`
4. `ansible all -m ping`
5. `ansible-playbook site.yml`

## Estrutura
Vagrantfile, ansible.cfg, inventario.ini, group_vars/all.yml, site.yml,
playbooks 01 a 07 e templates/.

## Rede
arq 192.168.56.126 | db 192.168.56.127 | app 192.168.56.167 | cli via DHCP (.50 a .100)
