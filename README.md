# Website

This website is built using [hugo](https://gohugo.io), Hugo is one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again.

## Installation

```bash
mise trust
mise install

bun install
```

## Local Development

```bash
hugo server --buildDrafts --disableFastRender
```

This command starts a local development server and opens up a browser window. Most changes are reflected live without having to restart the server.

## Build

```bash
hugo
```

This command generates static content into the `build` directory and can be served using any static contents hosting service.
