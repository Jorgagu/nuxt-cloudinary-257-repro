# nuxt-cloudinary-257-repro

Reproduction of [nuxt-modules/cloudinary#257](https://github.com/nuxt-modules/cloudinary/issues/257): `[@vue/compiler-sfc] Failed to resolve extends base type` with Nuxt 4.

The original StackBlitz linked in the issue is no longer available, so this project rebuilds it from the
[Nuxt v4 starter as of the issue date](https://github.com/nuxt/starter/tree/3b45411ad0fa2178f3028624cbec844b2f229994)
with `@nuxtjs/cloudinary` added to `modules`. Dependencies are pinned to the versions resolved on 2025-08-11
(`package-lock.json` plus `overrides` for the Vue packages).

| Branch             | `@nuxtjs/cloudinary` | Result of `npm run dev`, `GET /`                     |
|--------------------|----------------------|------------------------------------------------------|
| `main`             | 4.0.0                | 500, `Failed to resolve extends base type`           |
| `cloudinary-4.1.0` | 4.1.0                | 200, Nuxt welcome page                               |

Everything else is identical between the two branches: Nuxt 4.0.3, Vue 3.5.18, TypeScript 5.9.2.

## Run

- StackBlitz, `main`: https://stackblitz.com/github/Jorgagu/nuxt-cloudinary-257-repro
- StackBlitz, `cloudinary-4.1.0`: https://stackblitz.com/github/Jorgagu/nuxt-cloudinary-257-repro/tree/cloudinary-4.1.0
- Locally: `npm ci && npm run dev`, then open http://localhost:3000

## Root cause

- `@nuxtjs/cloudinary` 4.0.0 ships `CldImage.vue` as TypeScript source with
  `interface CldImageProps extends ImageOptions`, where `ImageOptions` comes from `@cloudinary-util/url-loader`.
  The consumer's `@vue/compiler-sfc` has to resolve that type when TypeScript is installed.
- `@cloudinary-util/url-loader` 6.0.0 exposes its types as `./dist/index.d.cts`, which `@vue/compiler-sfc`
  could not resolve before [vuejs/core#14402](https://github.com/vuejs/core/pull/14402) (released in Vue 3.5.28).
- `@nuxtjs/cloudinary` 4.1.0 ships precompiled components with runtime props, so no type has to be resolved.

With Vue 3.5.28 or later, 4.0.0 no longer fails either. See
[nuxt-cloudinary-257-modern](https://github.com/Jorgagu/nuxt-cloudinary-257-modern) for the current stack.
