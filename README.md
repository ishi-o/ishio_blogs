# ishio_blogs

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
