#### 1. Instalar MySQL en ambas maquinas
- `sudo apt install mysql`

#### 2. Configurar en ambas Maquinas Virtuales 
- `sudo nano /etc/mysql/mysql.conf.d/mysqld.c`
- Maestro 
  'bind-address = 0.0.0.0'
  'server-id = 1'
  'log_bin = /var/log/mysql/mysql-bin.log'
