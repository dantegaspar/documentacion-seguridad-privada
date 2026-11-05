# Convenciones, Reglas y Buenas Prácticas de Base de Datos (PostgreSQL)

Este documento define el estándar oficial para el diseño, modelado y mantenimiento de bases de datos en **PostgreSQL** dentro del proyecto. Su objetivo es garantizar la consistencia, el rendimiento, la integridad de los datos y la facilidad de mantenimiento a largo plazo.

---

## 1. Convenciones de Nomenclatura (Naming Conventions)

### 1.1 Reglas Generales
* **Formato (`snake_case`)**: Todos los identificadores (esquemas, tablas, columnas, índices, restricciones, funciones) deben escribirse en **minúsculas** separadas por guiones bajos (`_`).
  * *Por qué*: PostgreSQL convierte automáticamente a minúsculas cualquier identificador que no esté entre comillas dobles (`""`). Usar `camelCase` o `PascalCase` obliga a usar comillas en cada consulta.
* **Caracteres permitidos**: Solo letras (`a-z`), números (`0-9`) y guiones bajos (`_`). Deben comenzar siempre con una letra. No usar acentos, la letra `ñ` ni caracteres especiales.
* **Idioma**: Mantener un único idioma consistente en todo el modelo de dominio (por defecto **Español** para entidades del negocio y términos técnicos estándar aceptados como `id`, `uuid`, `status` o `created_at` según se acuerde; se recomienda **Español** para tablas/columnas de dominio y sufijos estándar). *Nota: Si el equipo prefiere 100% inglés o 100% español, debe aplicarse sin mezclar en una misma tabla.*
* **Evitar palabras reservadas**: Nunca usar palabras reservadas de SQL o PostgreSQL (`user`, `order`, `group`, `table`, `type`, `date`).
* **Longitud máxima**: Máximo **63 caracteres** (límite interno `NAMEDATALEN` de PostgreSQL).

### 1.2 Tablas y Vistas
* **Nombres en plural**: Las tablas representan colecciones de entidades, por lo que deben nombrarse en **plural** (`usuarios`, `guardias`, `turnos_vigilancia`, `incidentes`).
* **Tablas intermedias (Muchos a Muchos - N:M)**: Combinar el nombre de ambas tablas en plural o singular según legibilidad, preferiblemente en orden alfabético o jerárquico (`guardias_servicios`, `roles_permisos`).
* **Vistas**: Prefijar las vistas normales con `v_` y las vistas materializadas con `mv_` (ej. `v_guardias_activos`, `mv_resumen_asistencias_mensual`).

### 1.3 Columnas
* **Llave Primaria (PK)**: Llamar simplemente `id` en todas las tablas.
* **Llaves Foráneas (FK)**: Usar el nombre de la tabla referenciada en **singular** seguido de `_id` (ej. `guardia_id`, `cliente_id`, `sucursal_id`).
  * Si hay múltiples referencias a la misma tabla, agregar contexto semántico: `creado_por_usuario_id`, `aprobado_por_usuario_id`.
* **Booleanos**: Usar prefijos que denoten un estado verdadero/falso claro: `es_`, `tiene_`, `esta_`, `puede_` (o `is_`, `has_` si el esquema es en inglés). Ejemplo: `esta_activo`, `es_armado`, `tiene_licencia_vigente`.
* **Fechas y Timestamps**:
  * Para marcas de tiempo exactas (`TIMESTAMPTZ`): usar sufijo `_en` o `_at` (ej. `creado_en`, `actualizado_en`, `eliminado_en`).
  * Para fechas sin hora (`DATE`): usar prefijo `fecha_` (ej. `fecha_nacimiento`, `fecha_contratacion`, `fecha_vencimiento`).

### 1.4 Restricciones e Índices (Constraints & Indexes)
Nombrar explícitamente todas las restricciones e índices para facilitar la lectura de errores en los logs y las migraciones:

| Objeto | Patrón de Nombre | Ejemplo |
| :--- | :--- | :--- |
| **Primary Key** | `pk_<tabla>` | `pk_guardias` |
| **Foreign Key** | `fk_<tabla_origen>__<columna>` | `fk_turnos__guardia_id` |
| **Unique** | `uq_<tabla>__<columna(s)>` | `uq_usuarios__email` |
| **Check** | `ck_<tabla>__<descripcion>` | `ck_contratos__fechas_validas` |
| **Index (Normal)** | `idx_<tabla>__<columna(s)>` | `idx_incidentes__fecha_reporte` |
| **Index (Parcial/GIN)** | `idx_<tabla>__<col>_<tipo>` | `idx_guardias__activos_parcial` |

---

## 2. Tipos de Datos Recomendados

Elegir siempre el tipo de dato más específico y nativo de PostgreSQL.

| Caso de Uso | Tipo Recomendado | Tipo a EVITAR | Justificación |
| :--- | :--- | :--- | :--- |
| **Llaves Primarias (PK)** | `BIGINT GENERATED ALWAYS AS IDENTITY` o `UUID` (v7 preferido) | `SERIAL`, `INT`, `VARCHAR` | `IDENTITY` es el estándar SQL moderno. `UUIDv7` mantiene orden temporal y evita fragmentación de índices B-Tree. |
| **Textos y Cadenas** | `TEXT` (con `CHECK (char_length(col) <= N)` si requiere límite) o `VARCHAR(N)` | `CHAR(N)` | En Postgres, `TEXT` y `VARCHAR` tienen exactamente el mismo rendimiento. `CHAR(N)` desperdicia espacio rellenando con espacios en blanco. |
| **Fechas con Hora** | `TIMESTAMPTZ` (`TIMESTAMP WITH TIME ZONE`) | `TIMESTAMP` (sin zona horaria) | Siempre almacenar en UTC con zona horaria para evitar errores por cambios de horario o múltiples husos horarios. |
| **Dinero y Finanzas** | `NUMERIC(12, 2)` / `NUMERIC(15, 4)` | `FLOAT`, `REAL`, `MONEY` | Los tipos de punto flotante (`FLOAT`) sufren errores de redondeo binario. El tipo `MONEY` depende de la configuración regional (`lc_monetary`). |
| **Estados / Catálogos Fijos** | Tabla catálogo con FK o `TEXT` con `CHECK` | `ENUM` nativo (para listas cambiantes) | Los `ENUM` en Postgres no permiten eliminar o reordenar valores fácilmente sin recrear el tipo. |
| **Datos Semi-estructurados** | `JSONB` | `JSON` | `JSONB` se almacena en formato binario descompuesto, permite indexación (`GIN`) y búsquedas rápidas. |
| **Booleanos** | `BOOLEAN` | `SMALLINT` (`0`/`1`), `CHAR(1)` (`'S'`/`'N'`) | `BOOLEAN` es nativo, ocupa 1 byte y expresa intención clara (`true`, `false`, `null`). |

---

## 3. Estructura Base y Columnas de Auditoría

Toda tabla transaccional o de entidades principales debe incluir por defecto columnas de control y auditoría:

```sql
CREATE TABLE guardias (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    uuid UUID NOT NULL DEFAULT gen_random_uuid(),
    
    -- Columnas de negocio
    numero_empleado VARCHAR(30) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    apellido_paterno VARCHAR(100) NOT NULL,
    apellido_materno VARCHAR(100),
    esta_activo BOOLEAN NOT NULL DEFAULT TRUE,

    -- Columnas de auditoría estándar
    creado_en TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    actualizado_en TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    eliminado_en TIMESTAMPTZ, -- Solo si aplica Soft Delete
    creado_por_id BIGINT,
    actualizado_por_id BIGINT,

    CONSTRAINT pk_guardias PRIMARY KEY (id),
    CONSTRAINT uq_guardias__uuid UNIQUE (uuid),
    CONSTRAINT uq_guardias__numero_empleado UNIQUE (numero_empleado)
);
```

### Reglas sobre llaves primarias y exposición:
1. **Doble identificador (Opcional pero recomendado para APIs públicas)**: Usar `BIGINT` (`id`) para relaciones internas (JOINs rápidos y menor peso en índices) y una columna `uuid` (`UUID`) indexada para exponer recursos en URLs o APIs externas, evitando ataques de enumeración de IDs. Si se prefiere una sola llave primaria directa, usar **`UUID` (versión 7)**.
2. **Actualización automática de `actualizado_en`**: Implementar un *trigger* genérico reutilizable en la base de datos que actualice `actualizado_en = CURRENT_TIMESTAMP` en cada `BEFORE UPDATE`.

---

## 4. Reglas de Integridad y Diseño Relacional

1. **`NOT NULL` por defecto**: Todas las columnas deben definirse como `NOT NULL` a menos que exista una razón de negocio válida y documentada para permitir valores nulos.
2. **Integridad Referencial Obligatoria**: Toda relación entre tablas debe tener su respectivo `FOREIGN KEY`. No se permiten "relaciones lógicas" manejadas únicamente desde el código de la aplicación.
3. **Políticas de Borrado en FKs (`ON DELETE`)**:
   * Usar `ON DELETE RESTRICT` (o `NO ACTION`) por defecto para evitar borrados accidentales en cascada de información crítica (ej. no permitir borrar un cliente si tiene contratos o guardias asignados).
   * Usar `ON DELETE CASCADE` **únicamente** en tablas estrictamente dependientes (hijas débiles) que no tienen sentido sin el padre (ej. detalles de una bitácora o metadatos de una sesión).
4. **Borrado Lógico (Soft Delete) vs. Borrado Físico**:
   * Para entidades maestras y operativas (usuarios, guardias, clientes, incidentes, contratos), usar **Soft Delete** mediante la columna `eliminado_en TIMESTAMPTZ` (o `esta_activo BOOLEAN`).
   * Cuando se use `eliminado_en`, las restricciones `UNIQUE` que deban permitir reutilizar valores tras un borrado lógico deben implementarse como **Índices Únicos Parciales**:
     ```sql
     CREATE UNIQUE INDEX uq_usuarios__email_activo 
     ON usuarios (email) 
     WHERE eliminado_en IS NULL;
     ```
5. **Validaciones en Base de Datos (`CHECK`)**: No confiar solo en el backend. Agregar restricciones `CHECK` para reglas invariantes:
   * Rangos numéricos (`salario >= 0`, `calificacion BETWEEN 1 AND 5`).
   * Coherencia de fechas (`fecha_fin >= fecha_inicio`).
6. **Uso moderado de `JSONB`**: No usar `JSONB` para evadir el diseño relacional. Si un atributo se usa frecuentemente en cláusulas `WHERE`, `JOIN` o requiere integridad referencial, debe ser una columna o tabla relacional. Reservar `JSONB` para metadatos dinámicos, payloads de auditoría o configuraciones variables.

---

## 5. Estrategia de Indexación y Rendimiento

1. **Indexar todas las Llaves Foráneas (FK)**: PostgreSQL crea índices automáticamente para `PRIMARY KEY` y `UNIQUE`, pero **NO crea índices automáticamente en las columnas `FOREIGN KEY`**. Toda columna `*_id` debe tener su índice para evitar bloqueos y *Sequential Scans* en `JOIN`s y borrados.
2. **Índices Compuestos**: El orden de las columnas importa. Colocar primero las columnas evaluadas por igualdad (`=`) y al final las columnas evaluadas por rango (`>`, `<`, `BETWEEN`) o usadas en `ORDER BY`.
3. **Índices Parciales**: Si el 95% de las consultas filtran por registros activos (`WHERE esta_activo = TRUE` o `WHERE eliminado_en IS NULL`), crear índices parciales para ahorrar memoria RAM y disco.
4. **Tipos de Índice según necesidad**:
   * **B-Tree** (Default): Para igualdades, rangos y ordenamientos estándar.
   * **GIN**: Para búsquedas dentro de columnas `JSONB`, arreglos (`ARRAY`) o búsqueda de texto completo (*Full-Text Search*).
   * **BRIN**: Para tablas históricas masivas (logs, bitácoras de GPS, eventos) donde los datos se insertan en orden cronológico (`creado_en`).
5. **Evitar sobre-indexación**: Cada índice acelera las lecturas (`SELECT`) pero penaliza las escrituras (`INSERT`, `UPDATE`, `DELETE`). No crear índices "por si acaso".

---

## 6. Seguridad y Organización por Esquemas (Schemas)

1. **Separación por Esquemas (`Schemas`)**:
   * Evitar volcar cientos de tablas en el esquema `public` si el sistema crece en módulos bien diferenciados. Se puede organizar por dominios (ej. `auth`, `operaciones`, `rh`, `auditoria`) o mantener `public` limpio y separar únicamente `auditoria` / `catalogos`.
2. **Principio de Menor Privilegio**:
   * La aplicación **nunca** debe conectarse con el superusuario `postgres`.
   * Crear roles diferenciados:
     * Rol para migraciones (`app_migrator`): Propietario de las tablas (`DDL`: `CREATE`, `ALTER`, `DROP`).
     * Rol para la aplicación (`app_readwrite`): Solo permisos `DML` (`SELECT`, `INSERT`, `UPDATE`, `DELETE` — o incluso restringir `DELETE` en tablas de auditoría).
     * Rol de solo lectura (`app_readonly`): Para reportes o analítica.
3. **Datos Sensibles (PII)**:
   * Nunca almacenar contraseñas en texto plano (usar hashes robustos como Argon2id o bcrypt desde la aplicación).
   * Documentar explícitamente qué columnas contienen Información Personal Identificable (CURP, RFC, NSS, dirección, teléfonos) para controles de acceso y respaldos.

---

## 7. Documentación y Migraciones

1. **Comentarios en Base de Datos (`COMMENT ON`)**:
   * Toda tabla y toda columna cuyo propósito no sea 100% obvio debe incluir un comentario directo en el motor con `COMMENT ON`:
     ```sql
     COMMENT ON TABLE guardias IS 'Registro maestro del personal operativo de seguridad privada.';
     COMMENT ON COLUMN guardias.tipo_licencia_portacion IS 'Clasificación de permiso SEDENA para portación de arma.';
     ```
2. **Control de Versiones (Migraciones)**:
   * **Prohibido modificar la base de datos manualmente** en entornos compartidos (Dev, Staging, Prod).
   * Todo cambio estructural (`DDL`) o de catálogos semilla (`Seed Data`) debe realizarse mediante archivos de migración versionados en Git.
   * Cada migración debe ser **idempotente** o contar con su script de reversión (`up` / `down`) probado.
   * Al crear índices en tablas grandes en producción, utilizar `CREATE INDEX CONCURRENTLY` para no bloquear lecturas ni escrituras.
