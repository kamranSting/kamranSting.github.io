# Blue Team Portfolio

## Run locally
    pip install -r requirements.txt
    mkdocs serve
Open http://127.0.0.1:8000

## Publish
1. Create a public GitHub repo named kamranSting.github.io
2. Add your LinkedIn URL in docs/about.md
3. Push to main. The GitHub Action builds the site into a gh-pages branch.
4. Repo Settings > Pages > Source: Deploy from a branch > gh-pages / root
5. Site is live at https://kamranSting.github.io

## New write-up
Copy docs/lab/pfsense-segmentation.md, fill it in, add it to `nav` in mkdocs.yml.
