
##  Atualização do GLPI

### 1. Local para salvar os backups ? > sugestão : pasta raiz do usuário root ou o usuário
utilizado na instalação
```bash
cd /root
```
### 2. Realizar backup do banco de dados(exportação)
```bash
mysqldump -u usuario -p nome_do_banco > nome_do_arquivo.sql
```
Será salvo no diretório atual

### 3. Parar serviço do apache(para evitar erros)
```bash
systemctl stop apache2
```
### 4. Renomear pasta do glpi atual para .OLD
```bash
cd /var/www/
```
```bash
mv glpi glpi.OLD
```

### 5. Baixar, descompactar e renomear pasta nova do GLPI
```bash
wget https://github.com/glpi-project/glpi/archive/refs/tags/10.0.11.zip
```
```bash
unzip 10.0.11.zip
```
```bash
mv glpi-10.0.11 glpi
```
### 6. Copiar somente o arquivo downstream para /inc
```bash
cp /var/www/glpi.OLD/inc/downstream.php /var/www/glpi/inc/downstream.php 
```
### 7. Copiar conteúdo necessário da pasta antiga
Para realizar isso utilizaremos o utilitário ‘mc’
```bash
apt install mc
```
digite no terminal : mc + Enter

No nosso caso copiaremos as pasta:

**plugins, pics, marketplace**

//Seguir a realização das cópias conforme mostrado em vídeo//
Por fim, realizar novamente comando para privilégios do usuário apache :
```bash
chown www-data:www-data /var/www/glpi -Rf
```
Após isso, reiniciar o apache : 
```bash
systemctl restart apache2
```
E seguir a atualização pelo link raiz do glpi(no navegador)

**glpi.set2024.com**

Pós atualização : remover pasta ‘install’ da pasta raiz

```bash
Rm -rf /var/www/glpi/Install/
```

### ***Fim***
