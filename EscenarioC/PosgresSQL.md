#### 1. Instalar PostgressSQL en ambas maquinas
- `sudo apt update`
- `sudo apt install postgresql postgresql-client -y`

#### 2. Crear la base de datos en el maestro 
- `sudo -u posgres psql`

-- Tabla de centros médicos
CREATE TABLE CentrosMedicos (
    CentroID SERIAL PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Ciudad VARCHAR(100) NOT NULL,
    Direccion VARCHAR(200),
    Telefono VARCHAR(20)
);

INSERT INTO CentrosMedicos (Nombre, Ciudad, Direccion, Telefono)
VALUES ('Hospital Central', 'Quito', 'Av. Amazonas 1234', '022345678');

-- Tabla de usuarios del centro
CREATE TABLE UsuariosCentro (
    UsuarioID SERIAL PRIMARY KEY,
    CentroID INT NOT NULL,
    Email VARCHAR(75) NOT NULL UNIQUE,
    Contrasena VARCHAR(75) NOT NULL,
    FOREIGN KEY (CentroID) REFERENCES CentrosMedicos(CentroID)
);

INSERT INTO UsuariosCentro (CentroID, Email, Contrasena)
VALUES (1, 'noemi@ecentroguayaquil.com', '2003');

-- Tabla de especialidades
CREATE TABLE Especialidades (
    EspecialidadID SERIAL PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Descripcion VARCHAR(255)
);

-- Tabla de médicos
CREATE TABLE Medicos (
    MedicoID SERIAL PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Apellido VARCHAR(100) NOT NULL,
    EspecialidadID INT NOT NULL,
    CentroID INT NOT NULL,
    Email VARCHAR(100),
    Telefono VARCHAR(20),
    FOREIGN KEY (EspecialidadID) REFERENCES Especialidades(EspecialidadID),
    FOREIGN KEY (CentroID) REFERENCES CentrosMedicos(CentroID)
);

-- Tabla de asignación de especialidades
CREATE TABLE AsignacionEspecialidades (
    AsignacionID SERIAL PRIMARY KEY,
    MedicoID INT NOT NULL,
    EspecialidadID INT NOT NULL,
    FOREIGN KEY (MedicoID) REFERENCES Medicos(MedicoID),
    FOREIGN KEY (EspecialidadID) REFERENCES Especialidades(EspecialidadID)
);

-- Tabla de empleados
CREATE TABLE Empleados (
    EmpleadoID SERIAL PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Apellido VARCHAR(100) NOT NULL,
    Cargo VARCHAR(100),
    Email VARCHAR(100),
    Telefono VARCHAR(20)
);

-- Tabla de clientes
CREATE TABLE Clientes (
    ClienteID SERIAL PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Apellido VARCHAR(100) NOT NULL,
    Correo VARCHAR(100),
    Telefono VARCHAR(20)
);

#### 3. Configurar el archivo postgresql.conf en el maestro 
- Editar el archivo `postgresql.conf` para permitir conexiones remotas:
- `sudo vi /etc/postgresql/14/main/postgresql.conf`
--	iConfigurar lo siguiente
listen_addresses = '*'
wal_level = replica
max_wal_senders = 10
wal_keep_size = 64
synchronous_commit = on
synchronous_standby_names = 'standby1'

#### 4. Permitir conexión del esclavo (esto se hace en el maestro)
Ingresamos a este archivo 
- `sudo vi /etc/postgresql/14/main/pg_hba.conf`

Agregamos esta línea al final del documento
- host    replication     replicador     10.128.0.5/32        md5

#### 5. Crear el usuario replicador en el maestro
- `sudo -u postgres psql`
- `CREATE ROLE replicador WITH REPLICATION LOGIN ENCRYPTED PASSWORD '2003';`


#### 6. Reiniciamos el PostgreSQL
- `sudo systemctl restart postgresql`

#### 7. Preparamos el esclavo, vaciamos data y hacemos backup
Primereo detenemos el servicio en el esclavo y elimínanos la data
- `sudo systemctl stop postgresql`
- `sudo rm -rf /var/lib/postgresql/14/main/*`

Ahora ejecutamos el pg_basebackup  desde el esclavo: 
- `sudo -i -u postgres pg_basebackup -h 10.128.0.4 -D /var/lib/postgresql/16/main -U replicador -Fp -Xs -P -R`

#### 8. Configuramos el archivo postgresql.conf en el esclavo
Ingresamos a este archivo
- `sudo vi /etc/postgresql/14/main/postgresql.conf`
Cambiamos lo siguiente:
- primary_conninfo = 'host=10.128.0.4 port=5432 user=replicador password=2003'
- primary_slot_name = 'standby1'

#### 9. Iniciamos al esclavo
- `sudo systemctl start postgresql`
Verificamos el estado 
- `sudo -u postgres psql -c "SELECT * FROM pg_stat_wal_receiver;"`
#### 10. Verificamos la replicación
Vamos a conectarnos desde el maestro, y insertamos un dato:
- `sudo -u postgres psql -d hospital`
-INSERT INTO CentrosMedicos (Nombre, Ciudad, Direccion, Telefono)
VALUES ('Hospital Central', 'Quito', 'Av. Amazonas 1234', '022345678');

En el maestro, verificamos que el esclavo esté conectado:

- `sudo -u postgres psql -c "SELECT * FROM pg_stat_replication;"`s