# Noah Moran's Website

Personal website hosted at https://chucknoah.github.io, built with Jekyll and Academic Pages.

## Editing Content

- Homepage: `_pages/about.md`
- Publications listing: `_pages/publications.html`
- Individual papers: `_publications/`
- Mentorship and work in progress: `_pages/wip.html`
- Role Restoration: `_pages/role-restoration.md`
- Split Personas and Misalignment: `_pages/split-personas-and-misalignment.md`
- Navigation: `_data/navigation.yml`
- Profile and site settings: `_config.yml`
- Profile photo: `images/face_pic.jpg`

The site also keeps a 404 page. Unused demo pages, collections, and sample files have been removed.

## Figures

Use `<figure class="research-figure">` with an image and a `<figcaption>` to center research figures. Start each caption with `<strong>Figure 1:</strong>` (numbered in page order), followed by a one- or two-sentence description in normal text.

## Local Preview

Install Ruby and Bundler, then run:

```sh
bundle install
bundle exec jekyll serve
```

Open http://localhost:4000. Restart Jekyll after editing `_config.yml`.

Shared theme files in `_includes/`, `_layouts/`, `_sass/`, and `assets/` provide the site's styling and rendering. Keep these when editing content. The theme's MIT license is in `LICENSE`.
