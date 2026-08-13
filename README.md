# My Developer Blog

This website is built with [Docusaurus](https://docusaurus.io/), a modern static site generator.

## Repository Description

This repository contains a Docusaurus-based developer blog and knowledge base for DevSecOps topics. It supports local development with pnpm, an optional Docker and NGINX image, and automated deployment to GitHub Pages.

## Table of Contents

- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)
- [Deployment](#deployment)
  - [GitHub Pages](#github-pages)
  - [NGINX and Docker](#nginx-and-docker)

## Quickstart

### Prerequisites

- [Node.js](https://nodejs.org/) 22 or later
- [pnpm](https://pnpm.io/) 11.9.0, as specified in `package.json`
- [Docker](https://www.docker.com/products/docker-desktop) only when using the [NGINX and Docker](#nginx-and-docker) deployment option

1. Install the dependencies:

   ```shell
   pnpm install
   ```

2. Start the local development server:

   ```shell
   pnpm start
   ```

   Docusaurus opens the site in a browser and reflects most changes without a server restart.

3. Create a production build:

   ```shell
   pnpm build
   ```

   The generated static site is written to the ignored `build/` directory.

4. Deploy the site:

   Merge the change into `main`. The prepared GitHub Actions workflow builds and deploys the site to GitHub Pages automatically; no local deployment command is required. See [Deployment](#deployment) for details.

## Repository Structure

The committed project files are organized as follows:

- `.github/workflows/`
  - `main.yml`: Runs the CI/CD workflow for pull requests and pushes targeting `main`.
  - `deploy.yaml`: Reusable workflow that builds the Docusaurus site, uploads the Pages artifact, and deploys it when the workflow runs on the default branch.
  - `create-pr.yaml`: Attempts to create a pull request when a non-default branch is pushed.
  - `check-open-pr.yaml`: Verifies that a pushed feature branch has an open pull request.
- `blog/`: Contains blog posts plus author and tag metadata. The blog is enabled through the `BLOG_ENABLED` environment variable.
- `docs/`: Contains the guides, knowledge-base articles, project pages, category metadata, and documentation assets.
- `src/`: Contains custom React components, pages, styles, and theme extensions.
- `static/`: Contains files copied directly into the generated site, including images and `.nojekyll` for GitHub Pages.
- `.dockerignore` and `.gitignore`: Exclude generated output, dependencies, local environment files, and editor metadata from Docker builds or version control.
- `Dockerfile`: Builds the static site and serves it from an NGINX container.
- `docusaurus.config.ts`: Defines site metadata, environment-based deployment settings, navigation, plugins, and theme configuration.
- `sidebars.ts`: Generates the documentation sidebar from the `docs/` directory.
- `babel.config.js` and `tsconfig.json`: Configure Babel and TypeScript for Docusaurus development.
- `example.env`: Documents the environment variables used for the site URL, base path, repository, and optional blog.
- `package.json`: Defines project metadata, the required Node.js and pnpm versions, dependencies, and development scripts.
- `pnpm-lock.yaml`: Locks exact dependency versions for reproducible pnpm installs.
- `pnpm-workspace.yaml`: Defines the pnpm workspace and dependency build policy.
- `README.md` and `LICENSE`: Provide project documentation and licensing information.

Add documentation under `docs/` and blog posts under `blog/`. Docusaurus discovers the content and category metadata from those directories.

## Deployment

### GitHub Pages

The workflows in `.github/workflows/` validate pull requests targeting `main`. After a commit reaches `main`, the CI/CD workflow builds the site and deploys the generated artifact to GitHub Pages automatically.

### NGINX and Docker

To build and serve the site in an NGINX container, follow the [Docker and NGINX deployment guide](./docs/guides/deploy-docusaurus-with-docker-and-nginx.md).
