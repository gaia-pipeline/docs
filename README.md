# Gaia Documentation

This repository contains documentation about [Gaia](https://github.com/gaia-pipeline/gaia).

It can be viewed locally (for editing and format testing) or online at this address:
[Gaia Documentation](https://docs.gaia-pipeline.io/).

## To build it locally

Follow these steps:

### Get the repo

```
git clone git@github.com:gaia-pipeline/docs.git
git submodule init
git submodule update
```

### Install Hugo

Hugo is a static site generator tool located here: [Hugo](https://gohugo.io/). Please follow
the installation steps provided for your operating system here: [Installing Hugo](https://gohugo.io/getting-started/installing/).

### Testing

Add or edit a page. The pages are written in Markdown format, then Hugo is used to generate HTML
out of that format. For samples, please check out the contents in the `content` folder.

Once done, run:

```
hugo server
```

Visit `http://localhost:1313/` and find your page in the right section.
