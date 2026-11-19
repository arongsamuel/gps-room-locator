# Microsoft Teams app

The same room finder as a **personal tab** in Teams. Teams simply shows the hosted `index.html`, so there is no extra backend and no change to the privacy model.

## Install

1. Host the site (the manifest points at `https://atlasroomfinder.netlify.app/`; change `contentUrl`, `websiteUrl`, `developer.*Url` and `validDomains` if you host it elsewhere).
2. Zip the **contents** of this folder (`manifest.json`, `color.png`, `outline.png`, all at the zip root).
3. In Teams: **Apps → Manage your apps → Upload an app**
   - **Upload a custom app**, if your organisation allows it, or
   - **Submit an app to your org**, for admin approval.
4. Allow location for the app in its settings (**App permissions**).

## What's different inside Teams

- The page loads Microsoft's TeamsJS library (pinned with a Subresource Integrity hash), and **only** when it detects it's running in Teams. In a normal browser nothing extra is loaded.
- If browser GPS isn't passed through, the page falls back to Teams' own location API.
- Outlook sign-in uses Teams' authentication pop-up.
- **Limits:** Teams desktop has no GPS, and compass and step sensors may be unavailable in the Teams mobile webview. The browser version always has all features.

The manifest uses schema v1.19. Change `version` whenever you change the manifest.
