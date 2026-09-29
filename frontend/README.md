# Frontend

KROMA: Next.js / React / TypeScript, MapLibre, Zustand, TanStack Query.

From the root: `npm ci --prefix frontend`, then `npm run dev --prefix frontend`. Backend by default http://127.0.0.1:8000; `KROMA_BACKEND_URL` changes the proxy at start-up/build. `NEXT_PUBLIC_API_BASE_URL` specifies the direct origin API instead of the proxy if deployment is required.

Pages: Overview `/`, Events `/events`, Analytics `/analytics`, Analysis `/analytics?tab=area`, Data `/explorer`, Predict `/predict`, About `/about`, Research `/research`. The Operator menu contains information about the team.

SCENARIO clearly flags synthetic dynamics. TRAIN prediction shows in-sample warning. TEST is not on the map. Predict accepts existing full TRAIN package; JPEG is not issued as a model entry.

Checks: `npm run typecheck`, `npm run lint`, `npm test`, `npm run build` from `frontend`. Full instruction and demo - in the root [README](../README.md).
