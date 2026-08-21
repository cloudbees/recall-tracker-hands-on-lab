# Module 07 — Discovery, and where flags come from

**35 minutes** · Session 3: Feature management

Two things happen in this module. First you use the application properly — a real
discovery against live recall data. Then you connect Feature Management and find five
flags already waiting for you, which nobody created.

---

## 1. Run a discovery

Sign in to your `PROD` URL as `enterprise@example.com` and start a discovery.

Use these inputs. They are chosen because they return data:

| Field | Value |
|---|---|
| Company Name | anything |
| Product Category | `Medical Devices` |
| Supply Chain Role | `Manufacturer` |
| Distribution Region | `OR` |
| Employee Count | `1200` |

The discovery calls `recall-worker`, which queries the FDA's openFDA API live, and
builds a compliance matrix from what comes back.

> **Category and region both matter, and an empty result is a real answer.** Some
> combinations genuinely have no recalls on record — a category nobody has recalled in
> that state returns nothing, correctly. If your matrix is empty, try `Medical Devices`
> with `CA` before assuming something is broken.

Open the compliance matrix. Those requirements were derived from recall records that
existed before this workshop started. Nothing here is seeded fixtures.

---

## 2. Connect Feature Management

Everything so far has been build and release. This is the other half.

In your sub-organization, open **Feature Management** and copy your SDK key. Then set it
as `FM_KEY` — the variable you have been leaving as `unset` since Module 02 — on all
three environments.

### Then redeploy

The SDK key is read into each pod's environment when it deploys. Changing the variable
changes nothing that is already running, so `recall-core-api` and `recall-ai-service`
need to come back up with the new value.

Create a new release and run it through all three stages.

That is worth noticing on its own: **configuration changes need a deploy, and this is
the last time in this workshop that will be true.** Everything you change from here
takes effect without one.

---

## 3. Five flags you did not create

Go back to **Feature Management**. There are five flags in the list:

| Flag | Type |
|---|---|
| `recall.recallAdvisor` | boolean |
| `recall.exportPdf` | boolean |
| `recall.calendarView` | boolean |
| `recall.errorState` | boolean |
| `recall.headerTheme` | string, four variants |

Nobody added them. You did not create them in the UI, and no workflow registered them.

They came from the code. `packages/shared/src/fm/flags.ts` declares them:

```ts
export const featureFlags = {
  recallAdvisor:    new Rox.Flag(false),
  exportPdf:        new Rox.Flag(false),
  calendarView:     new Rox.Flag(false),
  errorState:       new Rox.Flag(false),
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

## 4. One warning, because it cannot be undone

> **Never delete a flag to reset it.**
>
> Deleting a flag in Feature Management is irreversible, and it reserves the name. The
> SDK cannot recreate it, and reusing that name needs a CloudBees Support request.
>
> This matters most for `recall.headerTheme`. A string flag's variant list is fixed when
> it first registers. If you want different variants, ship a new flag name — do not
> delete the flag to force it to re-register, because there will be nothing to
> re-register into.

Worth knowing in the room rather than discovering in your own organization later.

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
