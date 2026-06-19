# Chanaltín Docs

Documentación publicada en GitHub Pages:

https://hfoc11.github.io/

Este repositorio contiene el contenido fuente del sitio MkDocs. Los cambios en `main` se publican automáticamente mediante GitHub Actions.

## Estructura

- `docs/` - páginas publicadas en el sitio.
- `mkdocs.yml` - configuración de navegación y tema.
- `.github/workflows/deploy-docs.yml` - despliegue a GitHub Pages.

## Desarrollo local

```bash
python -m pip install -r requirements.txt
python -m mkdocs serve
```
