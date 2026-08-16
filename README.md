# ishio_blogs

Hexo blog source, deployed to GitHub Pages by [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml). The workflow only triggers when site content or config changes (`source/`, `scaffolds/`, `themes/`, `_config*.yml`, `package*.json`).

## Run locally

Only two things on your machine:

- `Node.js` >= 20 — provides `npx`, no global `hexo-cli` needed
- `pandoc` — required at runtime by `hexo-renderer-pandoc` (e.g. `sudo apt install pandoc`)

```bash
git clone --recurse-submodules https://github.com/ishi-o/ishio_blogs.git
cd ishio_blogs

# if you cloned without submodules
git submodule update --init --recursive

npx npm ci --legacy-peer-deps
npx hexo clean
npx hexo server
```

Other useful commands (all through `npx`):

```bash
npx hexo new post '<title>' # scaffold a new post in source/_posts/
npx hexo generate           # build static site into public/
```
