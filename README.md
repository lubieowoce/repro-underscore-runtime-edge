This is a repro app that has a path that starts with `%5F` and uses `runtime = "edge"`. `%5F` is a builtin workaround for [Private Folders](https://nextjs.org/docs/app/building-your-application/routing/colocation#private-folders), which make `_directories` non-routable.

### `next dev` - Broken

1. `npm run dev`, go to [http://localhost:3000/\_underscored](http://localhost:3000/_underscored)
2. Error: `Error: ENOENT: no such file or directory, open '/Users/owoce/code/repros/edge-manifest-minimal-NEXT-3687/.next/server/app/%5Funderscored/page_client-reference-manifest.js'`

Full log:

```
$ rm -rf .next; pnpm next dev
  ▲ Next.js 14.2.5
  - Local:        http://localhost:3000

 ✓ Starting...
 ✓ Ready in 814ms
 ○ Compiling /%5Funderscored ...
 ✓ Compiled /%5Funderscored in 1721ms (671 modules)
 ⨯ Error: ENOENT: no such file or directory, open '<REPO>/.next/server/app/%5Funderscored/page_client-reference-manifest.js'
    at readFileSync (node:fs:453:20)
    at evaluateInContext (file:///<REPO>/node_modules/.pnpm/next@14.2.5_react-dom@18.3.1_react@18.3.1__react@18.3.1/node_modules/next/dist/server/web/sandbox/context.js:401:50)
    <snip>
```

Notice how `.next` has two folders for this page, when it should only have one:

```console
$ tree .next/server/app
.next/server/app
├── %5Funderscored
│   └── page.js
└── _underscored
    └── page_client-reference-manifest.js
```

If we remove `export const runtime = "edge"`, the crash seems to go away. Oddly enough, Next still creates two directories, but somehow manages to work anyway.

### `next build` - OK

It works fine in `next build`, in which case we'll only have `.next/server/app/_underscored`.

### `next dev --turbo` - Broken (in a different way)

Turbopack breaks on this too, but in a diferent way.

```
  ▲ Next.js 14.2.5 (turbo)
  - Local:        http://localhost:3000

 ✓ Starting...
 ✓ Ready in 625ms
 ○ Compiling /_underscored ...
 ✓ Compiled /_underscored in 1220ms
 ⨯ [TypeError: Cannot read properties of undefined (reading 'clientModules')]
 ⨯ [TypeError: Cannot read properties of null (reading 'default')]
 GET /_underscored 500 in 1374ms
```

Removing `export const runtime = "edge"`, changes nothing here, it crashes with the same error either way.
