# Module 03 — Your first component

**45 minutes** · Session 1: Unify building blocks

One component, end to end, slowly. By the end of this module `recall-db` will have built
a real container image, pushed it to your DockerHub account, and registered it in Unify
as an artifact you can deploy.

Do it slowly. Module 04 is this same process four more times, so time spent
understanding it here is repaid four times over — read the workflow rather than only
running it.

---

## 1. Create the Component

In your sub-organization, create a new **Component** from your `recall-db` repository.

It appears in the repository list because of the GitHub app you installed in Module 01.
If it is missing, that is where to look — not here.

Unify scans the repository for `.cloudbees/workflows/` and finds two workflows:
`build.yaml` and `deploy.yaml`. That is all a Component is: a repository, and the
workflows inside it.

> **The name must be exactly `recall-db`.** The Application finds its components by
> name in Module 05. A component called `recall-db-sean` or `Recall DB` is skipped
> silently during a release — no error, just a component that never deploys.

---

## 2. Read `build.yaml` before you run it

Open `.cloudbees/workflows/build.yaml` in your repository. Six steps, and each one
exists for a reason worth knowing.

| Step | What it does |
|---|---|
| Checkout | Clones the repository into the workspace |
| Typecheck | `npm ci` then `npm run typecheck` |
| Configure Docker Hub credentials | Uses your `DOCKERHUB_USER` and `DOCKERHUB_TOKEN` |
| Build and push container image | Kaniko builds the image and pushes it |
| Extract artifact ID | Pulls the artifact's UUID out of the build output |
| Publish build evidence | Records what was built, where it went, and how to deploy it |

Three things in there are worth pausing on.

**The typecheck is a real gate, not decoration.** `next build` prints "Skipping
validation of types", so a green build without this step proves the code compiles, not
that it typechecks. Breaking a type and watching this step fail is a worthwhile two
minutes if you have them.

**Kaniko builds without a Docker daemon.** There is no Docker installed anywhere in
this workshop. Kaniko builds the image inside the cluster from your `Dockerfile`, which
is why nothing asked you to install anything locally.

**The image is tagged with the commit SHA, and nothing else:**

```
docker.io/<your-user>/recall-db:<commit-sha>
```

No `latest`. This matters in the next module and it will catch you if you skip it.

---

## 3. Run it

Run the **build** workflow. It takes a few minutes — most of that is `npm ci` and the
image build.

While it runs, watch which step is slowest. Do your own projects follow the same
pattern?

### Confirm the image is real

Open [hub.docker.com](https://hub.docker.com) and look at your repositories. There will
be a `recall-db` repository containing one tag: a long hexadecimal string.

That string is the commit SHA of what you just built. Nothing in this workshop is
simulated — that is a container image, in your registry, built from your repository.

---

## 4. Find the artifact

Go to the run's **Evidence** tab. The build published a table:

| Deploy input | Value |
|---|---|
| `artifact-id` | a UUID |
| `version` | the commit SHA |

**Note both down.** You need them in the next step, and they are easier to copy now
than to find again later.

An *artifact* is Unify's record of a thing you built: which image, from which commit, by
which run. It is what makes a version selectable in a release, and it is how Unify can
later tell you what is running in `PROD` and where it came from.

---

## 5. Deploy it to DEV

Run the **deploy** workflow. It asks for four inputs:

| Input | Value |
|---|---|
| `artifact-id` | the UUID from the evidence tab |
| `version` | the commit SHA from the evidence tab |
| `environment` | `DEV` |
| `component-repo` | leave as `self` |

`version` is the image tag to pull. Since builds push only a SHA tag, this has to be
that SHA.

> **`latest` does not exist.** People try it, so the workflow checks for it and stops
> with an explanation rather than letting Kubernetes fail with `ImagePullBackOff`
> several minutes later — an error that reads like a registry authentication problem
> and is not.

`component-repo` exists because of a Unify detail you will meet properly in Module 05:
a workflow called by another workflow runs in the *caller's* repository context, not
its own. When you dispatch by hand, `self` is correct.

### What the deploy actually does

`recall-db` is not a long-running service. Its deployment runs the database migrations
and then finishes — six migration files, creating the tables everything else needs, plus
the demo accounts using the `LOGIN_PASSWORD` you set in Module 02.

Watch the migration output in the run log. Seeing tables created is the first sign this
application is real rather than a shell.

The deploy is safe to run twice. Migrations that have already run are skipped.

---

## Before you move on

- [ ] A Component named exactly `recall-db`
- [ ] A green build
- [ ] A `recall-db` repository in your DockerHub account, with one SHA tag
- [ ] The `artifact-id` and `version` noted down
- [ ] A successful deploy to `DEV`, with migrations in the log

You have now done by hand everything a single component needs: build it, find the
artifact, deploy that artifact to an environment.

So consider what happens next. There are five components. Every one of them needs
building and deploying, in a particular order, every time anything changes — five
builds, five artifact IDs copied out of five evidence tabs, five deploys run in the
right sequence, and a mistake anywhere in that chain shows up somewhere else entirely.

Nobody does that twice by hand. Components that are always built and released together
are an **Application**, and an Application is the thing that runs this chain for you.

Module 04 builds the other four, because you cannot assemble components that do not
exist yet. Module 05 is where the five become one.

---

**Next:** [Module 04 — The remaining components](04-remaining-components.md)
