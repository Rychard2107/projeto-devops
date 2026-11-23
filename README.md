# Projeto 01 - DevOps com Vagrant e Ansible

- **Instituição:** IFPB - Campus João Pessoa
- **Disciplina:** Administração de Sistemas Abertos
- **Professor:** Leonidas Lima
- **Período:** 2026.2
- **Integrante:** Rychardson Zafler (matrícula 20251380026)

## Descrição

O projeto cria, com **Vagrant** (provider VirtualBox), quatro máquinas virtuais
Debian 12 (`debian/bookworm64`) e as configura automaticamente com **Ansible**,
seguindo a ideia de Infraestrutura como Código. O provisionador Ansible do
Vagrant executa `playbooks/site.yml` ao iniciar a VM `cli`, depois que as quatro
VMs já foram iniciadas.

| VM  | Função | IP (192.168.56.0/24) | Hostname |
|-----|--------|----------------------|----------|
| arq | Servidor de arquivos: DHCP, DNS, LVM, NFS | 192.168.56.126 (estático) | arq.rychardson.zafler.devops |
| db  | Banco de dados (MariaDB) + autofs | 192.168.56.127 (DHCP, reserva por MAC) | db.rychardson.zafler.devops |
| app | Aplicação (Apache) + autofs | 192.168.56.167 (DHCP, reserva por MAC) | app.rychardson.zafler.devops |
| cli | Cliente (Firefox, X11) + autofs | DHCP (faixa .50 a .100) | cli.rychardson.zafler.devops |

Domínio: `rychardson.zafler.devops`. Gateway: `192.168.56.1`.

### Vagrant

- Box `debian/bookworm64`, 512 MB de RAM (cli com 1024 MB), `linked_clone`,
  verificação de guest additions desabilitada e `insert_key = false`.
- A arq recebe 3 discos adicionais de 10 GB.
- Um gatilho (`trigger`) desabilita o servidor DHCP do VirtualBox; o DHCP da
  rede é fornecido pela VM arq.
- db e app recebem IP por DHCP com atribuição estática pelo MAC definido no Vagrantfile.
- Cada VM tem a porta SSH encaminhada para `127.0.0.1` (arq 2221, db 2222,
  app 2223, cli 2224), o que permite ao Ansible conectar antes de db/app/cli
  terem IP na rede privada.

### Ansible

| Playbook | O que faz |
|----------|-----------|
| `playbooks/01-comum.yml` | Todas as VMs: update/upgrade, chrony (pool.ntp.br), fuso America/Recife, grupo `ifpb`, usuários `rychardson` e `zafler`, geração das chaves SSH em `keys/`, SSH só com chave pública, root bloqueado, `AllowGroups vagrant ifpb`, banner, sudo para o grupo `ifpb`, cliente NFS |
| `playbooks/02-arq.yml` | LVM (VG `dados`, LV `ifpb` de 15 GB, ext4, montado em `/dados`), NFS (`/dados/nfs`, usuário `nfs-ifpb` sem shell, `all_squash`, `sync`), DHCP autoritativo (lease 180/3600 s, faixa .50 a .100, reservas de db e app), DNS BIND9 (zonas direta e reversa, forwarders 1.1.1.1 e 8.8.8.8) |
| `playbooks/03-rede-clientes.yml` | Faz db, app e cli pegarem IP no DHCP da arq |
| `playbooks/04-autofs.yml` | autofs monta `arq:/dados/nfs` em `/var/nfs` (db, app e cli) |
| `playbooks/05-db.yml` | Instala o `mariadb-server` |
| `playbooks/06-app.yml` | Instala o `apache2` e publica o `index.html` com a descrição do projeto e os dados do integrante |
| `playbooks/07-cli.yml` | Instala `firefox-esr` e `xauth` e habilita X11 forwarding no SSH |
| `playbooks/site.yml` | Executa todos os playbooks na ordem |

## Estrutura

```
projeto-devops/
├── Vagrantfile            # definição das 4 VMs e do gatilho DHCP
├── ansible.cfg            # configuração do Ansible
├── inventario.ini         # máquinas e grupos
├── requirements.yml       # coleções Ansible necessárias
├── group_vars/all.yml     # todas as variáveis (nomes, IPs, MACs, LVM, NFS)
├── keys/                  # chaves SSH geradas na execução (não versionadas)
└── playbooks/
    ├── site.yml
    ├── 01-comum.yml ... 07-cli.yml
    └── templates/         # chrony, sshd, dhcpd, exports, named, zonas, index.html
```

## Pré-requisitos

VirtualBox, Vagrant e Ansible (Linux, macOS ou WSL), com a rede host-only
`vboxnet0` em 192.168.56.1/24 e acesso à internet na primeira execução.

```bash
ansible-galaxy collection install -r requirements.yml
```

Se o Vagrant reclamar de `vm.disk`: `export VAGRANT_EXPERIMENTAL="disks"`.

## Execução

Execute a partir da raiz do projeto (onde estão `ansible.cfg` e `inventario.ini`):

```bash
vagrant up # sobe as 4 VMs e executa o playbook automaticamente
```

Para reaplicar a configuração depois que as VMs estiverem iniciadas:
`vagrant provision cli`.
Para executar uma etapa isolada ou validar a sintaxe, use Ansible diretamente,
por exemplo: `ansible-playbook playbooks/02-arq.yml` ou
`ansible-playbook playbooks/site.yml --syntax-check`.

## Acesso às máquinas

O playbook 01 gera as chaves SSH dos usuários em `keys/` (não versionadas). Acesso:

```bash
ssh -i keys/rychardson rychardson@192.168.56.126
ssh -i keys/zafler zafler@db.rychardson.zafler.devops
```

Para resolver os nomes no hospedeiro:

```bash
sudo resolvectl dns vboxnet0 192.168.56.126
sudo resolvectl domain vboxnet0 rychardson.zafler.devops
```

ou adicionar as entradas de arq, db e app em `/etc/hosts`.

A cli recebe IP por DHCP na faixa .50 a .100. Para acessá-la a partir do
hospedeiro, use a porta encaminhada (`ssh -p 2224 -i keys/rychardson rychardson@127.0.0.1`)
ou registre o IP dela no `/etc/hosts`.

## Verificação

```bash
dig @192.168.56.126 db.rychardson.zafler.devops +short   # DNS direto
dig @192.168.56.126 -x 192.168.56.167 +short             # DNS reverso
lsblk; sudo vgs; sudo lvs; df -h /dados                  # LVM (na arq)
sudo exportfs -v                                         # NFS (na arq)
ls /var/nfs; mount | grep nfs                            # autofs (db, app, cli)
curl http://app.rychardson.zafler.devops                 # Apache
ssh -X -p 2224 -i keys/rychardson rychardson@127.0.0.1 firefox-esr   # X11
timedatectl; chronyc sources                             # hora e fuso
```


## Observações

- O projeto foi desenvolvido individualmente; os logins `rychardson` e `zafler`
  correspondem ao nome e ao sobrenome do integrante.
- O IP do servidor app (192.168.56.167) segue a regra `1XY` do enunciado
  aplicada à matrícula do integrante.
- As chaves privadas em `keys/` nunca são enviadas ao repositório (`.gitignore`).
