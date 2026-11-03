# Product Studio

A local first, real time 3D visualization workspace built with React, TypeScript, Vite, Three.js, and WebAssembly Open CASCADE STEP import.

## Run locally

Use Node.js 20 or later.

```sh
npm install
npm run dev
```

Open the local URL printed by Vite. To create a production build, run `npm run build`. Run interaction and import checks with `npm test`.

## Publish a demo with GitHub Pages

The repository includes a GitHub Actions workflow at `.github/workflows/deploy-pages.yml`. This project is configured for `https://kars-impaqt.github.io/product-studio/`. Enable **Settings → Pages → Build and deployment → GitHub Actions** in the repository. Each push to `main` builds the app and publishes `dist`; the Vite base path is configured for GitHub project pages. The deployed app processes model and reference image uploads in the visitor's browser; those files are not sent to GitHub or another service.

## Supported input

- **STL** (`.stl`): parsed in the browser using Three.js STLLoader. STL has no assembly hierarchy or reliable material/color data, so it appears as one part.
- **STEP** (`.step`, `.stp`): parsed on-device using [`occt-import-js` 0.0.23](https://github.com/kovacsv/occt-import-js) (Open CASCADE WebAssembly, LGPL-2.1). The importer returns a node hierarchy plus triangulated mesh pieces; the node path is included in each selectable part name, and available per-mesh colors are retained until the user changes that part's color. Install/deploy this app with its npm dependencies so the WASM is emitted as a local asset.
- **Product reference image** (`.png`, `.jpg`, `.jpeg`, `.webp`): choose an image from the Product Image panel or drop it into the studio. The app locally samples the dominant color from the image center and applies it across the model, then previews the image beside the scene and in the viewport. Manually changing a part color returns to normal per-part coloring. Toggle or remove the reference at any time. Images are held locally in browser memory, limited to 20 MB, and are not sent to a service or included in PNG exports.
- **SolidWorks native** (`.SLDPRT`, `.SLDASM`) and ZIP assemblies: not parsed by this app. The upload interface explains that a compatible conversion backend is required and asks the user to export STEP from SolidWorks. ZIP is intentionally rejected until such a backend is configured.

## SolidWorks conversion integration

`src/modelImport.ts` is the conversion boundary. A production integration can add an opt-in backend adapter there, behind a deployment setting such as `VITE_CAD_CONVERTER_URL`. The adapter should authenticate with a licensed SolidWorks-compatible SDK/service, accept multipart CAD uploads, preserve referenced assembly parts, return a documented neutral mesh/STEP representation, and map its result into the existing `ImportedPart[]` shape. Obtain the required commercial license and tell users clearly before any model leaves their device. No converter URL, credentials, upload, or native SolidWorks support is configured in this version.

## Current scope and known limits

- Orbit, pan, zoom, fit-to-model, front/side/top/isometric views, part selection, and viewport face picking are provided. STEP triangle hits map back to the source B-rep face ranges when the importer exposes them; STL has no CAD face topology, so each triangle is selectable. Color edits apply to the selected CAD face or part, alongside material presets, lighting presets, background, ground shadows, and PNG export at 2×/3×/4×.
- Transparency in the PNG is available by enabling the transparent background option. Three.js renders a real time preview; it is not KeyShot quality photorealistic ray tracing.
- Very large CAD models can take time or exceed browser memory. The STEP library is synchronous while it triangulates, so progress reporting is limited to import stages, not per-face progress. STEP tessellation settings and units are governed by the importer; there is no CAD editing or geometry repair.
- STEP assembly tree fidelity depends on the shapes/names exposed by the selected STEP file and importer. The app keeps returned mesh pieces as separate selectable parts.
- Lighting presets adjust the viewport's key, fill, and ambient intensities. HDRI environments and custom light placement controls are future extensions.

## License notes

The application is an example implementation. [`occt-import-js` is published under LGPL-2.1](https://github.com/kovacsv/occt-import-js/blob/main/package.json); review its license and distribution requirements for your deployment. Three.js is MIT licensed. This project does not bundle a SolidWorks SDK or grant rights to process SolidWorks native files.

