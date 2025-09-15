# Bigfoot
# EightSecond Vlogger

EightSecond Vlogger is a conversational, low-code workspace for planning and rendering 8-second character vlog scenes with Vertex AI (Veo 3). It scaffolds project setup, character continuity, prompt assembly, and a lightweight render queue with mock rendering for demos.

## Quick start

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to explore the builder.

### Default flow

1. **Project Setup** – define delivery specs (aspect ratio, resolution, FPS, seed, guidance). Duration and model are fixed (8 seconds, `veo-3`).
2. **Character Builder** – capture anchors, negative constraints, reference images, and optional TTS settings.
3. **Scenes** – populate eight ordered cards (camera recipe, environment, beats, dialogue, audio, constraints) and hit **Validate** to enforce ≥2 beats plus dialogue.
4. **Render Dashboard** – queue single scenes or batches, watch status updates, and download the results. The queue runs two jobs at a time.

All data is stored locally in `localStorage` so you can refresh without losing progress.

## Mock mode vs. live mode

The sidebar toggle controls whether renders hit Vertex AI or return instantly from a placeholder pipeline.

- **Mock mode (default)** – waits a couple seconds then copies `public/mock-placeholder.mp4` into `public/renders/<job>.mp4`. Duration is pinned to 8.0 seconds.
- **Live mode scaffold** – the API layer assembles the full prompt and hands off to the `liveRender` function. Wire this up with Vertex AI credentials to enable real calls. Until credentials are supplied it will return an informative error in the job logs.

### Vertex AI integration notes

The render queue is implemented in `src/lib/renderQueue.ts`. When integrating with Vertex AI:

- Use the supplied `buildPromptPackage(project, character, scene)` helper for prose prompts and model parameters (seed, guidance, duration, reference images).
- Replace `liveRender` with a call to the Veo 3 SDK/REST API. Ensure outputs are trimmed to exactly 8.0 seconds and copied into `public/renders` for download links to work.
- Optional TTS metadata is exposed in the job object so you can synthesize dialogue before muxing audio.

## API surface

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/jobs` | `GET` | List current/previous render jobs with status, logs, and output URLs. |
| `/api/jobs` | `POST` | Queue a single scene for render (mock or live depending on toggle). |
| `/api/jobs/batch` | `POST` | Queue multiple scenes at once, respecting the two-job concurrency limit. |

Responses intentionally include only lightweight job metadata so the dashboard can poll without heavy payloads.

## Project structure highlights

- `src/context/BuilderContext.tsx` – shared state backed by `localStorage`.
- `src/lib/prompt.ts` – deterministic prompt template enforcing continuity language and duration notes.
- `src/lib/renderQueue.ts` – minimal in-memory queue with mock + live hooks.
- `src/components/*` – reusable UI building blocks (scene editors, chip inputs, etc.).
- `public/mock-placeholder.mp4` – 8-second placeholder clip used in mock mode.

## Testing & linting

```bash
npm run lint
```

The project uses Next.js App Router with Tailwind CSS for styling. Feel free to adapt the components for production deployments.
