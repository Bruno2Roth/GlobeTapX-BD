# GlobeTapX — Base de datos

Este repositorio versiona el esquema PostgreSQL y las migraciones de Supabase de GlobeTapX.

## Archivos y migraciones

- `bd.sql`: snapshot del esquema para consultar el DDL. La integración de GitHub de Supabase no ejecuta este archivo; los cambios aplicables deben estar en `supabase/migrations/`.
- `supabase/migrations/20261003000000_initial_globetapx_schema.sql`: línea base con las tablas y restricciones iniciales.
- `supabase/migrations/20261004000000_normalize_country_docs_and_favorite_unique.sql`: consolida la documentación de países en `PaisInfo`, valida correspondencias antes de retirar tablas antiguas y evita favoritos duplicados por usuario y evento.
- `supabase/migrations/20261005000000_seed_globetapx_app_content.sql`: carga contenido compartido de países, categorías y eventos, migrado desde `completar.sql`. No crea cuentas de usuario.
- `supabase/config.toml`: configuración de Supabase CLI.

Las migraciones se aplican por timestamp. No edites ni vuelvas a ejecutar una migración ya aplicada para corregir una base existente: agrega una nueva migración con timestamp.

## Estado de GlobeTapX 2.0

Revisado el 6 de octubre de 2026. El historial del proyecto registra estas tres migraciones: la línea base, la normalización y la carga de contenido.

Hay un bloqueo pendiente en las claves primarias: las 14 tablas de la aplicación tienen su columna `ID` (o `id`, en `NumerosEmergenciaa`) sin valor predeterminado ni identidad. El esquema inicial crea secuencias, pero no las conecta a esas columnas para generar IDs automáticamente. Cuando el backend inserta un usuario sin enviar `ID`, PostgreSQL devuelve SQLSTATE `23502`; también se observaron fallos al guardar en `zLogCambios`.

Todavía no hay una migración correctiva en este repositorio. Por eso, actualizar este README no corrige el registro ni modifica la base.

### Corrección pendiente

1. Crear una migración nueva para la base actual que configure la generación automática de IDs en las 14 claves primarias. Antes de usar cada secuencia, sincronizarla con el mayor ID existente para evitar colisiones y conservar las filas actuales.
2. Corregir la migración inicial y el snapshot `bd.sql` para que las instalaciones nuevas nazcan con los IDs configurados.
3. Probar en una base local o de desarrollo: alta de usuario, escritura de `zLogCambios`, creación de otras filas y conservación de los datos previos.

No vuelvas a ejecutar `bd.sql` ni la migración inicial sobre GlobeTapX 2.0: el proyecto ya tiene datos y registra las tres migraciones aplicadas.

## Seguridad y rendimiento pendientes

- Las 14 tablas de la aplicación tienen RLS habilitado y no registran políticas. Si todo el acceso pasa por el backend, valida que la autorización se aplique allí y mantén cualquier clave de servicio solo en el servidor. Si el cliente consulta Supabase directamente, hacen falta políticas limitadas por tabla, operación y usuario; no abras el acceso general para resolver errores.
- El asesor de rendimiento encontró 13 columnas de claves foráneas sin índice. Añade índices después de revisar filtros y relaciones que usa la aplicación, evitando duplicar índices existentes.
- El README del proyecto describe `Estadisticas` como una relación 1 a 1 con `Usuario`, pero `Estadisticas.IDUsuario` no es único. Confirma esa regla y, si sigue vigente, revisa duplicados antes de agregar `UNIQUE`.
- `spatial_ref_sys` y las funciones de PostGIS requieren una revisión aparte de propiedad y permisos. No cambies sus permisos en bloque ni habilites RLS a ciegas sobre objetos administrados por la extensión.

## Conectar el repositorio a Supabase

En la integración de GitHub de Supabase, selecciona `Bruno2Roth/GlobeTapX-BD`, la rama `main` y el directorio de trabajo `.` (la carpeta `supabase/` está en la raíz). Si habilitas **Deploy to production**, los nuevos archivos de `supabase/migrations/` se aplican al proyecto de producción al subir o integrar cambios a la rama configurada. Editar solo `bd.sql` no despliega cambios.

La integración de GitHub y los despliegues con CLI están disponibles en todos los planes. La creación automática de ramas de vista previa para pull requests requiere Pro.

### Proyecto Supabase nuevo y vacío

Las migraciones del repositorio crean el esquema, normalizan las tablas y cargan contenido compartido. La tercera migración está en `supabase/migrations/`, por lo que forma parte de la secuencia; no contiene usuarios de prueba. En un entorno local enlazado, revisa el historial y aplica las migraciones pendientes:

```sh
supabase login
supabase link --project-ref <PROJECT_REF>
supabase migration list
supabase db push
```

Comprueba que el historial remoto coincida con los archivos de `supabase/migrations/`.

### Base existente con datos

Antes de aplicar migraciones, haz un backup y compara el esquema real con la línea base y el historial remoto.

El comando `supabase migration repair --status applied <VERSION>` solo actualiza el registro de migraciones; no ejecuta SQL. Úsalo únicamente cuando hayas confirmado que ese cambio ya existe en la base. No lo uses para marcar la línea base de GlobeTapX 2.0: ya registra las versiones `20261003000000`, `20261004000000` y `20261005000000`. La corrección de IDs debe llegar como una migración nueva.

No guardes contraseñas, claves de Supabase ni volcados con información de usuarios en GitHub.

## Documentación oficial

- [Integración de GitHub de Supabase](https://supabase.com/docs/guides/deployment/branching/github-integration)
- [Migraciones de base de datos](https://supabase.com/docs/guides/deployment/database-migrations)
- [Comando migration list](https://supabase.com/docs/reference/cli/supabase-migration-list)
