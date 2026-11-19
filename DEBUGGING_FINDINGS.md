# Nocturne Glasshouse — Build & Debugging Findings

**Project repository:** https://github.com/Pinakbakshi/nocturne-glasshouse  
**Deployment target:** GitHub Pages  
**Intended viewer:** https://pinakbakshi.github.io/nocturne-glasshouse/  
**Recorded:** 2026-11-19

## 1. ChatGPT model and implementation stack

- **ChatGPT model used for the implementation and debugging session:** GPT-6.
- **3D engine:** Three.js r160 (JavaScript library; not an AI model).
- **Controls:** Three.js OrbitControls for pointer and touch navigation.
- **Rendering:** WebGL in the browser.
- **Hosting and CI/CD:** GitHub Pages with GitHub Actions.

The glasshouse is procedurally assembled in JavaScript as an interactive 3D scene. It is not a static AI-generated image.

## 2. Work completed

1. Created a separate public repository: `Pinakbakshi/nocturne-glasshouse`.
2. Added the 3D viewer as the root-level `index.html`.
3. Added a GitHub Actions workflow at `.github/workflows/pages.yml`.
4. Configured the viewer to load Three.js and its add-ons through an import map. This is important because OrbitControls imports the bare module specifier `three`; resolving the add-on by URL alone is not sufficient unless the browser also knows how to resolve that specifier.
5. Set up the scene with the greenhouse structure, glass enclosure, framing, lighting, vegetation and interactive camera controls.
6. Removed the earlier glasshouse viewer and its deployment workflow from the separate `chatgpt-project-archive` repository, so this project is maintained independently.
7. Investigated the GitHub Actions failures and revised the Pages workflow.
8. Confirmed that subsequent GitHub Actions deployment runs completed successfully.

## 3. Debugging findings

### Issue A — GitHub Pages site provisioning failed

**Observed:** The initial workflow failed at `actions/configure-pages@v5` with a permissions/resource-access error. The artifact upload and deployment steps were skipped.

**Cause indicated by the run:** The action was asked to enable/provision Pages automatically using `enablement: true`, but the GitHub Actions integration did not have the required permission to perform that repository-level setup.

**Correction:** Removed the `enablement: true` setting from the workflow. GitHub Pages must be enabled/configured for the repository through its Settings → Pages page, with **Source = GitHub Actions**. The workflow then uses the normal configure, upload-artifact and deploy-pages actions.

### Issue B — Browser module resolution

**Risk identified and addressed in the viewer:** Three.js add-on modules such as OrbitControls import `three` by its bare module name. A browser cannot resolve that name without an import map or a compatible bundler.

**Correction:** The viewer defines an import map mapping `three` and `three/addons/` to the matching jsDelivr Three.js r160 module paths. This avoids the common unresolved bare-specifier error in a native browser-module setup.

### Issue C — Mobile interaction and rendering

The viewer uses OrbitControls configured for touch interaction and includes camera/view controls. Actual behavior still depends on the iPhone browser supporting WebGL and being able to reach the external CDN module URLs.

## 4. Deployment workflow

The workflow in `.github/workflows/pages.yml`:

1. Runs on pushes to `main` affecting `index.html` or the workflow file, and can also be started manually.
2. Checks out the repository.
3. Runs `actions/configure-pages@v5`.
4. Uploads the repository contents as a Pages artifact using `actions/upload-pages-artifact@v3`.
5. Publishes the artifact using `actions/deploy-pages@v4`.

Required workflow permissions are `contents: read`, `pages: write` and `id-token: write`.

## 5. Timeline and elapsed debugging window

The available GitHub Actions timestamps establish this sequence (UTC, 19 November 2026):

| Time | Event |
|---|---|
| 17:40:42 | First deployment workflow run started; failed |
| 17:41:49 | Second run started; failed |
| 17:44:01 | Another run started after workflow revision; failed |
| 17:50:45 | First successful run after the Pages workflow/configuration correction |
| 17:51:14 | A subsequent deployment run also succeeded |

**Measured elapsed window:** 10 minutes 3 seconds from the first run start (17:40:42) to the first successful run start (17:50:45). The subsequent successful run began at 17:51:14.

This is wall-clock elapsed time across the recorded automation runs, not a measurement of continuous model compute time. It includes GitHub Actions queue/runtime and any human-side repository setting changes. The exact active AI-only debugging time and exact duration of the human intervention cannot be reconstructed reliably from the available timestamps, so they are not presented as measured values.

## 6. Verification status

- GitHub Actions run **44581511596**: completed successfully.
- GitHub Actions run **44581594193**: completed successfully.
- Earlier failed runs: **44579749361**, **44579945085**, **44580327697**.
- Latest run history: https://github.com/Pinakbakshi/nocturne-glasshouse/actions

A successful Pages workflow is evidence that GitHub's deployment job completed. It does not, by itself, prove that every device/browser can render the 3D scene. Final acceptance should include opening the published URL in iPhone Safari, waiting for the CDN modules to load, and testing rotate, pinch-zoom, camera presets and fullscreen.

## 7. Practical lessons

1. Keep repository-level Pages enablement separate from the deployment workflow unless the workflow token explicitly has permission to manage Pages settings.
2. For native browser ES modules, map bare module specifiers with an import map or use a bundler.
3. Keep the GitHub Actions permissions explicit and minimal.
4. Treat a green deployment run and a successful end-user browser test as two separate verification stages.
5. When testing on mobile, verify WebGL availability and network access to the module CDN as well as touch controls.

---
*This report records the implementation and debugging evidence available in the conversation and GitHub Actions history. It distinguishes verified events from items that still require a real-device browser acceptance test.*
