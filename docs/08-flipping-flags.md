# Module 08 — Flipping flags

**40 minutes** · Session 3: Feature management

Five flags, all off. Everything behind them is already deployed and running.

In this module you turn them on, one at a time, and watch a running application change
behaviour. Nothing redeploys. No workflow runs. Keep a Unify tab and your application
open side by side, because seeing it is the whole point.

Work against `PROD` — you may as well change production, since you can change it back.

---

## How fast, and why

Flag changes reach the running services over Server-Sent Events, so about **two
seconds**. You do not need to refresh for the SDK to have the new value, though you will
often need to refresh for a page to *re-render* with it.

That is the difference worth internalising: a deploy is minutes and replaces a process,
a flag change is seconds and changes a decision the process was already making.

---

## 1. `recall.headerTheme` — the visible one

Start here because the effect is unmissable. It is a string flag with four variants:

| Variant | Appearance |
|---|---|
| `default` | Corporate blue and cyan |
| `dark` | Near-black surfaces, greyscale accents |
| `vibrant` | Saturated purple and pink |
| `branded` | CloudBees blue and purple |

Open your compliance matrix, then change the variant in Unify and refresh the page.

Try `branded`, and consider what it means: you have just re-skinned an application for a
customer without building anything. The variants are declared in the code — the browser
SDK and the server SDK both register the same four — and choosing between them is an
operational decision.

---

## 2. `recall.recallAdvisor` — a component goes dark

This is the flag that justifies a five-component application.

Turn it **on**. Within a couple of seconds a chat launcher appears on the compliance
matrix. Ask the Recall Advisor something about your recalls.

Now turn it **off** while your application is open.

The launcher disappears. And here is the part worth watching properly: **the gate lives
inside `recall-ai-service`**, not in the API that proxies to it. With the flag off, that
service returns `403` to anything that asks — and `recall-core-api` and `recall-web-ui`
stay completely healthy. Every other page keeps working. The compliance matrix still
loads. Discovery still runs.

One service stopped offering a feature. Nothing else noticed.

> **Watch the pods if you have `kubectl`.** Ask your facilitator for access and run
> `kubectl get pods -n recall-<you>-prod -w` while you flip it. Nothing restarts. No
> pod is replaced, no container is recreated, nothing goes `Pending`. A demo that
> claims "no deploy" is much more convincing when the pod ages keep climbing.

That boundary is the argument for feature flags in a distributed system. You cannot
demonstrate it with one service, because there is nothing for the failure *not* to
spread to.

---

## 3. `recall.exportPdf` and `recall.calendarView` — both sides agreeing

These two are read in the browser, in `recall-web-ui`:

```tsx
{fm.isEnabled('recall.exportPdf', false) && ( ... )}
{fm.isEnabled('recall.calendarView', false) && ( ... )}
```

Turn each on and refresh. An export button appears on the matrix; a calendar view
appears in the navigation.

Notice what the code does when the flag is off: the control is not disabled or hidden
with CSS, it is **not rendered at all**. There is nothing in the page to inspect,
re-enable with developer tools, or discover by accident. Server-side, the same flag
gates the endpoint. Both sides read the same flag name from the same platform.

That matters because a client-only flag is a suggestion. A flag read on both sides is a
decision.

---

## 4. `recall.dashboardRedesign` — a kill switch

The last one is not what its name suggests, and that is the point.

Turn it on, then try to sign in. You do not get a new dashboard: you get "Service
temporarily unavailable", and you cannot get in. Refresh and it holds. If you were
already signed in, your next page load signs you out and returns you here.

Now turn it off and sign in. Everything is back.

That is a rollback measured in seconds — no deployment, no revert commit, no pipeline,
no waiting. In a real incident it is the difference between a two-minute outage and a
forty-minute one.

And notice which flag did it. A feature you were looking forward to turned out not to
be ready, and taking it back cost one click. That is the argument for shipping behind a
flag: not that nothing will go wrong, but that going wrong stops being expensive.

Turn it back off before you continue.

---

## What just happened, and what did not

| | |
|---|---|
| Workflows run | 0 |
| Releases created | 0 |
| Pods restarted | 0 |
| Images built | 0 |
| Behaviour changes | 5 |

Everything you changed in this module was already deployed in Module 06. The code paths
existed, the containers were running, and the decision about whether to take them was
being made at runtime, per request, against a value held in the platform.

Which is the honest description of what a feature flag is: **not a switch, but a
decision moved out of the deploy and into the platform.**

---

## Before you move on

- [ ] All four themes tried, including `branded`
- [ ] `recall.recallAdvisor` on, a question asked, then off — with the rest of the
      application still working
- [ ] `recall.exportPdf` and `recall.calendarView` both on
- [ ] `recall.dashboardRedesign` demonstrated, then turned back off
- [ ] You can say why the Recall Advisor gate lives in `recall-ai-service` rather than
      in `recall-core-api`

Every flag so far has been on or off for everybody. Module 09 is where a flag stops
being a switch.

---

**Next:** [Module 09 — Progressive rollout and targeting](09-rollout-and-targeting.md)
