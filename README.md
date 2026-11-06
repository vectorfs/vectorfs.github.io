# vectorfs.github.io

The VectorFS website — landing page, documentation, and release notes — built with
[OINK](https://oink.pgsty.com/) v1.5.0 on Hugo Extended 0.168.0 and deployed to
GitHub Pages. Live at <https://vectorfs.io/>.

The product itself lives in [vectorfs/vectorfs](https://github.com/vectorfs/vectorfs).

## Run locally

```bash
git clone https://github.com/vectorfs/vectorfs.github.io.git
cd vectorfs.github.io
hugo server
```

Open <http://localhost:1313/>. The first run downloads the pinned OINK Hugo module
(`v1.5.0`). You need Git, Go 1.27.3 or newer, and Hugo Extended 0.168.0 or newer —
no external Sass, Node.js, or npm.

Production build, the same way CI runs it:

```bash
hugo --cleanDestinationDir --gc --minify --environment production \
  --printPathWarnings
```

`public/`, `resources/`, and caches are Git-ignored. The upstream LibSass
deprecation warning is expected; keep warnings visible and do not enable
`--panicOnWarning` or `HUGO_PANICONWARNING`.

## Content map

```text
content/
├── _index.*            home (renders data/home sections)
├── docs/
│   ├── introduction/   overview, architecture
│   ├── get-started/    requirements, installation, quickstart
│   ├── tutorial/       file pipeline, emulator, MCP, deployment
│   ├── reference/      commands, configuration, API, limits
│   └── develop/        build from source, design documents
└── blog/
    └── release/        release announcements
```

English files end in `.md`; Chinese peers end in `.zh.md` and sit side by side.
Keep translated files together and keep explicit heading IDs aligned between
languages. The landing page copy lives in `data/home/en.yaml` and
`data/home/zh.yaml`.

## Languages

English is the default language; Chinese is a full peer. French stays declared
but disabled so `.fr.md` suffixes are recognized and ignored. To change the
profile:

```bash
cp examples/hugo.single.yaml hugo.yaml     # English only
cp examples/hugo.bilingual.yaml hugo.yaml  # English + Chinese (current)
```

Restore the three-language configuration from Git before switching profiles
again.

## Deployment (GitHub Pages)

`.github/workflows/github-pages.yaml` builds on every push to `main` (and on
manual dispatch), then publishes through the Pages deployment API. One-time
setup:

1. Open **Settings → Pages** and choose **GitHub Actions** as the source.
2. Push to `main`, or run **Deploy to GitHub Pages** from the Actions tab.

The workflow installs Go 1.27.3 and Hugo Extended 0.168.0, downloads the pinned
OINK module, builds with `--minify`, derives the correct base URL from the Pages
configuration (project subpath or custom domain), and uploads the artifact. The
custom domain `vectorfs.io` is configured in **Settings → Pages** (source stays
**GitHub Actions**); the root `CNAME` file is inert under Actions-mode
deployments — the domain comes from the Pages settings.

## License

MIT. The VectorFS mark and product are Apache-2.0, see the main repository.
