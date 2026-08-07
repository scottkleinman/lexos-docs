# Version Selector Implementation

## The `dev` Docs

The deployment workflow automatically deploys to the `dev` folder in the lexos `gh-pages` branch. This has a version selector implemented by `mike` (configured) in `mkdocs.yml`.

## Earlier Releases

Earlier releases are not re-built upon deployment; they are now static websites served from the lexos `gh-pages` branch. Currently, there is only one release, `v0.1.0-beta.31`, which is mirrored in `stable-beta` and `latest`. Every `index.html` file in these folders has a JavaScript include, `assets/javascripts/version_selector.js`, which injects the version selector in the page with "v0.1.0-beta.31" pre-selected. As long as there are no re-builds to the site, the version selector should remain the same. Note that the selector's only other option is "dev". If further versions are to be displayed by `mike` at a later date, the `version_selector.js` script will beed to be updated in the `v0.1.0-beta.31`, `stable-beta`, and `latest` folders.
