# Module 06 — Release orchestration

**50 minutes** · Session 2: Release orchestration

This is the module where the application becomes real. One release, five components,
three environments, one approval — and at the end of it, your own URL with your own copy
of Recall Tracker behind it.

---

## 1. Read the release workflow

`app-recall-tracker` has a second workflow: `release-wf.yaml`. Four jobs, three stages.

```
  DEV ──► Approve ──►  STAGING ──►  PROD
   └── stage: DEV ──┘   stage 2      stage 3
```

Each of `DEV`, `STAGING` and `PROD` calls the same `deployer.yaml` you read in Module 05,
with a different environment:

```yaml
DEV:
  environment: DEV
  uses: ./.cloudbees/workflows/deployer.yaml
  with:
    manifest: ${{ inputs.manifest }}
    environment: ${{ job.environment }}
```

One deployer, called three times. Nothing about deploying is written three times — which
is why adding a fourth environment later would be a five-line change rather than a
rewrite.

> **What makes this a *release* workflow rather than an ordinary one** is four things
> together: a `workflow_dispatch` trigger, a required string input named `manifest`,
> a `metadata.stages` block naming the stages, and sequential `needs:` between the stage
> jobs. Miss any one and Unify treats it as a normal workflow — it will not appear as an
> option when you create a release. Worth knowing before you write your own.

### The approval gate

So what is the fourth job? Between `DEV` and `STAGING` sits a job that deploys nothing:

```yaml
Approve:
  needs: DEV
  timeout-minutes: 4320
  delegates: cloudbees-io/manual-approval/custom-job.yml@v1
```

It waits for a human. `approvers` is deliberately left blank in this workshop so you can
approve your own release — in a real pipeline you would name people or groups there, and
`disallowLaunchByUser` would stop the person who started the release from also approving
it.

The timeout is 4320 minutes: three days. A gate that expires over a weekend is a gate
that fails releases for reasons unrelated to the software.

---

## 2. Create a release

In your Application, create a new release. Unify offers you a manifest: a list of your
components, each with the versions it can find artifacts for.

**This is where Module 04 pays off.** Four of your components have exactly one version to
choose. The one you pushed a commit to has two — the original, and the one your commit
triggered. Pick either; the point is that you are choosing, and that the choice is
recorded.

Select a version for all five components, and start the release.

---

## 3. Watch the DEV stage

Open the `DEV` stage and expand the `manifest` job first. It prints what the release
actually asked for:

```
=== Deploying to: DEV ===

Expected component keys:
  present  recall-db
  present  recall-worker
  present  recall-ai-service
  present  recall-core-api
  present  recall-web-ui
```

Five `present`. An `ABSENT` line here means a component was not in the manifest and will
not deploy — which is the one failure mode that otherwise looks like success.

Then watch the shape of the run: `DB` alone, then three in parallel, then `Web-UI` last.
That is the dependency graph you read in Module 05, executing.

`recall-db` will report `Up to date, nothing to apply` — you already migrated this
database by hand in Module 03. The other four are deploying for the first time.

---

## 4. Check DEV before you approve

When `DEV` finishes, the release stops and waits for you.

Do not approve it yet. A gate whose answer is always yes is a delay, not a control — so
use it for what it is for, and go and look at what you are about to promote:

```
https://<your-name>-dev.<workshop-domain>
```

Three things to confirm:

- The page loads.
- No certificate warning — your `hostname` is inside the workshop's wildcard.
- You can sign in as `enterprise@example.com`, with the `LOGIN_PASSWORD` you set in
  Module 02. Those accounts exist because you migrated this database in Module 03.

If any of that fails, **do not approve**. Leave the gate where it is, fix the problem,
and start a new release — that is exactly the outcome the gate is there to allow. A
release you cannot decline is not a release process.

Once `DEV` is genuinely working, approve it.

---

## 5. Watch it happen twice more

`STAGING` runs the same deployer against a different environment, then `PROD` after it.
Watch the namespace in each stage's output change:

```
recall-<you>-dev  →  recall-<you>-staging  →  recall-<you>-prod
```

Same images. Same workflow. Different `namespace` and `hostname`, because those are
environment-scoped, and everything else inherited from your organization.

Nothing was rebuilt. The artifacts that went to `PROD` are the exact images that were
tested in `DEV` — same digests, not rebuilt from the same source. That distinction is
most of what release orchestration is for.

---

## 6. Open your application

`https://<your-name>-prod.<workshop-domain>`

Sign in with one of the demo accounts seeded back in Module 03:

| Email | |
|---|---|
| `small@example.com` | a small business |
| `midmarket@example.com` | a mid-market company |
| `enterprise@example.com` | an enterprise |

The password is the `LOGIN_PASSWORD` you set in Module 02.

You now have three running copies of a five-service application, at three URLs, that you
built from source and released through a gate.

---

## When a release goes wrong

| Symptom | Cause |
|---|---|
| The release workflow is not offered when creating a release | One of the four required elements is missing from `release-wf.yaml`, or Unify has not rescanned yet — allow two or three minutes |
| A stage goes green but a component did not deploy | Read the `manifest` job. An `ABSENT` line names it, and the cause is a component name that does not match |
| A component job fails immediately, with no steps run | A variable it needs does not exist in that environment. `STAGING` and `PROD` need `namespace` and `hostname` too, not just `DEV` |
| A deploy fails cloning the repository | `YOUR-GITHUB-ORG` was not replaced everywhere in `deployer.yaml` — check `component-repo` as well as `uses:` |
| The release fails in seconds with "outside the calling workflow's scm organization" | The org in `uses:` does not match your GitHub account's capitalisation. Compare the two URLs in the error — they differ only by case |
| The site loads but every API call fails | `Web-UI` reached `Core-API` before it was ready. Re-run the stage; if it persists, `Core-API` itself failed |
| TLS warning in the browser | `hostname` is outside the workshop's certificate. Verify Setup would have caught it, so check you fixed all three environments |

---

## Before you move on

- [ ] A release with a chosen version for each of the five components
- [ ] `DEV` green, with five `present` lines in the manifest job
- [ ] Your `DEV` URL checked and working *before* you approved
- [ ] An approval you granted yourself
- [ ] `STAGING` and `PROD` green
- [ ] Your application open in a browser at your `PROD` URL, signed in as a demo account

That is the whole delivery path: source, build, artifact, release, gate, three
environments. Session 3 is about changing what it does without touching any of it.

---

**Next:** [Module 07 — Discovery and where flags come from](07-discovery-and-flags.md)
