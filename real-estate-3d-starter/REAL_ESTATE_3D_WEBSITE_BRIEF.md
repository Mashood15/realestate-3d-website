# Real Estate Cinematic 3D Website — Implementation Brief

## Goal
Build a premium real-estate marketing website in **Next.js static export mode**. The hero and main storytelling section use a real-time 3D scene: as visitors scroll, the camera moves around a modern villa, approaches the facade, reveals the interior, and pulls back to an aerial neighborhood view.

This project must work without a backend at runtime. It should be deployable as static files to any static host that supports HTML, CSS, JavaScript, and assets.

## Required stack
- Next.js App Router with `output: "export"`
- TypeScript
- React Three Fiber (`@react-three/fiber`)
- Drei (`@react-three/drei`)
- Three.js
- GSAP + ScrollTrigger
- CSS Modules or global CSS; responsive, accessible HTML overlays

Install:
```bash
npm install three @react-three/fiber @react-three/drei gsap
npm install -D @types/three
```

If starting from scratch:
```bash
npx create-next-app@latest real-estate-3d --typescript --eslint --app
```

In `next.config.ts`, configure:
```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "export",
  images: { unoptimized: true },
};

export default nextConfig;
```

Use a client component for all WebGL, scroll animation, and browser APIs. Avoid server-side imports of browser-only code. Add `dynamic(() => import(...), { ssr: false })` if needed, from a client component.

## Design direction
- Visual tone: premium, architectural, editorial, cinematic; avoid generic template appearance.
- Palette: warm limestone, charcoal, muted olive, soft bronze; use restrained accents.
- Typography: oversized high-contrast serif headlines paired with a clean sans-serif UI font.
- Motion: slow, intentional, eased; no excessive bouncing or constant spinning.
- Layout: full-viewport canvas behind/alongside clear text; sticky/pinned story section; clear CTA.
- Content is fictional placeholder copy; make all project data easy to replace.

## Storyboard / scroll sequence
Use a pinned section about 500–650vh tall on desktop, with the canvas fixed/sticky inside it. Map normalized scroll progress `0..1` to the following keyframes. Smoothly interpolate camera and scene state. Do not trigger one-shot animations that break when scrolling backwards.

| Progress | Camera position (x,y,z) | Look-at (x,y,z) | Scene action | Overlay |
|---|---|---|---|---|
| 0.00 | (8, 4.5, 12) | (0, 1.5, 0) | Full villa, hero lighting | “A quieter kind of extraordinary.” |
| 0.18 | (4, 3.2, 8) | (0, 1.5, 0) | Approach front facade | “Architecture in balance.” |
| 0.36 | (0.8, 2.4, 4.8) | (0, 1.4, 0) | Facade close-up; accent lights brighten | “Made for the way you live.” |
| 0.54 | (-3, 2.2, 3.8) | (0, 1.3, 0) | Lateral move; show interior through glass | “Space to breathe.” |
| 0.72 | (-5, 5.5, 7) | (0, 1, 0) | Rise and pull back to garden | “A private world, connected.” |
| 1.00 | (0, 15, 18) | (0, 0, 0) | Elevated overview of villa and site | “Find your place here.” |

These coordinates are illustrative for the procedural placeholder scene. If a downloaded villa GLB is used, tune camera keyframes to its scale and orientation.

## Asset strategy (important)
AI-generated 3D models are not a prerequisite for the first build. Create a complete visual prototype using procedural Three.js geometry first, then replace the placeholder villa with a quality `.glb` model later.

Asset priority:
1. A high-quality architectural villa GLB with separately named meshes where possible.
2. Environment: ground/terrain, driveway, trees, shrubs, exterior lights.
3. Optional interior props: sofa, table, chairs, pendant lights.
4. Environment HDRI and PBR textures.
5. Fallback: procedural model included in `src/components/scene/ProceduralVilla.tsx`.

**Do not make the site depend on an asset that has not been provided.** The page must still render if no GLB file exists.

Recommended public asset sources (check each asset’s license before commercial use):
- Poly Haven: https://polyhaven.com/
- Sketchfab downloadable models: https://sketchfab.com/
- CGTrader: https://www.cgtrader.com/
- BlenderKit: https://www.blenderkit.com/
- Meshy AI: https://www.meshy.ai/
- Tripo AI: https://www.tripo3d.ai/

For web delivery, prefer optimized `.glb` files. Keep the initial 3D download small; compress models/textures, use KTX2 textures or Draco mesh compression only if the loader is configured for them, and lazy-load secondary scenes.

## Required scene implementation
Create:
- `src/components/scene/RealEstateCanvas.tsx`: Canvas, lights, environment, model and camera rig.
- `src/components/scene/ProceduralVilla.tsx`: reliable no-external-asset villa placeholder built from boxes, planes, cylinders, and roof geometry.
- `src/components/scene/ScrollCameraRig.tsx`: receives normalized progress and updates camera position/look-at with interpolation.
- `src/components/scene/ModelLoader.tsx`: optional GLB loader; if `/models/villa.glb` is absent or fails, use `ProceduralVilla`.
- `src/components/ScrollStory.tsx`: pinned story layout and GSAP ScrollTrigger progress.
- `src/components/sections/ProjectIntro.tsx`: hero and CTA.
- `src/components/sections/ProjectDetails.tsx`: static feature cards, specifications, gallery and inquiry CTA.
- `src/app/page.tsx`: compose the page.
- `src/app/globals.css`: design system and responsive layout.

Use `prefers-reduced-motion` to provide a reduced-motion experience. On small screens, lower DPR, simplify shadows, and reduce or disable expensive effects.

## Animation requirements
- Use ScrollTrigger with `scrub`, not independent autoplay timelines.
- Animation must work when scrolling up as well as down.
- Pin only the storytelling region, not the whole page.
- Use responsive breakpoints and refresh ScrollTrigger after layout changes.
- Animate camera position and look-at target; don't rely only on rotating the house.
- Add subtle independent motion only when it does not fight scroll progress.
- Cross-fade overlay text between story beats.
- Avoid scroll-jacking: native page scrolling should remain natural.
- Keep CTA buttons clickable above the canvas (`pointer-events` layering handled deliberately).

## Static export requirements
- No API routes, server actions, runtime database, or request-time server rendering.
- Use local/static project data.
- For images, use `<img>` or Next Image with `unoptimized: true`.
- Verify `npm run build` creates the `out/` directory.
- All model and texture paths must be rooted under `/public`.
- Test the exported `out/` directory using a static server, not only `next dev`.
- Use relative-safe asset paths if deploying under a subpath; otherwise use root-relative paths.
- If using a hosting platform, configure SPA/fallback behavior only if needed; this is a single static Next export.

## Quality and acceptance criteria
1. `npm run build` succeeds and exports static files.
2. No runtime dependency on an external model; procedural fallback always works.
3. On desktop, scroll maps smoothly to camera keyframes and overlay copy.
4. Reverse scrolling restores previous states correctly.
5. Mobile layout is readable and scene is performant.
6. Keyboard-accessible CTA and navigation; sensible contrast.
7. Reduced-motion mode avoids long camera journeys.
8. Console has no missing-asset errors or React/Three warnings.
9. All third-party assets have a recorded license/source.
10. README explains how to replace the procedural villa with `public/models/villa.glb`.

## Implementation order
1. Scaffold Next.js static export.
2. Build procedural villa and garden (no downloaded assets).
3. Render it in React Three Fiber with camera and lights.
4. Add pinned scroll story and camera keyframes.
5. Add typography, section overlays, CTA and project details.
6. Test mobile/reduced motion/static export.
7. Optionally replace the procedural villa with a licensed GLB.

## Important instruction to the coding agent
Build a polished working implementation, not just a plan or mockup. Do not stop at placeholders for the whole scene: the procedural villa should be a visible, recognizable low-poly modern house with walls, roof, windows, door, driveway, garden and trees. Keep the code modular and type-safe. If a feature is too expensive, implement a simpler working version and document the limitation.
