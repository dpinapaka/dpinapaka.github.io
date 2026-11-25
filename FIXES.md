# Chime — code fixes

## Changes made

1. **Service worker registration (`src/lib/notify.ts`)**
   - Previously, `registerWorker()` only awaited `navigator.serviceWorker.ready`. That promise does not create a service worker registration, so a fresh install would never activate the app's `public/sw.js` worker.
   - It now calls `navigator.serviceWorker.register()` and stores the returned registration. The worker URL is resolved relative to `document.baseURI`, supporting deployments hosted in a subdirectory as well as at a domain root. Registration remains optional in unsupported or insecure contexts, and failures do not stop the rest of the app.

2. **Running reminders (`src/lib/schedule.ts`)**
   - Previously, `nextOccurrence()` only searched from one minute before the current time and returned the first start time. Once a reminder had been running for more than a minute, it disappeared from the Next Up card even when its end time was still in the future.
   - It now searches the previous 24 hours and chooses the first occurrence that has not ended. This preserves the existing “Running now” countdown and also covers an item that crosses midnight. The form's longest available duration is two hours.

3. **Date minimum (`src/components/ReminderForm.tsx`)**
   - Previously, today's date was captured once when the module loaded. When the app remained open across midnight, the date input's minimum could be stale.
   - Today's date is now recalculated when the form renders.

4. **Relative app entry (`index.html`)**
   - Changed the source entry URL from `/src/main.tsx` to `./src/main.tsx` to align with the Vite configuration's relative `base: './'` and support preview/deployment paths that are not rooted at the domain.

## Validation

- TypeScript schedule utilities were checked with the locally available TypeScript compiler.
- Runtime checks cover a reminder that is running, one that has ended, and the next upcoming reminder.
- A full Vite/React production build could not be run in this environment because the archive does not contain `node_modules`, and package-registry access was unavailable for installing the locked dependencies.
