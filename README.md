# Blue Team Portfolio

Source for https://kamranSting.github.io, a MkDocs (Material theme) site with my home lab write-ups and study notes.

## Run locally

    pip install -r requirements.txt
    mkdocs serve

Open http://127.0.0.1:8000

## Deployment

A GitHub Action builds the site on every push to `main` and publishes it to the `gh-pages` branch, which GitHub Pages serves.

## Adding a write-up

Copy `docs/lab/pfsense-lab-network.md`, fill it in, and add it to `nav` in `mkdocs.yml`.
