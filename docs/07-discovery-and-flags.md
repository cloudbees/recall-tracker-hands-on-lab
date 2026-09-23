# Module 07 — Discovery, and where flags come from

**35 minutes** · Session 3: Feature management

Two things happen in this module. First you use the application properly — a real
discovery against live recall data. Then you connect Feature Management and find five
flags already waiting for you, which get created by the code.

---

## 1. Run a discovery

Sign in to your `PROD` URL and create a new account to start a discovery.

The discovery calls `recall-worker`, which queries the FDA's openFDA API live, and
builds a compliance matrix from what comes back.

> **Category and region both matter, and an empty result is a real answer.** Some
> combinations genuinely have no recalls on record — a category nobody has recalled in
> that state returns nothing, correctly. If your matrix is empty, try `Medical Devices`
> with `NY` and see if you get a different result.
>
> You can either change the URL from matrix to discovery or create a new account to
> get back to the discovery page.

---

## 2. Connect Feature Management

Everything so far has been build and release. This is the other half.

In your sub-organization, open **Feature Management** and copy your SDK keys. The easiest
way to obtain them is creating a dummy flag. Then you can copy each environments SDK key
and set the `FM_KEY` — the variable you have been leaving as `unset` since Module 02 — on all
three environments.

### Then redeploy

The SDK key is read into each pod's environment when it deploys. Changing the variable
changes nothing that is already running, so `recall-core-api` and `recall-ai-service`
need to come back up with the new value.

Create a new release and run it through all three stages.

That is worth noticing on its own: **configuration changes need a deploy, and this is
the last time in this workshop that will be true.** Everything you change from here
takes effect without one.

> [!NOTE]
> On key security
> 
> Your web UI serves its Feature Management SDK key from `/api/fm-config`, to anyone
> who asks. You can open it in a browser and read the key.
> 
> That is correct, and deliberate. The browser-side SDK runs on the user's machine, so
> it needs a key the user's machine can read — the same way a publishable Stripe key or
> a Google Maps API key works. It permits reading flag configuration and nothing else.
> 
> Flag *changes* go through the Unify UI, authenticated as you.

---

## 3. Five flags you did not create

Go back to **Feature Management**. There are five flags in the list:

| Flag | Type |
|---|---|
| `recall.recallAdvisor` | boolean |
| `recall.exportPdf` | boolean |
| `recall.calendarView` | boolean |
| `recall.dashboardRedesign` | boolean |
| `recall.headerTheme` | string, four variants |

Nobody added them. You did not create them in the UI, and no workflow registered them.

They came from the code. `packages/shared/src/fm/flags.ts` declares them:

```ts
export const featureFlags = {
  recallAdvisor:     new Rox.Flag(false),
  exportPdf:         new Rox.Flag(false),
  calendarView:      new Rox.Flag(false),
  dashboardRedesign: new Rox.Flag(false),
};

export const headerTheme =
  new Rox.RoxString('default', ['default', 'dark', 'vibrant', 'branded']);
```

When a service starts with a valid SDK key, the SDK reports the flags it finds in the
code, and the platform records them.

**That is the boundary worth taking away from this module.** A flag's *existence* and
its *default* live in the code, with the developer. A flag's *value* lives in the
platform, with whoever operates it. Neither side has to ask the other for permission,
and neither can invent a flag the other has not agreed to — because a flag nobody
declared in code is a flag nothing reads.

Notice also that every default is `false`. Until now your application has been running
with all of these off, which is why you have not seen a Recall Advisor, a PDF export or
a calendar view. They were built and deployed the whole time.

---

## Before you move on

- [ ] A compliance matrix with requirements in it, from a live discovery
- [ ] `FM_KEY` set on all three environments
- [ ] A release run after setting it
- [ ] Five flags visible in Feature Management, all of them off
- [ ] You can say which side owns a flag's existence, and which side owns its value

Every flag is off, and the features behind them are already deployed. Module 08 turns
them on.

---

**Next:** [Module 08 — Flipping flags](08-flipping-flags.md)
