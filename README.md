# immersive-vr-tour-webxr

Web version of [Spring2024ImmersiveVRTour](https://github.com/EmpathyBytes/Spring2024ImmersiveVRTour), built with Unity WebGL + [WebXR Export](https://github.com/De-Panther/unity-webxr-export).

The goal is to let visitors open the Empathy Bytes VR tour in a browser and press **Enter VR** in a headset (Quest Browser) without installing an app. This is a separate Unity project so the Quest app stays untouched and Meta's XR SDK never mixes with the web packages.

**Test build:** https://empathybytes.github.io/immersive-vr-tour-webxr/

Right now the project contains the WebXR Interactions sample scene only. It proves the setup works before the Freshman Dorm is brought in.

---

## Current status

| Step | Status |
|---|---|
| Web project set up with WebXR Export (this setup) | Done |
| Sample scene enters VR in Chrome with the Immersive Web Emulator | Done |
| Sample scene enters VR in Quest Browser | Needs testing on a Quest |
| Bring the Freshman Dorm into the project | Next |
| Swap Meta's camera rig for the WebXR rig | Later |
| Shrink the build, final hosting, launch page on the website | Later |

---

## Requirements

- **Unity 2022.3.62f2** (same version as the Quest project) with the **WebGL Build Support** module
- **Git LFS** (`brew install git-lfs`, then `git lfs install`)
- For testing without a headset: Chrome with the [Immersive Web Emulator](https://chromewebstore.google.com/search/Immersive%20Web%20Emulator) extension

## Opening the project

1. Clone the repo. Git LFS downloads `.fbx` and image files automatically.
2. In Unity Hub: **Add > Add project from disk**, select the repo folder, and open it with **2022.3.62f2**.
3. The first open imports all packages and takes a few minutes. `Library/` is regenerated locally and is not in git.

---

## Packages

| Package | Version | Why |
|---|---|---|
| `com.de-panther.webxr` (WebXR Export) | 0.25.0 | Adds WebXR support to Unity WebGL builds |
| `com.de-panther.webxr-interactions` (WebXR Interactions) | 0.25.0 | WebXR rig, input and the XR Interaction Toolkit sample |
| `com.unity.xr.interaction.toolkit` | 2.6.5 | Interactions (grab, teleport, UI). The WebXR sample needs 2.5.2 or later, and its guide was mostly tested on 2.x. XRI 3.x renamed many APIs, so the latest 2.x was chosen. |
| `com.unity.xr.hands` | 1.5.1 | Required by Project Validation for the hands sample |
| `com.unity.render-pipelines.universal` | 14.0.12 | URP, same pipeline as the Quest project |
| `com.unity.inputsystem` | 1.14.0 | Installed as a dependency of XR Interaction Toolkit |

The WebXR packages come from OpenUPM through a scoped registry in `Packages/manifest.json`:

```
Name: OpenUPM
URL: https://package.openupm.com
Scope: com.de-panther
```

---

## How the project was set up

These are the steps that were done, in order, so they can be repeated or checked.

1. Created a **Universal 3D (URP)** project in Unity 2022.3.62f2 and switched the platform to **WebGL**.
2. Set up Git LFS for `.fbx` and image files (see `.gitattributes`). `.gitignore` uses GitHub's Unity template plus `.DS_Store` and `.vscode/`.
3. Added the OpenUPM scoped registry and installed WebXR Export, WebXR Interactions and XR Interaction Toolkit.
4. **Window > WebXR > Copy WebGLTemplates**: copies the templates into `Assets/WebGLTemplates`.
5. **Project Settings > XR Plug-in Management > Web tab**: enabled **WebXR Export**.
6. **Project Settings > Player > Resolution and Presentation > WebGL Template**: **WebXR2020**.
7. **Project Settings > Player > Other Settings > Active Input Handling**: **Both**.
8. **Project Settings > Input System Package > Background Behavior**: **Ignore Focus** (saved in `Assets/InputSystem.inputsettings.asset`), so controller tracking isn't lost when the browser tab loses focus.
9. **Project Validation**: fixed every issue. This installed XR Hands and imported the XRI **Starter Assets**, **Hands Interaction Demo** and XR Hands **HandVisualizer** samples.
10. Imported **XR Interaction Toolkit Sample** from WebXR Interactions. Its scene is at `Assets/Samples/WebXR Interactions/0.25.0/XR Interaction Toolkit Sample/Scenes/WebXRInteractionToolkit.unity` and is the only scene in Build Settings.

### Other settings that matter

- **Color Space: Gamma.** WebXR Export requires Gamma on Unity 2022 (it logs a warning on Linear). On Unity 6 it would be Linear instead.
- **Decompression Fallback: on** (Player > Publishing Settings). The build is Brotli-compressed. Hosts that can't send `Content-Encoding: br` headers, like GitHub Pages, still work this way, but the first load is slower. Turn it off once the build is on a host that sends the right headers.
- **WebGL template variables.** The WebXR2020 template uses two custom variables, `DESCRIPTION` and `WEBXR_POLYFILL_CONFIG`. They are set in Player Settings (`m_TemplateCustomTags` in `ProjectSettings/ProjectSettings.asset`). If they are missing, the build fails with `ReferenceError: DESCRIPTION is not defined`.
- **WebXR settings must be registered.** `Assets/XR/Settings/WebXRSettings.asset` has to be listed as `WebXR.Settings` in `ProjectSettings/EditorBuildSettings.asset`. Opening the WebXR page in XR Plug-in Management does this automatically. If it is missing, the page loads but pressing **VR** throws `Cannot read properties of undefined (reading 'VRRequiredReferenceSpace')`.

---

## Building

1. **File > Build Settings**, platform **WebGL**, scene `WebXRInteractionToolkit` checked.
2. Click **Build** (not **Build And Run**) and choose `Builds/WebGL`.
   - **Build And Run** serves the page over HTTP, and WebXR only works on HTTPS or `localhost`.
   - `Builds/` is in `.gitignore`. The build is about 11 MB.

### Testing locally

Serve the build folder and open it in Chrome:

```bash
python3 -m http.server 8080 --directory Builds/WebGL
```

Then open `http://localhost:8080`. `localhost` counts as a secure context, so WebXR works without HTTPS.

Without a headset the **VR** button stays gray, because the browser reports that no VR device is available. To test Enter VR on a computer:

1. Install the **Immersive Web Emulator** extension in Chrome and pin it to the toolbar.
2. Open the page and click the extension's toolbar button to **enable emulation for this site** (it is off per site by default).
3. Reload, wait for the page to load, and click **VR**. The emulator shows a floating panel with the headset and both controllers.

---

## Deploying the test build (GitHub Pages)

The test build is hosted from the **`gh-pages`** branch. That branch only contains the build output (`index.html`, `Build/`, `TemplateData/`, `.nojekyll`) and is **never merged into `main`**. `main` holds the Unity project.

To update it after a new build:

```bash
git clone --branch gh-pages https://github.com/EmpathyBytes/immersive-vr-tour-webxr.git gh-pages-deploy
cd gh-pages-deploy
git rm -rq . && git clean -fdq
cp -R ../immersive-vr-tour-webxr/Builds/WebGL/. .
touch .nojekyll
git add -A
git commit -m "Update test build"
git push
```

GitHub Pages republishes in about a minute. Browsers cache the old build, so hard-refresh (`Cmd+Shift+R`) when testing a new one.

GitHub Pages is only used for testing. Choosing the final host (with proper compression headers) is a separate step.

## Testing on a Quest

1. Open **Quest Browser** and go to https://empathybytes.github.io/immersive-vr-tour-webxr/
2. Wait for the first load (about 20 to 30 seconds).
3. Press **VR** in the bottom-right corner and allow the browser to enter VR.
4. It works if the view switches to an immersive 360° scene and the controllers show up.

---

## Repo layout

```
Assets/
  Samples/            Imported samples (WebXR XRI sample, XRI Starter Assets, Hands demo, HandVisualizer)
  WebGLTemplates/     WebXR2020 and WebXRFullView2020 templates from WebXR Export
  XR/                 XR Plug-in Management settings and the WebXR loader/settings assets
  Settings/           URP render pipeline assets
  InputSystem.inputsettings.asset
Packages/manifest.json   Package list and the OpenUPM scoped registry
ProjectSettings/         Player, XR, input and build settings
```

## References

- [WebXR Export: Using XR Interaction Toolkit](https://github.com/De-Panther/unity-webxr-export/blob/master/Documentation/Using-XR-Interaction-Toolkit.md)
- [Unity: Deploying WebGL builds](https://docs.unity3d.com/2022.3/Documentation/Manual/webgl-deploying.html)
