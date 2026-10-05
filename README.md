# GlobeTapX — Base de datos

Este repositorio versiona el esquema PostgreSQL de GlobeTapX para Supabase.

## Archivos

- `bd.sql`: snapshot DDL del esquema corregido. No incluye datos de usuarios y sirve como referencia del estado final.
- `supabase/migrations/20261003000000_initial_globetapx_schema.sql`: línea base PostgreSQL anterior a las correcciones, para crear una instalación vacía.
- `supabase/migrations/20261004000000_normalize_country_docs_and_favorite_unique.sql`: migra la documentación de `PaisDocumentacion` y `DocumentacionPais` a `PaisInfo`, valida correspondencias antes de borrar las tablas antiguas, deduplica favoritos y agrega unicidad por usuario y evento.
- `supabase/config.toml`: configuración mínima de Supabase CLI.

## Conectar este repositorio a Supabase

En Supabase, conecta el proyecto con GitHub y selecciona `Bruno2Roth/GlobeTapX-BD) y la rama `main`. Activa **Deploy to production** para que Supabase aplique las nuevas migraciones cuando lleguen a esa rama. La creación automática de ramas de preview es opcional. Supabase procesa los archivos de `supabase/migrations`; editar sólo `bd.sql` no ejecuta cambios en la base.

### Si el proyecto Supabase ya tiene datos

No actives el despliegue de producción ni apliques `bd.sql` directamente sobre la base existente. Primero haz un backup y revisa la historia de migraciones y el esquema real.

Para una base que coincide con la línea base PostgreSQL anterior, enlaza el proyecto con Supabase CLI y marca esa línea base como ya aplicada para que no intente recrear tablas:

```sh
supabase login
supabase link --project-ref <PROJECT_REF>
supabase migration list
supabase migration repair --status applied 20261003000000
supabase db push
```

La orden `migration repair` sólo actualiza el registro de migraciones: **no ejecuta SQL**. Úsala únicamente después de confirmar que el esquema remoto coincide con la línea base. Si el esquema real difiere, primero hay que reconciliarlo con un dump del proyecto y ajustar la línea base; no marques la versión aplicada a ciegas.

La migración de normalización aborta antes de borrar las tablas antiguas si encuentra documentación sin un país correspondiente o varias filas que colisionan con el mismo país. Después de verificar y aplicar la migración, activa **Deploy to production**. A partir de ahí, los cambios de esquema nuevos deben agregarse como archivos con timestamp en `supabase/migrations/`.

## Instalación vacía

En un proyecto Supabase vacío, las migraciones se aplican en orden: primero la línea base y después la normalización. Este repositorio no carga datos iniciales ni datos de usuarios automáticamente.

No guardes contraseñas, tokens de Supabase ni volcados con información de usuarios en GitHub.
