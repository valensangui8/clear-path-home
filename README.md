# ▶️ [Watch the demo video on YouTube](https://youtu.be/pDMhwLp6dIc)
## 🎞️ [Watch in full HD on Google Drive](https://drive.google.com/file/d/1kf4Bf6HnrVT5OelpiZ6-Gd9X2HAKNMBZ/view)

# Clear Path Home: getting 50,000 Knicks fans home without a crush

> After the Knicks win, Clear Path Home fuses official alerts, social posts and 311 reports, has Jev verify each one, and gives every fan 3 crowd-aware routes home.

Built in 45 minutes at the **Plug and Play × PMAI Hackathon: Rapid Response (#AIWeekNY)**, Challenge 2: *"The Knicks are going to win their second championship in town, and people are hyped. Could official alerts, social media updates and public reports help map street closures and plan alternate routes home?"*

## In 60 seconds: what the app does, step by step
It's 11:15 PM, the Knicks just won at Madison Square Garden, and 50,000 fans hit Midtown at once. Clear Path Home takes each fan from *"everything is closed"* to *"here's my way home"*:

1. **Collect.** On open, it pulls every signal at once: NYPD and Notify NYC alerts, MTA status and turnstiles, DOT traffic sensors, social posts and 311 reports. A radar sweeps out from your location and paints every street by congestion.
2. **Verify (Jev).** **Jev** (TypeSafe's decision model) decides what each report says (road closed, crowd, station closed, station packed, reopened, hazard, rumor) and how credible it is, with **calibrated probabilities**. A viral *"HERALD SQUARE STATION SHUT DOWN!!!"* gets low credibility, and the MTA's *"Herald Sq is OPEN"* overrides it.
3. **Fuse.** Code, not the model, decides what's closed. An official source counts in full, a public report counts by its credibility, two credible reports confirm a closure, and an official "reopened" clears older reports.
4. **Escalate.** A single unverified report that would change routes only slows them down. It never blocks a street until **a human confirms it** in the operator console.
5. **Locate.** It uses your live GPS (or a pinned spot) and lets you pick where you're going: Brooklyn, Queens, Hoboken, the Upper West Side or Grand Central.
6. **Route.** It gives 3 options per mode, each with an arrival time:
   - 🚗 **Drive:** congested blocks in red and the delay each costs.
   - 🚇 **Transit:** walk + wait + ride, where the wait depends on how packed the platform is.
   - 🚶 **Walk:** routes around closures and crowds.
7. **Recommend.** The ranking prefers the fastest option but penalizes crowds above 70%. Every card shows its load and how many app users are already heading there.
8. **Spread the crowd.** Every fan who follows a recommendation adds load to that station or road, so the next riders are shifted elsewhere. The app **spreads people out instead of herding them into one crush**.
9. **Wait it out.** If your option is packed, it suggests a nearby place to wait 20 minutes (a 24h diner, Moynihan Train Hall, Bryant Park) and shows how much the crowd drops by then.
10. **Navigate.** Tap Go for Waze-style turn-by-turn directions with a live ETA.

**Result:** instead of 50,000 people converging on Penn Station, each fan gets a verified, explainable route home, and the load is balanced across stations and streets.

- 🎬 **Demo video file (1:53):** [demo-video/knicks.mp4](https://github.com/valensangui8/clear-path-home/blob/main/demo-video/knicks.mp4) · subtitles: [`.srt`](demo-video/knicks.srt) · [`.vtt`](demo-video/knicks.vtt) · [narration script](demo-video/knicks-script.md)
- 🌐 **Live app (mobile-first):** https://rapid-response-command.vercel.app/knicks
- 🖥️ **Operator console:** https://rapid-response-command.vercel.app/knicks/ops

## The problem
After a championship, official alerts lag behind reality, and social media is fast but full of rumors. Navigation apps don't know about pedestrian frozen zones or packed platforms. Worse, they send everyone to the same "best" option, which is exactly how crushes happen.

## What Clear Path Home does
| Feature | How it works |
|---|---|
| **Live signal map** | Official alerts, MTA status, DOT sensors, social posts and 311 reports are fused into one map. Closures are drawn in red, crowds in orange, unconfirmed reports dashed, and stations show their load %. |
| **Jev verification** | Two typed questions per report: what kind it is (`choice`) and whether it's a first-hand or official observation (`noul`), both with calibrated probabilities. An LLM turns free-text reports into street segments. |
| **Explicit fusion rules** | Official = weight 1.0, public = 0.45 × credibility, confirmed at ≥ 0.8, and a newer official "reopened" wipes older evidence. Reports with weight < 0.1 never touch the map. |
| **Human in the loop** | Single unverified reports that change routes are penalized, not blocked, until an operator confirms or dismisses them (`/knicks/ops` and the ⚠ sheet). |
| **3 routes × 3 modes** | Drive (congestion-colored, closed or crowded streets off-limits), transit (walk + load-based wait + ride, closed stations skipped), walk (crowd-slowed, closures avoided). |
| **Crowd balancing** | Simulated fans pick options (most follow the app), load builds on chosen stations and roads and decays over time, and recommendations shift. A ⏩ button fast-forwards 8 minutes. |
| **Wait-it-out stops** | When your option is packed, it suggests a calm nearby place and the forecast load after 20 minutes. |
| **Waze-style navigation** | Turn-by-turn banner, route progress, live ETA and the number of app users on your route. |

## Architecture
```
alerts + social + 311 ─▶ Jev (choice: kind, noul: credibility) ─┐
                     └─▶ LLM geocoder (text → street segments) ─┴─▶ code policy: fuse evidence ─▶ closures / crowds / station status
                                                                                                        │
mocked traffic + station load + simulated fans ─────────────────────────────────────────────────────────┴─▶ 3 routes × 3 modes ─▶ navigation
```
- `src/lib/knicks.ts`: Midtown grid, demo signal feed, fusion rules, places to wait
- `src/lib/knicksAi.ts`: Jev questions and the LLM geocoder
- `src/lib/knicksSim.ts`: mocked traffic, crowds and station load, k-shortest routes per mode, the self-reinforcing load loop
- `src/components/KnicksApp.tsx`, `WazeMap.tsx`: mobile app (opening scan, routes, navigation) and map
- `src/components/KnicksDashboard.tsx`: operator console
- `src/lib/gen.ts`: rotates across free AI Gateway models (free tier is ~5 req/min per model)

## Run it
```bash
npm install
vercel env pull .env.local          # AI Gateway auth (VERCEL_OIDC_TOKEN)
echo "TYPESAFE_API_KEY=..." >> .env.local   # optional
npm run dev                         # http://localhost:3000/knicks
```
**No Jev key? Nothing stops.** Without `TYPESAFE_API_KEY`, judgments fall back to keyword rules. Every model call has a timeout and a fallback.

## Regenerate the video
Recorded with [Argo](https://github.com/shreyaskarnik/argo) (Playwright + local Kokoro TTS, no API keys):
```bash
npm run build && npx next start -p 3211
cd video && npm install && npx playwright install chromium
BASE_URL=http://localhost:3211 npx argo pipeline knicks     # → video/videos/knicks.mp4 (+ .srt/.vtt)
```

## Simulated in the demo
The signal feed (16 reports) is sample data. Traffic, station load, the crowd of other app users and the source counters in the opening scan are mocked. Routing runs on a Midtown street grid (14th–59th St, 10th–1st Ave) without one-way streets, and travel beyond Midtown (bridge, tunnel, train ride) is a fixed estimate. The Jev judgments in the app and the video are live model outputs.
