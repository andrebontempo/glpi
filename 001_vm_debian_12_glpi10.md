## 1 Preparar VM linux com pacote web e banco de dados

### Instalar a VM Debian 12 e atualizar com o comando abaixo:
```bash
apt update && apt upgrade -y && apt full-upgrade -y && apt autoremove -y && apt clean
```
### Pacotes para manipular arquivos(até aqui é padrão):
```bash
   apt install -y xz-utils bzip2 unzip curl
```
## Instalar dependências no sistema(aqui começa a intalação específica do GLPi)
```bash
apt install -y apache2 libapache2-mod-php php-soap php-cas php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,bz2}
```
### Instalar o Serviço MySQL
```bash
apt install -y mariadb-server
```
### Criar Banco de Dados no Mysql
```bash
mysql -u root -p
```
```bash
 create database glpi10 character set utf8mb4 collate utf8mb4_bin;
```
```bash
create user connectdb@localhost identified by 'glpilab@udemy2024';
```
```bash
grant all privileges on glpi10.* to connectdb@localhost;
```
```bash
 flush privileges;
```
```bash
quit;
```
### Setar ipfixo e ativar acesso via ssh
```bash
apt install openssh-server
```
### Habilitar login root no ssh : 
```bash
nano /etc/ssh/sshd_config
```
Definir : 'PermitRootLogin yes'
Salvar e fechar : Ctrl + O, Enter, Ctrl + X
### Reiniciar o serviço : 
```bash
systemctl restart ssh
```
### Setar ip fixo na máquina : nano /etc/network/interfaces
Ps : ajuste para a faixa de ip, máscara etc da sua rede
```bash
iface "interface" inet static
address 192.168.1.80
netmask 255.255.255.0
network 192.168.1.0
broadcast 192.168.1.255
gateway 192.168.1.1
```
Salvar e fechar : Ctrl + O, Enter, Ctrl + X
### Reiniciar a vm : 
```bash
systemctl reboot
```
Por fim, validar se o acesso ssh continua OK
