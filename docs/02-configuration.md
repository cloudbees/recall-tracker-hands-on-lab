# Module 02 — Configuration

**30 minutes** · Session 1: Unify building blocks

This module sets every value the workshop needs, then proves each one works before
anything is built.

That order is deliberate. A wrong DockerHub token will eventually fail — but it fails
inside a build, with an error about registry authentication, three modules from now.
Here it fails in a workflow whose only job is to find it, and the message names the
thing that is actually wrong.

You will not build anything in this module. You will finish it knowing your setup is
sound.

---

## 1. Create your three environments

In your sub-organization, go to **Configurations → Environments** and create three:
`DEV`, `STAGING`, `PROD`.

> **Create them yourself — do not use any you already see.** An environment inherited
> from the parent organization is shared. A value you change in it would land on
> everyone else in the room, mid-session, with nothing to indicate why their deploy
> started going somewhere unexpected.

Each environment carries its own configuration, set as you create it. Your facilitator
will give you the two values that are yours specifically:

| Key | Type | DEV value |
|---|---|---|
| `namespace` | Property | `recall-<your-name>-dev` |
| `hostname` | Property | `<your-name>-dev.<workshop-domain>` |
| `FM_KEY` | Property | `unset` |

`STAGING` and `PROD` follow the same pattern — `recall-<your-name>-staging`,
`<your-name>-prod.<workshop-domain>`, and so on. All three need all three keys.

`namespace` and `hostname` must both be lowercase. A Kubernetes namespace and a DNS
name each reject anything else.

> **`hostname` must be one label in front of the workshop's domain.** Your facilitator
> issued one wildcard TLS certificate, and a wildcard covers a single level: `you-dev.`
> then the domain, nothing deeper. A hostname outside it deploys perfectly happily and
> then fails in the browser in Module 06, with a certificate warning that appears to be
> your browser's fault. Verify Setup checks this for you against the domain your
> facilitator set, so a mistake here surfaces in the next few minutes rather than two
> sessions later.

`FM_KEY` is the Feature Management SDK key, and you do not have one yet — you get it in
Module 07, once flags exist. Setting it to the literal word `unset` now is not a
placeholder for tidiness: a workflow that references a variable which does not exist
dies before its first step runs. More on that below.

---

## 2. Set your organization values

Still in your sub-organization, go to **Configurations → Properties**. These four apply
across all three environments, so they are set once.

| Key | Type | Value |
|---|---|---|
| `DOCKERHUB_USER` | Property | Your DockerHub username |
| `DOCKERHUB_TOKEN` | Secret | The **access token** from Module 00, not your password |
| `kubeconfig` | Secret | Pre-populated by your facilitator |
| `LOGIN_PASSWORD` | Secret | Your choice. Becomes the password for your demo accounts |

**`LOGIN_PASSWORD`** is read while `recall-db` runs its migrations, and it becomes the
password for the three demo accounts you will sign in as later —
`small@example.com`, `midmarket@example.com` and `enterprise@example.com`. Pick
something you will still have in an hour. If it is missing when the database deploys,
the accounts will fail to be created, the deploy still succeeds, and you find out at the
sign-in page.

### What you inherit

You will also see properties marked **Inherited**. Database credentials, the FDA API
key and a few others are set once at the parent organization and flow down.

That is how you will have a working database without anyone handing you a password.

---

## 3. Two rules about values

Both of these were learned the hard way, and both produce errors that name the wrong
thing.

**Every key must exist, even when you have nothing to put in it.** Referencing a
variable that does not exist kills a workflow before any step runs:

```
failed to run workflow: evaluate "${{ vars.FM_KEY }}": vars.FM_KEY is undefined
```

There is no failed step to inspect, because nothing ran. This is why `FM_KEY` is
`unset` rather than absent.

**A property cannot be saved empty.** Hence the literal string `unset` as the
convention for "nothing yet" — the workflows recognise it and treat it as empty. It
also gives you something you can overwrite later, which a blank field does not.

---

## 4. Create the Application

`app-recall-tracker` is the repository holding the workflows that release the other
five together. It needs to exist in Unify for step 5.

Connect the repository as an Application and link your three environments to it. You
will come back in Module 05 to attach the five components; for now it only needs to
exist and know about your environments.

---

## 5. Run Verify Setup

From the Application, run the **Verify Setup** workflow. Choose `DEV`.

Give it two to three minutes. Most of that is the runner starting up, not the checks —
it has not hung.

It checks five things:

| Check | What it proves |
|---|---|
| variables | Every property exists and is not empty |
| secrets | Every secret exists and is not empty |
| dockerhub | Your username and token actually authenticate |
| cluster | Your kubeconfig reaches the cluster and can deploy into your namespace |
| database | The database is reachable and has the extension the app needs |

### Expect WARN, not PASS

A clean run at this stage looks like this:

```
  WARN  variables
  WARN  secrets
  PASS  dockerhub
  PASS  cluster
  PASS  database

Setup is usable, with warnings.
```

**Two WARNs is the correct result.** Everything they name is deliberately not set yet.

`variables` warns about two:

- **`FM_KEY` is `unset`.** Flags fall back to their code defaults, which are off. This
  matters in Module 07 and not before.
- **`S3_BUCKET` is `unset`.** The workshop does not provision S3, so document upload and
  export to S3 do not work. This never becomes a problem.

`secrets` warns about three, all of them optional API keys your facilitator has left
out: `ANTHROPIC_API_KEY`, `VOYAGE_API_KEY` and `ADMIN_PASSWORD`. The Recall Advisor and
semantic search are reduced without the first two, and the admin endpoints stay
disabled without the third — which is intended.

Do not try to fix any of them. Each WARN line says either which module it starts to
matter in, or that it is expected.

A **FAIL** is different, and the line names what to fix. Correct it in the UI and run
the workflow again; it is safe to run as many times as you like.

Then run it twice more, for `STAGING` and `PROD`. Configuration is per environment, so
a pass on `DEV` says nothing about the other two — and a typo in one namespace is
much cheaper to find now than during a release in Module 06.

---

## When a check fails

| Symptom | Cause |
|---|---|
| `dockerhub` FAIL | A DockerHub password was pasted instead of an access token |
| `cluster` FAIL, "not authorized" | A problem with the `kubeconfig` your facilitator set. Tell them — you cannot fix this one |
| `cluster` FAIL, namespace not found | `namespace` does not match what your facilitator provisioned — check the spelling and the environment suffix |
| `database` FAIL | Usually a `namespace` typo — the database lives inside your namespace. If `cluster` also failed, it is the same cause |
| Workflow fails with no steps shown | A property is missing entirely rather than empty. The error names the variable |
| `variables` FAIL rather than WARN | Something other than `FM_KEY` is missing — the line above the summary says which |

---

## Before you move on

- [ ] Three environments created by you, each with `namespace`, `hostname` and `FM_KEY`
- [ ] Every `hostname` ends in the workshop domain
- [ ] `DOCKERHUB_USER`, `DOCKERHUB_TOKEN` and `LOGIN_PASSWORD` set at organization
      scope, with `kubeconfig` already there from your facilitator
- [ ] Verify Setup run against all three environments, each ending WARN or PASS

Everything from here builds on this. Module 03 takes one component and puts it all the
way through.

---

**Next:** [Module 03 — Your first component](03-first-component.md)
