#### 1. Instalar MySQL en ambas maquinas
- `sudo apt update`
- `sudo apt install mysql-server -y`

#### 2. Editar en ambas Maquinas Virtuales el siguiente archivo
- `sudo nano /etc/mysql/mysql.conf.d/mysqld.c`
- Maestro 
  bind-address = 0.0.0.0'
  server-id = 1'
  log_bin = /var/log/mysql/mysql-bin.log'

- Esclavo 
   bind-address = 0.0.0.0
   server-id = 2

Reiniciar MySQL en ambas maquinas 
- `sudo systemctl restart mysql'

#### 3. Crear usuario replicación en el maestro

CREATE USER 'replicator'@'%' IDENTIFIED BY 'claveSegura';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
FLUSH PRIVILEGES;

- Verificamos el estado del binlog
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;

#### 4. Exportar la base de datos
mysqldump -u root -p --databases Hospital --master-data=2 > hospital.sql

Copia  a la maquina virtual del esclavo 
`scp hospital.sql usuario@IP_ESCLAVO:/tmp/`

#### 5. Importar la base de datos en el esclavo
`mysql -u root -p < /tmp/hospital.sql`


#### 6.Obtener File y Position del maestro
SHOW MASTER STATUS;

#### 7. Configuramos la replicación en el esclavo
CHANGE MASTER TO
  MASTER_HOST='IP_MAESTRO',
  MASTER_USER='replicator',
  MASTER_PASSWORD='claveSegura',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=XXX;

START SLAVE;

Verificamos su estado 

`SHOW SLAVE STATUS\G`

#### 8. Probar replicación

Insertar en el Maestro 
INSERT INTO CentrosMedicos (Nombre, Ciudad, Direccion, Telefono)
VALUES ('Clinica GCP', 'Guayaquil', 'Av. 9 de Octubre', '0933456789');

Verificar en el esclavo: 
SELECT * FROM CentrosMedicos;
