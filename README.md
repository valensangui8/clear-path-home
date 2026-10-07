# Clear Path Home: getting 50,000 Knicks fans home without a crush

> The Knicks just won the title at Madison Square Garden and Midtown is gridlocked. Clear Path Home fuses official alerts, social posts and 311 reports, has Jev verify each one, and gives every fan 3 crowd-aware routes home by car, transit or walking.

Built in 45 minutes at the **Plug and Play × PMAI Hackathon: Rapid Response (#AIWeekNY)**, Challenge 2: *"The Knicks are going to win their second championship in town, and people are hyped. Could official alerts, social media updates and public reports help map street closures and plan alternate routes home?"*

- ▶️ **Demo video (1:53):** [demo-video/knicks.mp4](demo-video/knicks.mp4) · subtitles: [`.srt`](demo-video/knicks.srt) · [`.vtt`](demo-video/knicks.vtt) · [narration script](demo-video/knicks-script.md)
- 📱 **Live app (mobile-first):** https://rapid-response-command.vercel.app/knicks
- 🖥️ **Operator console:** https://rapid-response-command.vercel.app/knicks/ops

## In 60 seconds: what the app does, step by step
It's 11:15 PM, the Knicks just won, and tens of thousands of fans pour out of MSG at once. NYPD sets a frozen zone, Penn Station closes its 7th Ave entrances, and every map app sends everyone to the same station.

1. **Collect.** When you open the app, it pulls every signal at once: NYPD and Notify NYC alerts, MTA service status and turnstiles, DOT traffic sensors, social posts (X, Instagram, Reddit) and 311 reports. A radar sweeps out from your location and paints every street by congestion. Every station shows how full it is.
2. **Verify (Jev).** For each report, **Jev** (TypeSafe's decision model) decides what it says: road closed, crowd, station closed, station packed, reopened, hazard, or rumor. It also gives a **calibrated credibility score**. A viral *"HERALD SQUARE STATION SHUT DOWN!!!"* gets low credibility, and the MTA's own *"Herald Sq is OPEN"* overrides it.
3. **Fuse with explicit rules.** Code, not the model, decides what's closed:
   - An official source counts in full.
   - A public report counts by its credibility.
   - Two credible independent reports confirm a closure.
   - A newer official "reopened" clears older reports.
   - A single unverified report only slows routes down until **a human confirms or dismisses it**.
4. **Route by mode.** Pick where you're going and how:
   - 🚗 **Drive:** 3 routes, with congested blocks in red and the delay each one costs. Streets that are closed or full of people are off-limits to cars.
   - 🚇 **Transit:** 3 options. Each one is walk + wait + ride, and the wait depends on how packed the platform is. Closed stations are skipped.
   - 🚶 **Walk:** 3 routes that avoid closures and slow down through crowds.
5. **Spread the crowd.** Every fan who follows a recommendation adds load to that station or road, so the next riders are shifted elsewhere. The app **spreads people out instead of herding them into one crush**. If your option is packed, it suggests a nearby place to wait 20 minutes (a 24h diner, Moynihan Train Hall, Bryant Park) while it clears.
6. **Navigate.** Tap Go for Waze-style turn-by-turn directions with a live ETA.

**Result:** instead of 50,000 people converging on Penn Station, each fan gets a verified, explainable route home, and the city's load is balanced across stations and streets.

## The problem
After a championship, the official alerts lag behind reality, and social media is fast but full of rumors. Navigation apps don't know about pedestrian frozen zones or packed platforms. Worse, they send everyone to the same "best" option, which is exactly how crushes happen.

## Why it works
- **Many sources, one verified map.** Official, social and 311 reports are fused with visible weights, not a black box.
- **Calibrated AI, human in the loop.** Jev returns probabilities. Low-credibility rumors never touch the map. Uncertain reports wait for an operator.
- **Load-aware recommendations.** Ranking prefers time but penalizes crowds above 70%, and the load from other users feeds back into the next recommendation.
- **Works with any phone.** It's a mobile-first web app that uses live GPS and needs no install.

## Tradeoffs (what we didn't build, on purpose)
- Traffic, station load and the crowd simulation are **mocked** for the demo. Jev's judgments are live.
- Routing runs on a Midtown street grid (14th–59th St, 10th–1st Ave), with no one-way streets. Travel beyond Midtown (bridge, tunnel, train ride) is a fixed estimate.
- If Jev is unavailable, a keyword fallback keeps the app running.

## Stack
- **Next.js 16** (App Router) + **Tailwind**, **Leaflet** maps.
- **Jev / TypeSafe System One** for typed, calibrated judgments (`choice`, `noul`).
- **Vercel AI SDK + AI Gateway** to turn free-text reports into street segments.
- **Argo** (Playwright + local Kokoro TTS) for the demo video.

## Code map
| File | What it does |
|---|---|
| `src/lib/knicks.ts` | Midtown grid, demo signal feed, evidence fusion rules, POIs |
| `src/lib/knicksAi.ts` | Jev questions (report kind + credibility) and the LLM geocoder |
| `src/lib/knicksSim.ts` | Mocked traffic, crowds and station load; 3 routes per mode; the self-reinforcing load loop |
| `src/components/KnicksApp.tsx` | Mobile app: opening scan, destinations, modes, route cards, navigation |
| `src/components/WazeMap.tsx` | Map: radar reveal, congestion-colored routes, station load bubbles |
| `src/components/KnicksDashboard.tsx` | Operator console (`/knicks/ops`) |

## Run it
```bash
npm install
cp .env.example .env.local   # add TYPESAFE_API_KEY (Jev) and AI_GATEWAY_API_KEY; without them it falls back to keywords
npm run dev                  # http://localhost:3000/knicks
```

Re-render the demo video:
```bash
npm run build && npx next start -p 3211
cd video && npm install && BASE_URL=http://localhost:3211 npx argo pipeline knicks
```
