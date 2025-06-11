# PruebaBD

Este proyecto contiene la implementación de distintos **esquemas de replicación de bases de datos** utilizando varios sistemas gestores, como parte del laboratorio práctico en Google Cloud. Se trabaja con una base de datos médica de ejemplo para simular diferentes estrategias de replicación.

## 📚 Descripción general

La base de datos contiene las siguientes tablas:

- `CentrosMedicos`: Información básica sobre centros hospitalarios.
- `UsuariosCentro`: Accesos asociados a cada centro.
- `Especialidades`: Lista de especialidades médicas.
- `Medicos`: Médicos y su especialidad.
- `AsignacionEspecialidades`: Relación entre médicos y especialidades.
- `Empleados`: Personal administrativo y de apoyo.
- `Clientes`: Pacientes registrados.

Esta base se replica utilizando diferentes métodos, adaptados a distintos motores y topologías.

## 🧪 Escenarios de replicación implementados

### 🔵 Escenario A: Replicación Activa/Pasiva (MySQL)
Se implementa una replicación maestro-esclavo (source → replica) usando MySQL con binlogs y un usuario con permisos de `REPLICATION SLAVE`. Ideal para alta disponibilidad y recuperación ante fallos.

### 🟠 Escenario B: Replicación Snapshot, Transaccional y Merge (SQL Server)
Se configuran tres tipos de replicación:
- **Snapshot**: toma una copia completa de los datos en un momento específico.
- **Transaccional**: replica los cambios conforme se producen.
- **Merge**: permite que tanto el publicador como el suscriptor hagan cambios, con reconciliación posterior.

### 🟢 Escenario C: Replicación Síncrona (PostgreSQL / Cloud Spanner)
Se configura replicación síncrona en PostgreSQL o se utiliza Cloud Spanner con consistencia fuerte. En PostgreSQL se usan archivos WAL, `pg_basebackup` y `replication slots` para mantener la sincronización en tiempo real entre el nodo principal y el secundario.

### 🔴 Escenario D: Réplica Set (MongoDB)
Implementación de un conjunto de réplicas en MongoDB donde hay un nodo primario y varios secundarios. Proporciona alta disponibilidad, failover automático y escalabilidad en lectura.

### 🟣 Escenario E: Replicación Heterogénea (MySQL → BigQuery / PostgreSQL → Pub/Sub)
Conexión de motores diferentes mediante herramientas como `Debezium`, `Dataflow`, `Cloud SQL Federation` o `Cloud Pub/Sub`. Útil para análisis en tiempo real o integración de datos en distintos servicios de Google Cloud.




## ☁️ Requisitos generales

- Google Cloud Platform (GCE y SQL Server VM)
- SQL Server Management Studio / MySQL CLI / pgAdmin / Compass
- Credenciales con permisos adecuados de replicación
- Conexión SSH entre instancias (cuando aplique)

---

---

## 🏥 Base de datos utilizada: HOSPITAL

```sql
-- Tabla de centros médicos
CREATE TABLE CentrosMedicos (
    CentroID INT PRIMARY KEY AUTO_INCREMENT,
    Nombre VARCHAR(100) NOT NULL,
    Ciudad VARCHAR(100) NOT NULL,
    Direccion VARCHAR(200),
    Telefono VARCHAR(20)
);

INSERT INTO CentrosMedicos (Nombre, Ciudad, Direccion, Telefono)
VALUES ('Hospital Central', 'Quito', 'Av. Amazonas 1234', '022345678');

-- Tabla de usuarios por centro
CREATE TABLE UsuariosCentro (
    UsuarioID INT PRIMARY KEY AUTO_INCREMENT,
    CentroID INT NOT NULL,
    Email VARCHAR(75) NOT NULL UNIQUE,
    Contrasena VARCHAR(75) NOT NULL,
    FOREIGN KEY (CentroID) REFERENCES CentrosMedicos(CentroID)
);

INSERT INTO UsuariosCentro (CentroID, Email, Contrasena)
VALUES (1, 'noemi@ecentroguayaquil.com', '2003');

-- Tabla de especialidades
CREATE TABLE Especialidades (
    EspecialidadID INT PRIMARY KEY AUTO_INCREMENT,
    Nombre VARCHAR(100) NOT NULL,
    Descripcion VARCHAR(255)
);

-- Tabla de médicos
CREATE TABLE Medicos (
    MedicoID INT PRIMARY KEY AUTO_INCREMENT,
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
    AsignacionID INT PRIMARY KEY AUTO_INCREMENT,
    MedicoID INT NOT NULL,
    EspecialidadID INT NOT NULL,
    FOREIGN KEY (MedicoID) REFERENCES Medicos(MedicoID),
    FOREIGN KEY (EspecialidadID) REFERENCES Especialidades(EspecialidadID)
);

-- Tabla de empleados
CREATE TABLE Empleados (
    EmpleadoID INT PRIMARY KEY AUTO_INCREMENT,
    Nombre VARCHAR(100) NOT NULL,
    Apellido VARCHAR(100) NOT NULL,
    Cargo VARCHAR(100),
    Email VARCHAR(100),
    Telefono VARCHAR(20)
);

-- Tabla de clientes
CREATE TABLE Clientes (
    ClienteID INT PRIMARY KEY AUTO_INCREMENT,
    Nombre VARCHAR(100) NOT NULL,
    Apellido VARCHAR(100) NOT NULL,
    Correo VARCHAR(100),
    Telefono VARCHAR(20)
);


