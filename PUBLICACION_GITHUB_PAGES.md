# Publicacion correcta en GitHub Pages

Este archivo resume lo que se hizo para publicar correctamente la documentacion de Chanaltin y deja una guia para futuras IAs o automatizaciones.

## Estado final

- Repositorio fuente de documentacion: `https://github.com/hfoc11/chanaltin-docs`
- Repositorio publicado como GitHub Pages raiz: `https://github.com/hfoc11/hfoc11.github.io`
- Sitio publico final: `https://hfoc11.github.io/`
- Ejemplo de ruta valida: `https://hfoc11.github.io/specs/trip-scheduling/`

El sitio se publica desde el repositorio especial `hfoc11.github.io`, no desde la ruta de proyecto `hfoc11.github.io/chanaltin-docs/`. Esto evita rutas largas y permite enlaces como `/specs/trip-scheduling/`.

## Cambios realizados

- Se organizo la documentacion MkDocs en paginas independientes bajo `docs/specs/`.
- Se actualizo `mkdocs.yml` para crear submenus bajo `Especificaciones`.
- Se configuro `site_url: https://hfoc11.github.io/` para que los assets y enlaces canonicos apunten a la raiz correcta.
- Se publico el sitio estatico generado por MkDocs en el repo `hfoc11/hfoc11.github.io`.
- Se verifico que el home, la pagina de ejemplo y el CSS respondan correctamente:
  - `https://hfoc11.github.io/`
  - `https://hfoc11.github.io/specs/trip-scheduling/`
  - `https://hfoc11.github.io/assets/stylesheets/...`

## Estructura fuente importante

- `mkdocs.yml`: configuracion del sitio, navegacion y `site_url`.
- `docs/index.md`: pagina inicial.
- `docs/specs/index.md`: indice de especificaciones.
- `docs/specs/*.md`: paginas por modulo funcional.
- `docs/diagrams/architecture.md`: diagramas.
- `requirements.txt`: dependencias para construir el sitio.

## Modulos Odoo involucrados

Para futuras IAs: el desarrollo funcional documentado esta relacionado con Odoo 18.0 y debe tomar como referencia estos modulos nativos y enterprise instalados localmente.

### Rutas base

- `D:\ODOO\odoo-18.0\addons`
- `D:\ODOO\odoo-18.0\enterprise\enterprise\addons`

### Modulos especificos

- `D:\ODOO\odoo-18.0\addons\fleet` - flota, vehiculos, unidades y datos relacionados con placas.
- `D:\ODOO\odoo-18.0\addons\hr` - empleados, conductores y datos del personal.
- `D:\ODOO\odoo-18.0\addons\maintenance` - mantenimiento preventivo/correctivo y bloqueos operativos.
- `D:\ODOO\odoo-18.0\enterprise\enterprise\addons\industry_fsm` - servicio externo / Field Service.
- `D:\ODOO\odoo-18.0\enterprise\enterprise\addons\industry_fsm_repair` - reparaciones vinculadas a servicio externo.
- `D:\ODOO\odoo-18.0\addons\sale` - ventas, ordenes de cliente y flujo comercial.
- `D:\ODOO\odoo-18.0\addons\purchase` - compras, ordenes de servicio/proveedor y costos de terceros.
- `D:\ODOO\odoo-18.0\enterprise\enterprise\addons\account_accountant` - contabilidad avanzada.

Estos modulos deben revisarse antes de proponer modelos nuevos, porque muchos objetos de negocio ya existen en Odoo y conviene extenderlos en lugar de duplicarlos.

## Comandos seguros para validar

Desde la raiz del repo `chanaltin-docs`:

```powershell
.\.venv\Scripts\python.exe -m mkdocs build --strict
```

Si no existe `.venv`, crear entorno e instalar dependencias:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m mkdocs build --strict
```

## Publicacion manual correcta

Publicar el sitio generado en el repositorio raiz de GitHub Pages:

```powershell
git remote add root-pages https://github.com/hfoc11/hfoc11.github.io.git
.\.venv\Scripts\python.exe -m mkdocs gh-deploy --clean --remote-name root-pages --remote-branch main --force
git remote remove root-pages
```

Despues de publicar, verificar:

```powershell
Invoke-WebRequest -Uri "https://hfoc11.github.io/" -UseBasicParsing
Invoke-WebRequest -Uri "https://hfoc11.github.io/specs/trip-scheduling/" -UseBasicParsing
```

## Precauciones importantes

- No publicar solo en `hfoc11/chanaltin-docs` si se espera que funcionen URLs como `https://hfoc11.github.io/specs/...`.
- No cambiar `site_url` a `https://hfoc11.github.io/chanaltin-docs/` mientras el sitio principal se sirva desde `hfoc11.github.io`.
- No commitear `.venv/`, `site/` ni `logs/`; deben quedar ignorados por `.gitignore`.
- Despues de ejecutar `mkdocs gh-deploy`, revisar `git status` antes de commitear. Ese comando puede dejar el workspace en un estado inesperado si se usa sobre el mismo checkout.
- Si el estilo del home se pierde, revisar que los CSS bajo `/assets/stylesheets/` respondan con HTTP 200.

## Verificacion final esperada

El resultado correcto debe ser:

```text
home=200
spec=200
css=200
title=Chanaltin Docs
```
