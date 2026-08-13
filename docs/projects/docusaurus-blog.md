---
sidebar_position: 3
title: Docusaurus Blog
description: How I configured my DevSecOps learning journal.
---

import GithubLinkAdmonition from '@site/src/components/GithubLinkAdmonition';

# Docusaurus Blog

This project is my DevSecOps learning journal and portfolio. It is based on the Developer Akademie Docusaurus starter.

<GithubLinkAdmonition
  link="https://github.com/FriggemannMichael/my-Blog"
  title="Project repository"
  type="tip"
>
  This repository contains the implementation.
</GithubLinkAdmonition>

## Table Of Contents

- [Quickstart](#quickstart)
- [Usage](#usage)
  - [Configuration](#configuration)
  - [Deployment](#deployment)
  - [Validation](#validation)

## Quickstart

### Prerequisites

Git, Node.js 22 or later, and pnpm 11.9.0 must be installed.

Clone the repository:

```bash
git clone https://github.com/FriggemannMichael/my-Blog.git
```

Change into the project directory:

```bash
cd my-Blog
```

Install the dependencies:

```bash
pnpm install --frozen-lockfile
```

Start the development server:

```bash
pnpm start
```

Open `http://<your_ip>:3000/my-Blog/` after the server starts.

## Usage

### Configuration

I configured the starter by:

1. standardizing dependency management on pnpm;
2. replacing the template identity and repository links;
3. using `GIT_REPOSITORY_URL` for docs and blog edit links;
4. updating the navbar and footer;
5. adding the `/docs/projects` route;
6. improving the homepage button spacing and mobile wrapping.

#### Environment variables

Copy `example.env` to `.env` only when overriding the safe defaults. Never commit `.env`.

| Variable | Purpose |
| --- | --- |
| `DEPLOYMENT_URL` / `BASE_URL` | Public site origin and repository path |
| `GITHUB_ORG` / `GITHUB_PROJECT` | GitHub Pages account and repository |
| `DEPLOYMENT_BRANCH` | Production deployment branch |
| `GIT_REPOSITORY_URL` | Repository and edit-link source |
| `BLOG_ENABLED` | Enables the optional blog when set to `true` |

Only public values belong in `example.env`; secrets and private infrastructure data stay outside Git.

### Deployment

GitHub Actions builds the static site and uploads it as a GitHub Pages artifact. Pull requests validate the build, while deployment is restricted to commits on `main`.

After Pages uses **GitHub Actions** as its source, the site is published at `https://friggemannmichael.github.io/my-Blog/`. CI must use the pinned pnpm version and lockfile.

### Validation

```bash
pnpm install --frozen-lockfile
pnpm run typecheck
pnpm run build
```

Before submission, I also verify the routes and links, green CI checks, and the absence of credentials, SSH keys, `.env`, or private infrastructure data.

## Further references

- [Docusaurus documentation](https://docusaurus.io/docs)
- [Docusaurus deployment guide](https://docusaurus.io/docs/deployment)
- [pnpm documentation](https://pnpm.io/)
- [GitHub Pages documentation](https://docs.github.com/pages)
