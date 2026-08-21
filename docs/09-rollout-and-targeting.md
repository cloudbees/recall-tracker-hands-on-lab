# Module 09 — Progressive rollout and targeting

**40 minutes** · Session 3: Feature management

Everything in Module 08 was on or off for everyone. That is a feature toggle.

This module is the other thing a flag can be: a **rule, evaluated per user, every time**.
That is the step from toggling features to choosing who gets them — and it is the part
most organisations actually buy feature management for.

---

## 1. Ship to 10% of users

Take `recall.exportPdf` — a low-risk feature to be wrong about — and instead of enabling
it for everyone, set a percentage split: **10% on, 90% off**.

Now sign in as each of your three demo accounts in turn and look for the export button on
the compliance matrix.

Some will have it, some will not. Refresh a few times as the same user and notice
something more important than the split itself:

> **The same user stays in the same bucket.** A user who has the feature keeps having it;
> a user who does not, keeps not having it. The split is a consistent hash of the user's
> identity, not a coin toss per request.
>
> This is the difference between a rollout and a flicker. If 10% were re-rolled on every
> page load, every user would see the button appear and disappear at random — which is a
> worse experience than either state, and would make the feature impossible to support.

Now move it: 10% → 50% → 100%, checking as you go.

That progression is the whole point. You have just shipped a feature to production
incrementally, with an audience you chose, and you could stop or reverse it at any point
in seconds. Compare that with the alternative: ship to everyone at once and hope, or
maintain a separate branch until you are confident.

---

## 2. Target by who the user is

A percentage is blunt — it does not care *who* is in the 10%. Often you want a specific
audience: paying customers, one region, internal staff, a single account that reported a
bug.

The application already tells Feature Management who each user is. When someone signs in,
it sets custom properties from their company record:

```ts
Rox.setCustomStringProperty('companySize', companySizeBucket(employeeCount));
Rox.setCustomStringProperty('state', props.state);
Rox.setCustomStringProperty('productCategory', props.productCategory);
```

`companySize` is derived from employee count:

```ts
if (employeeCount >= 500) return 'enterprise';
if (employeeCount >= 100) return 'mid-market';
return 'small';
```

Which puts your three demo accounts in three different buckets:

| Account | Employees | `companySize` |
|---|---|---|
| `enterprise@example.com` | 1200 | `enterprise` |
| `midmarket@example.com` | 250 | `mid-market` |
| `small@example.com` | 45 | `small` |

### Build a target group

Create a target group matching `companySize` equals `mid-market`, and use it to enable
`recall.recallAdvisor` for that group only.

Then sign in as each of the three accounts.

Only `midmarket@example.com` gets the Recall Advisor. Both neighbours are excluded — and
that is worth doing deliberately, because a rule that includes one of three proves more
than a rule that includes one of two. With two buckets you cannot tell targeting from a
coin flip.

You changed nothing about the users, and nothing about the application. You wrote a rule.

---

## 3. Why this is the interesting part

An enterprise customer asks for a feature. You build it, and now you want to give it to
them without giving it to everyone.

The alternatives are all bad. A separate branch means a merge you will do badly later. A
per-customer build means an image per customer. A configuration file means a deploy per
change and no way to reverse it quickly.

A targeted flag is one rule, changed in seconds, reversible in seconds, evaluated per
user per request — and the code has exactly one path through it.

That is also how a beta programme works, how a regional rollout works, and how you give
an unhappy customer an escape hatch at 4pm on a Friday without shipping anything.

---

## Try this if you have time

- Combine conditions: `companySize` is `enterprise` **and** `state` is `OR`. Only
  `enterprise@example.com` matches, and only because of the discovery you ran in
  Module 07.
- Put `recall.headerTheme` behind a target group, so `branded` reaches one customer
  and everyone else keeps `default`.
- Set a percentage split on a flag you have targeted, and work out which applies first.

---

## Before you move on

- [ ] `recall.exportPdf` rolled from 10% to 50% to 100%, checked at each step
- [ ] You saw the same user stay in the same bucket across refreshes
- [ ] A target group on `companySize`, and one flag reaching one account only
- [ ] You can say why consistent hashing matters more than the percentage itself
- [ ] You can name three things this replaces: a branch, a per-customer build, a
      config-file deploy

---

**Next:** [Module 10 — Wrap-up](10-wrap-up.md)
