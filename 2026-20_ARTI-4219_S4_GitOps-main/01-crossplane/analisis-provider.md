# =====================================================================
# Análisis del Provider PostgreSQL
# =====================================================================

## Provider: tages/provider-postgresql v0.1.0

### 1. Managed Resources disponibles

El provider expone los siguientes Managed Resources:

| API group | Kind | Propósito |
|---|---|---|
| `default.postgresql.upbound.io/v1alpha1` | `Privileges` | Gestiona privilegios predeterminados para objetos nuevos. |
| `grant.postgresql.upbound.io/v1alpha1` | `Role` | Define una relación entre un rol y los privilegios que puede conceder. |
| `physical.postgresql.upbound.io/v1alpha1` | `ReplicationSlot` | Gestiona un slot físico de replicación. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Database` | Crea y administra bases de datos PostgreSQL. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Extension` | Instala y configura extensiones de PostgreSQL. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Function` | Gestiona funciones almacenadas. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Grant` | Concede privilegios sobre bases de datos, esquemas u objetos. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Publication` | Gestiona publicaciones para replicación lógica. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Role` | Crea y administra roles o usuarios de PostgreSQL. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Schema` | Crea y administra esquemas. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Server` | Gestiona definiciones de servidores externos. |
| `postgresql.postgresql.upbound.io/v1alpha1` | `Subscription` | Gestiona suscripciones de replicación lógica. |
| `replication.postgresql.upbound.io/v1alpha1` | `Slot` | Gestiona un slot de replicación lógica. |
| `user.postgresql.upbound.io/v1alpha1` | `Mapping` | Asocia un usuario de PostgreSQL con un usuario externo. |

La mayoría de estos recursos son de ámbito de clúster y se asocian a un
`ProviderConfig` mediante `spec.providerConfigRef`.

### 2. Campos requeridos del recurso Database

El manifiesto usa la API `postgresql.postgresql.upbound.io/v1alpha1` y el
recurso `Database`. Dentro de `spec.forProvider`, el campo requerido es:

- `name` (`string`): nombre único de la base de datos en el servidor
	PostgreSQL. El CRD lo valida mediante una regla CEL.

Los campos opcionales son:

- `owner` (`string`): rol propietario de la base de datos. Si se omite, se
	utiliza el usuario que ejecuta la operación.
- `allowConnections` (`boolean`): permite o bloquea conexiones; por defecto
	se permiten.
- `connectionLimit` (`number`): límite de conexiones concurrentes; `-1`
	significa sin límite.
- `encoding` (`string`): codificación de caracteres, por ejemplo `UTF8`.
- `isTemplate` (`boolean`): indica si la base puede clonarse por usuarios con
	privilegio `CREATEDB`.
- `lcCollate` (`string`): orden de clasificación (`LC_COLLATE`).
- `lcCtype` (`string`): clasificación de caracteres (`LC_CTYPE`).
- `tablespaceName` (`string`): tablespace predeterminado de la base de datos.
- `template` (`string`): base de datos plantilla, normalmente `template0`.

Aunque el provider admite todos esos campos, esta PoC solo necesita pasar
`name` (y opcionalmente `owner`) desde la Composition.

### 3. Información requerida por el ProviderConfig

El `ProviderConfig` debe indicar cómo obtener las credenciales y apuntar al
Secret que contiene la conexión. En este proyecto se utiliza:

```yaml
apiVersion: postgresql.upbound.io/v1beta1
kind: ProviderConfig
metadata:
	name: postgresql-config
spec:
	credentials:
		source: Secret
		secretRef:
			namespace: crossplane-system
			name: postgresql-credentials
			key: connection
```

El valor de la clave `connection` debe ser un objeto JSON con la información
de conexión a PostgreSQL:

```json
{
	"host": "postgresql.postgresql.svc.cluster.local",
	"port": "5432",
	"username": "postgres",
	"password": "platform123",
	"database": "postgres",
	"sslmode": "disable"
}
```

Por tanto, son necesarios el host, el puerto, el usuario, la contraseña, la
base de datos inicial y el modo SSL. El nombre `postgresql-config` debe
coincidir con `spec.providerConfigRef.name` de cada Managed Resource o de la
Composition.
