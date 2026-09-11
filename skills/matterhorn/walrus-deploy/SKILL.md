---
name: walrus-deploy
description: How publishing to Walrus works inside Matterhorn — what makes a project publishable, how to declare the build and output directory in .matterhorn/publish.json, framework-by-framework export settings, why the site is content-addressed, who owns the resulting storage, and how to diagnose a rejected publish. Use when the user wants to deploy, publish, or host their project, or when a publish has failed.
---

# Publishing to Walrus from Matterhorn

Matterhorn hosts user projects on **Walrus**, Mysten Labs' decentralised
storage network, with the site registered as an object on **Sui**. This skill
is about the Matterhorn side of that: what you must do so the user's Publish
button works. For the Walrus protocol itself — blobs, quilts, epochs, the CLI
— see the `walrus-overview`, `walrus-cli`, and `walrus-http-api` skills.

## Your role: prepare, never publish

Publishing is a **server-side action**, triggered when the user clicks Publish
in the Walrus tab. It spends real WAL and SUI from Matterhorn's funded wallet.

You do not run it. You have no wallet, no keys, and no way to spend tokens —
that is deliberate, not a missing feature. Never try to `walrus store` the
user's site, never ask them for a private key, and never tell them they need
to buy anything. Publishing is free to them.

What you *do* own is the state of the workspace at the moment they click. Two
responsibilities:

1. Leave a **built, static** site on disk.
2. **Declare where it is**, in `.matterhorn/publish.json`.

## Declaring the publish config

Write `.matterhorn/publish.json` at the workspace root as soon as the project
has something publishable, and keep it current — if you change the build
script or move the output, update the file in the same turn.

```json
{
  "buildCommand": "npm run build",
  "outputDir": "dist",
  "spaFallback": true
}
```

| field | meaning |
|---|---|
| `buildCommand` | Run before collecting files. **Omit entirely** for a static site with no build step — an empty string or a no-op command is worse than absence. |
| `outputDir` | Directory holding `index.html`, relative to the workspace root. `"."` if the site is at the root. Must stay inside the workspace: absolute paths and `..` are rejected. |
| `spaFallback` | Route unmatched paths to `/index.html`. `true` for client-side routers only. |

All three are optional, and a malformed file is ignored rather than failing
the publish — but then the server falls back to guessing, and the guess is
what you were supposed to replace.

### When to set `spaFallback`

`true` for React Router, Vue Router, or any app where `/about` is rendered by
JavaScript and there is no `about.html` on disk. Without it that URL 404s.

`false` for a multi-page site with real files (`index.html`, `about.html`).
With it wrongly `true`, every typo silently renders the homepage instead of a
404, which hides broken links from the user.

## What Walrus can and cannot serve

Walrus serves **bytes**. There is no server process, no Node runtime, no API
routes, no SSR, no runtime environment variables, no database.

If the project needs a backend, publish the frontend and have it call an
external API over HTTPS. Say so plainly rather than publishing something that
will be broken in a way the user only discovers later.

### By framework

| stack | what to do | `outputDir` |
|---|---|---|
| Plain HTML/CSS/JS | nothing to build | `.` |
| Vite | `npm run build` | `dist` |
| Create React App | `npm run build` | `build` |
| Next.js | set `output: "export"` in `next.config.js`; no `getServerSideProps`, no API routes, `images.unoptimized: true` | `out` |
| Astro | `npm run build` (default static output) | `dist` |
| SvelteKit | `@sveltejs/adapter-static` | `build` |
| Eleventy | `npx eleventy` | `_site` |
| Hugo / Jekyll | `hugo` / `jekyll build` | `public` / `_site` |

**Never publish an unbuilt Vite or CRA tree.** Its `index.html` loads
`/src/main.tsx`, which no browser can execute. Walrus would store it happily
and the visitor would get a blank page, so the server rejects it outright with
`not_built`. Build first.

## Limits

- **10 MiB** per file, **400 files** per site.
- `.env`, `.env.local`, `.env.production`, `.env.development` and `.DS_Store`
  are never uploaded. Do not rely on that as your only protection: a secret
  compiled into a JS bundle *does* get published, and a published blob is
  public and permanent. Client-side code must only ever hold public keys.
- `node_modules`, `.git`, `target`, `.venv`, `__pycache__` and caches are
  skipped. Keep build output out of the source tree so it is not collected
  twice.

## Paths inside the published site

The site is served from its own subdomain, so **root-absolute paths work**:
`/styles.css`, `/assets/logo.svg`. Relative paths work too.

What does not work is anything resolved at runtime from a server: no
`process.env`, no `fetch("/api/...")` to your own origin, no rewrites.

## After publishing

Publishing writes the files to Walrus and returns a **site id** — the blob id
of the manifest. The site is *content-addressed*: identical files produce an
identical site id.

Two consequences worth telling the user about:

- **Republishing unchanged content costs nothing and changes nothing.** The
  server detects it before uploading and reuses the existing deploy.
- **Any real change produces a new site id**, and the previous version stays
  on Walrus until its storage lapses. Old deploys remain reachable.

Registering the site on **Sui** is a separate click that costs gas. It is what
gives the site a public URL. The Sui object and every storage object are
transferred to *the user's own Sui address*, not Matterhorn's — they own what
they publish even though Matterhorn paid for it. If asked who controls the
site: they do, and they can export the key from the Walrus tab.

## When a publish is rejected

| error | meaning | fix |
|---|---|---|
| `no_output` | No `index.html` found anywhere the server looked | Build the project, then write `.matterhorn/publish.json` naming the real output directory |
| `not_built` | `index.html` loads a `.ts`/`.tsx`/`.jsx` module | Run the build; publish the output directory, not the source root |
| `build_failed` | The declared `buildCommand` exited non-zero | Read the returned log, fix the build, try again |
| `sandbox_not_created` | No workspace yet | Nothing to do in the sandbox |

If the user reports a blank page after a successful publish, suspect an
unbuilt tree, an asset referenced with a path that only existed under the dev
server, or a `fetch` to a backend that is not there.

## A worked example

The user has a Vite React app with client-side routing.

```bash
npm run build          # produces dist/
ls dist/index.html     # confirm it exists
```

```json
{
  "buildCommand": "npm run build",
  "outputDir": "dist",
  "spaFallback": true
}
```

Then tell them: "Built to `dist/` and recorded it for publishing — click
Publish in the Walrus tab whenever you're ready."
