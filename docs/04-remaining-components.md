# Module 04 — The remaining components

**45 minutes** · Session 1: Unify building blocks

Four more components, built the same way as the first. No new concepts in this module —
which is itself the point. By the end you will have five components and five images, and
a clear sense of why doing this by hand does not scale.

---

## The four

Create a Component from each repository, and run its **build** workflow.

| Component | What it is | Worth noticing |
|---|---|---|
| `recall-worker` | Discovery. Calls the live FDA and CPSC APIs. | Deploys a service with no route in from outside. Only things inside your namespace can reach it. |
| `recall-ai-service` | The Recall Advisor. | Owns a feature flag. In Session 3 you will turn this component on and off without touching it. |
| `recall-core-api` | The API surface. | Everything talks to this, and it talks to the database. It is the component that fails loudly when something underneath it is missing. |
| `recall-web-ui` | Pages and the browser-side flags. | The only one with an Ingress. This is where your URL comes from. |

**Names must match exactly**, as in Module 03. The Application finds its components by
name in the next module.

---

## The workflows are identical

Worth confirming rather than taking on trust. Open `build.yaml` in any two of these
repositories and compare them: the same six steps, in the same order. The only
differences are the image name and the path to the `Dockerfile`.

```
docker.io/<your-user>/recall-worker:<commit-sha>
docker.io/<your-user>/recall-ai-service:<commit-sha>
docker.io/<your-user>/recall-core-api:<commit-sha>
docker.io/<your-user>/recall-web-ui:<commit-sha>
```

Five components, five near-identical build workflows. That repetition is normal, and it
is one of the things a template repository is for — but notice that *nothing* about it
knows the other four exist. Each build workflow builds one thing and stops.

---

## Run the four builds

You do not have to wait for one before starting the next. Builds are independent: no
component's build reads another's output, so run them concurrently and get the four
done in roughly the time of one.

**Deploys are not independent, and that difference matters.** `recall-core-api` cannot
usefully start before the database has its tables, and `recall-web-ui` has nothing to
talk to before the API is up. Nothing in these five repositories expresses that ordering.
Hold that thought — it is the whole subject of Module 05.

### Confirm as you go

After the four builds finish, [hub.docker.com](https://hub.docker.com) should show five
repositories, each with a single SHA tag:

```
recall-db          recall-worker      recall-ai-service
recall-core-api    recall-web-ui
```

If one is missing, its build did not push. Open that run and read the failed step.

---

## Do not deploy them

You deployed `recall-db` by hand in Module 03 to see what a component deploy looks like.
Do not do that four more times.

You would have to get the order right yourself, run four workflows, and copy four
artifact IDs out of four evidence tabs — and then repeat all of it every time anything
changed. That is the work Module 05 hands to the Application, and doing it by hand first
would only prove that it is tedious.

You also do not need to note these artifact IDs down. The Application reads them from
the artifacts Unify already recorded when each build pushed its image. Copying values
between browser tabs was a symptom of doing this one component at a time.

---

## When a build fails

| Symptom | Cause |
|---|---|
| The repository is not offered when creating the Component | It was not selected when you installed the GitHub app in Module 01. Fix it on GitHub, in the app's configuration |
| `Typecheck` fails | A real type error, or `npm ci` could not resolve the lockfile. The step names the file |
| `Configure Docker Hub credentials` fails | Your token, not this component. Verify Setup would have caught it, so suspect an expired or revoked token |
| Kaniko fails on the `Dockerfile` path | The workflow expects the repository root as build context. If you restructured the repository, that is why |
| Build is green but no image in DockerHub | Look at the push line in the Kaniko step — it names the destination it actually used |

---

## Before you move on

- [ ] Five Components, named exactly `recall-db`, `recall-worker`, `recall-ai-service`,
      `recall-core-api`, `recall-web-ui`
- [ ] Five green builds
- [ ] Five repositories in your DockerHub account, one SHA tag each
- [ ] Only `recall-db` deployed — the other four built but not deployed

Five components that each know how to build and deploy themselves, and nothing that
knows they belong together. That is the gap Module 05 closes.

---

**Next:** [Module 05 — The Application](05-the-application.md)
