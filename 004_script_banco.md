## 6 Script Backup Banco de Dados By Aristides Neto

Passo 1 : definir um local e criar 2 pastas(1 para o backup do banco e o a outra para os arquivos
de logs)

Sugestão : mkdir /backup_glpi
Dentro de /backup_glpi criar backups_bancos e logs_backups
mkdir backups_bancos
mkdir logs_backups

Passo 2 : Criar arquivo .sh com o script
nano scriptbackup.sh

conteúdo : acessar o link e copiar >>> https://aristides.dev/script-de-backup-de-banco-de-
dados-mysql/

Passo 3 : Editar as linhas de acordo com suas informações :
# Define usuario e senha do banco
USER='root'
PASS='root'
# Caminho do arquivo de log
LOG_DIR=/var/log/backup
# Diretorio onde serão salvos os backups
DIR_BK=/var/backups/database
# Lista dos bancos de dados que serão realizados o backup
DATABASES=(banco01 banco02)

Ctrl + O, Ctrl + X
Habilitar execução do arquivo : chmod +x scriptbackup.sh
Testar script : ./ scriptbackup.sh
Acessar a pasta do backup e verificar se funcionou

Passo 4 : Ativar rotina de backup na cron do Linux

nano /etc/crontab
30 22 * * * root /backup_glpi/scriptbackup.sh
Ctrl + O, Ctrl + X
systemctl restart cron

nesse exemplo o backup será realizado todos os dias às 22h30
Ajuste ou acrescente novos horários de acordo com sua necessidade

## ***Fim***



