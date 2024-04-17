## Customization

### Logo

Override `/themes/novela/layouts/partials/icons/ui/logo.html` with your own file at `/layouts/partials/icons/ui/logo.html`; include your logo in SVG format for desktop and mobile formats.

Novela supports light and dark mode. To have your logo respond in kind, add `class="change-fill"` to the svg path(s).

### Socials

In order for the Socials to be surfaced in Forestry, you should copy the theme's `config/_default/social.yaml` to your project.

### Authors

You should register authors as a taxonomy in your project's `config.yaml``

```yaml
taxonomies:
  author: authors
```

#### Creating authors

Authors must be added in `content/authors`.
Create a folder per author and add an `_index.md` file in it.

Here's an example of the front matter fields supported by default:

```yaml
# /content/authors/firstname-lastname/_index.md
---
title: Dennis Brotzky
bio: |
  Written by You. This is where your author bio lives. Share your work, your
  joys and of course, your Twitter handle.
avatar: /images/dennis-brotzky.jpg
featured: true
social:
  - title: unsplash
    url: https://unsplash.com
  - title: github
    url: https://github.com
  - title: github
    url: https://github.com
  - title: github
    url: https://github.com
  - title: github
    url: https://github.com
---
```

#### Assigning authors to posts.
Ad the name of the author to the "authors" field:

```yaml
authors:
  - Dennis Brotzky
  - Thiago Costa
```