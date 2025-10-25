# Grunt's Lab

Repositorio personal con apuntes de hacking, seguridad ofensiva, cloud, programación y más. El sitio se publica en [igruntplay.github.io](https://igruntplay.github.io) usando MkDocs + Material.

## Stack
- [MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- Markdown con extensiones `pymdownx`

## Entorno
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

En Windows PowerShell:
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Comandos útiles
- `mkdocs serve` levanta el sitio localmente en `http://127.0.0.1:8000` con recarga en vivo.
- `mkdocs build` genera el sitio estático en `site/`.
- `mkdocs gh-deploy --force` publica en la rama `gh-pages` (GitHub Pages).

## Estructura
```
mkdocs.yml       # configuración de navegación, tema y plugins
docs/            # contenido en Markdown, imágenes, labs, cheatsheets
```

## Contribuciones
Este repositorio es sobre todo para uso personal, pero si encontrás errores o querés sugerir mejoras, abrí un issue o PR 😉
