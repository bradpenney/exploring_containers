# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Repository Overview

**Exploring Containers** teaches containerization as its own topic domain — not a Kubernetes prerequisite, not a single-tool cheat sheet. It covers what a container actually is, the runtimes underneath it (containerd, CRI-O, runc), Docker and Podman as tools, image building and distribution, and the security/supply-chain discipline production demands. It follows the same editorial standards, tier model, and shared visual brand as the other `exploring_*` sites (see `~/notes/STYLING.md` for the shared brand layer).

**This is a full-tier, eventually-monetized site** — same treatment as Exploring Kubernetes, Linux, Python, and CS. It is NOT a hobby site like `exploring_electronics` or `exploring_cnc`: it gets tiers, personas, a paywalled Mastery tier, and is a strong future [[project_pathways|Pathway]] candidate once it has depth.

## Site Boundary (read before writing anything k8s-adjacent)

Containerization content is deep enough to straddle this site, Exploring Kubernetes, and Exploring Linux. Keep the split clean — no repetition across sites:

- **This site owns:** what a container is (namespaces/cgroups as they apply to containers), the OCI image & runtime specs, container runtimes (containerd, CRI-O, runc), Docker (build/networking/volumes/Compose), Podman (rootless, pods concept, systemd/quadlet integration), image building/layers/distribution, container security (rootless, capabilities, seccomp/AppArmor, scanning, supply chain/SBOM), and the general sidecar/init-container *pattern* as an orchestrator-agnostic concept.
- **Exploring Kubernetes owns:** kubelet↔CRI mechanics, how Pods consume containers, and `initContainers`/sidecar containers as k8s API/manifest features — the k8s-specific mechanics, not the general pattern. Cross-link into this site rather than re-explaining runtimes or the CRI from scratch.
- **Exploring Linux:** no ownership here — cross-link only where namespaces/cgroups already come up in a Linux systems-administration context.

## Tier Model

Four-section model shared across all `exploring_*` sites: **Day One → Essentials → Efficiency → Mastery**.

| Tier | Directory | Persona |
|------|-----------|---------|
| **Day One** | `day_one/` | The developer whose app has to become a container — two scenarios, one shared foundation then a fork: (1) a single app/service (web app, API, ETL job) being containerized for the first time, to ship to a team or land on a platform, or (2) a monolith someone (a manager, a platform team) has said must leave VMs, forcing the reader to think about microservice scope for the first time |
| **Essentials** | `essentials/` | Someone building images and running containers with intent — knows Docker or Podman exists, wants the real mental model underneath (namespaces/cgroups, what a runtime does) |
| **Efficiency** | `efficiency/` | Platform engineer / senior dev going beyond a single container — composition (Compose, Podman pods), self-hosted registries, the CRI, capability dropping and scanning |
| **Mastery** *(paywalled tier)* | `mastery/` | Platform engineer / SRE who owns this in production — OCI spec depth, runtime selection (containerd vs. CRI-O vs. gVisor/Kata), signing, SBOMs, supply chain trust |

**Persona targeting is internal — never spell it out in published content** (same rule as every other `exploring_*` site). No "Who This Is For: the developer who inherited a container" sections in the actual articles. Frame everything as content/outcomes; the personas above live in this file and in memory only. A broad "Who Is This For?" section on `index.md` framed around reader situation/pain/goal is fine (see the landing page already written) — that's different from exposing the tier-targeting strategy inside an article.

## Topic × Tier Grid (initial sketch, 2026-07-12)

Topics are nav-label groupings, not directories — articles live flat inside each tier directory (`essentials/first_dockerfile.md`, not `essentials/docker/first_dockerfile.md`) until a topic within a tier has 3+ articles, then split it into a subdirectory (same convention as Exploring Kubernetes' `efficiency/networking/`).

| Topic | Essentials | Efficiency | Mastery |
|---|---|---|---|
| Container Fundamentals | Namespaces & cgroups: what actually isolates a container | Multi-container patterns (sidecar/init/ambassador, orchestrator-agnostic) | The OCI Image & Runtime Specs, line by line |
| Docker | Your First Dockerfile; Docker networking basics | Docker Compose; volumes & bind mounts | Multi-stage builds & BuildKit |
| Podman | Podman for Docker users; rootless containers | Podman pods; Quadlets (systemd integration) | Rootless in production |
| Container Runtimes | Meet containerd & CRI-O | The CRI: how Kubernetes talks to containerd; runc & the OCI runtime | Choosing a runtime for production (containerd vs. CRI-O vs. gVisor/Kata) |
| Images & Registries | Image layers & caching; pushing/pulling with a registry | Tagging & versioning; self-hosting a registry (Harbor/Distribution) | Image provenance & SBOMs |
| Container Security | Security basics: what isolation does and doesn't protect | Dropping capabilities & seccomp profiles; scanning images (Trivy + CI gate) | Signing & verifying images (cosign, Sigstore) |

**Day One articles** (paradigm-level, not yet in the grid above since Day One predates topic grouping): a 3-article shared foundation (Overview; What Is a Container, Really? — this also owns the image-vs-container distinction; Getting Docker Running on Your Machine — CLI-first per-OS install, added 2026-07-15, deliberately no Docker Desktop) that then forks into two parallel 4-article pathways, mirroring the kubectl/Helm fork in Exploring Kubernetes' Day One:

- **Containerizing a Single App** — Writing Your First Dockerfile → Building and Running Your Image → Debugging a Container That Won't Behave → Sharing It With Your Team (registry push, handoff toward "getting it on a platform").
- **Breaking Up a Monolith** — Why the Monolith Has to Move → Finding the Seams: How to Scope a Microservice → Containerizing the First Slice → Living With a Partial Migration.

Day One owns the "what is a container" framing — Essentials' Container Fundamentals topic starts one level deeper (namespaces/cgroups) and does not repeat it. The monolith pathway stays at decision-making depth (how to recognize a seam, what makes a slice safe to peel off first) — it is not a distributed-systems or DDD treatise; that's out of scope for Day One and arguably out of scope for this site entirely.

Cross-link seams for later: Container Runtimes ↔ Kubernetes (kubelet/CRI article); Container Fundamentals' sidecar/init pattern ↔ Kubernetes `initContainers` article; Container Security's rootless ↔ Linux security content.

## Important Preferences

**Git Operations**: The user handles all git operations (commits, pushes, etc.) themselves. Do not commit or push changes.

**MkDocs Operations (updated 2026-07-30):** `poetry run mkdocs build --strict` is allowed for testing/verification — use it to confirm changes actually build cleanly before handing off. `mkdocs serve` is allowed too if a live preview is genuinely needed, but only on a non-default port (3000 is almost always occupied by something else) and only as a short-lived test — never left running. The user still handles real preview sessions and all deploys.

## SEO Strategy and Publication Process

**CRITICAL**: This site uses a draft/publish workflow to ensure only vetted content appears in search engines and the sitemap.

### Required Metadata for Every Article

```yaml
---
date: "2026-01-15 12:00"
title: Clear, Descriptive Title (50-60 chars ideal)
description: Compelling description for search results (150-160 chars ideal)
---
```

- **Date**: Required — the RSS feed dates entries from this field. Omitting it logs an RSS build warning and leaves the feed entry undated.
- **Title**: Unique across the site, descriptive. If it contains a colon, quote the whole string (unquoted colons cause PyYAML to misparse frontmatter silently).
- **Description**: Summarize what the reader will learn.
- Titles >60 chars and descriptions >160 chars get truncated in search results.

### The Exclude Plugin Strategy

**Day One is published (2026-07-14, +1 article 2026-07-15)** — all 11 articles (3-article shared foundation + the app/monolith pathways) are uncommented in `nav:` and removed from the `exclude` glob. `mkdocs.yaml` still excludes `essentials/*`, `efficiency/*`, `mastery/*` wholesale. As those articles are vetted and published, switch from the tier-wide glob to individually-listed remaining drafts (same pattern the other sites use once a tier has its first live article — see Exploring GitOps' or Exploring Kubernetes' `mkdocs.yaml` for the shape this takes).

### How to Publish an Article

1. Complete the [Quality Standards Checklist](#quality-standards-checklist) below.
2. Remove the article's path from the `exclude` plugin's `glob` list in `mkdocs.yaml` (remove the whole line, don't comment it — if the whole tier glob is still in place, replace it with the remaining individual draft paths).
3. Uncomment the article in the `nav:` section.
4. Verify: `poetry run mkdocs build --strict`, then check `site/sitemap.xml` includes the new URL.
5. Update this file's exclude-configuration notes to reflect what's still excluded.

### Common SEO Mistakes to Avoid

1. Linking to unpublished articles — check the exclude list first.
2. Forgetting to update the exclude list when publishing.
3. Missing metadata — every article needs date, title, description.
4. Leaving articles in navigation but still excluded — nav and exclude list must align.

## CRITICAL: No Repetition — Respect Reader's Time

1. **Cross-link instead of repeating** — if a concept is explained elsewhere (including on Exploring Kubernetes or Exploring Linux), link to it.
2. **Only repeat for significantly different perspectives** — brief intro vs. deep dive is fine; the same explanation twice is not.
3. **Progressive depth, not repetition** — each article builds without re-explaining the previous one.
4. **Audit before publishing** — use the Explore agent to search for repeated concepts across published articles in the same tier before marking any article complete.

## Content Guidelines

### Tone and Style by Tier

- **Day One**: empathetic, reassuring, concrete-before-abstract. The reader is doing this for the first time under real pressure — a team waiting on a deployable artifact, or a mandate to get off VMs — acknowledge that without being condescending. Safety/blast-radius framed gently ("this won't break anything you're exploring read-only").
- **Essentials**: peer-to-peer. Drop the hand-holding Day One used. Lead with *why* a design works the way it does (why namespaces isolate what they do, why an image is layered instead of a flat file). Assume competence, not expertise.
- **Efficiency**: platform-engineer register. Operational ownership assumed — multi-container composition, registries, the CRI, capability dropping. Production consequences, not just mechanics.
- **Mastery**: production/SRE register. Full technical depth, no apologies for complexity — spec-reading, runtime trade-offs, supply-chain trust. This is the paywalled tier; content here should feel like it's worth paying for (real depth: SBOM internals, signing verification chains, runtime isolation guarantees, not survey-level overviews).

**Core principles (every tier):**

- Empathetic or peer-appropriate openings anchored in a real scenario, not abstract definitions first.
- Wit in parentheticals and asides, not emoji spam (limit 1-3 per article, used strategically).
- Explain the *why* before the *how* — a design decision before the command that exercises it.
- No "Excited to share" energy anywhere; no over-the-top marketing language ("incredible", "game-changing").

**⚠️ Watch the formula, not just the topic (2026-07-28):** "You've done X" asserts the reader's own history — a bet that fails whenever someone in the audience didn't do that exact thing, and it reads as alienating when it misses. Prefer grounding the hook in the practice/scenario itself: "Working code that runs fine on one machine is the easy part" instead of "You've got working code. It runs fine on your machine." Still concrete, still scenario-first — just not a bet on this specific reader's biography.

**Required Sections (every substantial article):**

1. Opening hook with real-world relevance (tier-appropriate register)
2. Core content with commands/config and actual output shown
3. Safety/blast-radius notes where destructive operations are involved
4. Common pitfalls / "Watch Out For"
5. Practice exercises with nested solutions (`??? question` containing `??? tip "Solution"`)
6. Quick Recap / Key Takeaways
7. What's Next
8. Further Reading, organized by category (Official Docs, Deep Dives, Related Articles)

### Container-Specific Writing Guidelines

**Command and tool names in prose:** always wrap in backticks — `docker`, `podman`, `containerd`, `crictl`, `runc`, `buildah`, `skopeo`, `cosign`, `trivy`.

```markdown
✅ "Use `docker build` to create an image"
❌ "Use docker build to create an image"
```

**Command output blocks:**

```markdown
``` bash title="List Running Containers"
docker ps
# CONTAINER ID   IMAGE     COMMAND     STATUS         PORTS     NAMES
# 3f2a91c7e88b   nginx     "nginx…"    Up 2 minutes   80/tcp    web
```
```

**Dockerfile / config blocks:** always use titles, line numbers, inline annotations.

```markdown
``` dockerfile title="Dockerfile" linenums="1"
FROM node:20-slim AS build  # (1)!
WORKDIR /app
COPY package*.json ./
RUN npm ci  # (2)!
```

1. Multi-stage build — this stage never ships in the final image
2. `ci` not `install` — reproducible from the lockfile
```

**Command safety labels:**

- ✅ **Safe (Read-Only):** `docker ps`, `docker logs`, `docker inspect`, `podman ps`, `crictl images`
- ⚠️ **Caution (Modifies State):** `docker build`, `docker run`, `docker exec`, `docker compose up`
- 🚨 **DANGER (Destructive):** `docker rm -f`, `docker system prune -a`, `docker volume rm`

**Anti-patterns to avoid:**

- Don't teach `:latest` as an acceptable production tag anywhere past Day One — pin digests or explicit versions.
- Don't recommend running containers as root without explicitly flagging it as a decision, not a default (rootless is the standard being taught, per [[feedback_teach_highest_standard]] — rank alternatives, label weaker ones legacy/anti-pattern).
- Don't conflate Docker-the-tool with "containers" generically — be precise about what's Docker-specific vs. universal to the OCI model.
- **Never let Docker silently monopolize** (Brad likes Podman a lot, 2026-07-14): wherever Docker is taught — Day One included — Podman gets a solid explicit callout (rootless by default, daemonless, CLI-compatible, `alias docker=podman`). Day One's canonical callout lives in `day_one/app/first_dockerfile.md`; Essentials+ treats Podman as a full peer per the topic grid.

### Article Layout and Visual Structure

Same visual toolkit as the other `exploring_*` sites:

**Visual density is a standing requirement (Brad, 2026-07-14: reader feedback says the sites aren't visual enough).** Every substantial article gets at least one mermaid diagram; find the *structural* shape in the content (decision tree, fork, gate, hub-and-spoke, side-by-side comparison) — never a linear A→B→C flowchart that restates numbered steps.

1. **Mermaid diagrams** (workflow/architecture) at the top of concept articles — slate + amber accent scheme:
   - Standard Node (Slate 800): `fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff`
   - Highlighted Node (Amber 600): `fill:#d97706,stroke:#cbd5e0,stroke-width:2px,color:#fff`
   - Darker Node (Slate 900): `fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff`
   - Warning/Danger (Red 600): `fill:#c53030,stroke:#cbd5e0,stroke-width:2px,color:#fff`
2. **Card grids** (`<div class="grid cards" markdown>`, opt into `two-col`) — "why it matters" before commands/config.
3. **Content tabs** (`=== "Tab Name"`) for tool alternatives — "Docker" vs. "Podman", "containerd" vs. "CRI-O".

Context always precedes commands/config — never open a section with raw syntax.

### Cross-Linking Strategy

- Link forward/backward within this site's learning path.
- Cross-link to **Exploring Kubernetes** for kubelet/CRI/Pod-consumption/`initContainers` mechanics — don't re-teach them here.
- Cross-link to **Exploring Linux** where namespaces/cgroups already appear in a systems-administration context.
- Cross-link to **Exploring GitOps** where image builds feed a CI→registry→Flux pipeline.

## Quality Standards Checklist

Before uncommenting an article in `mkdocs.yaml`:

**✅ Content Quality**

- [ ] No-repetition audit against other published articles in this tier (and against Kubernetes/Linux where the site-boundary split applies)
- [ ] Opening hook matches the tier's persona and register
- [ ] Commands/config shown with real, verified output
- [ ] Safety/blast-radius notes where relevant
- [ ] Practice exercises with nested solutions
- [ ] Quick Recap, What's Next, Further Reading (categorized)

**✅ Technical Accuracy**

- [ ] Commands tested or validated against current tool versions
- [ ] Config (Dockerfile/Compose/Quadlet/etc.) is valid and follows the standard this site teaches (rootless-first, pinned tags, least-privilege)
- [ ] External links validated with WebFetch before publishing

**✅ Formatting**

- [ ] All code blocks have `title=` (and `linenums="1"` for Dockerfiles/YAML)
- [ ] Tool/command names backticked in prose
- [ ] Blank lines before all lists
- [ ] Admonitions use a meaningful type — never generic `note`
- [ ] Frontmatter present: `date`, `title` (quoted if it contains a colon), `description`

**✅ Integration and Links**

- [ ] Pre-publication link audit — never link to an unpublished article
- [ ] Cross-links added between published articles in the same tier
- [ ] "Part of [Tier]" callout where applicable
- [ ] Site-boundary cross-links to Kubernetes/Linux where relevant, not duplicated content

## Project Structure

- `docs/` — Markdown content in the four-tier model (`day_one/`, `essentials/`, `efficiency/`, `mastery/`), flat within each tier until a topic group needs its own subdirectory
- `docs/images/` — logo (`exploring_containers.png`) and diagrams
- `docs/stylesheets/extra.css` — shared amber-phosphor brand layer (identical across all `exploring_*` sites) + site-specific functional CSS
- `mkdocs.yaml` — site configuration and navigation (everything currently commented out — nothing published yet)
- `pyproject.toml` — Poetry dependencies

## Common Commands

```bash
# Install dependencies
poetry install

# Serve locally (http://localhost:8000)
poetry run mkdocs serve

# Build static site (ALWAYS use --strict for link validation)
poetry run mkdocs build --strict
```

## Final Notes

This site teaches containerization as a subject in its own right — the runtime, the image, the tool, the isolation guarantees — so that Kubernetes, Linux, and every other site can assume the reader already understands what a container is instead of re-teaching it. When in doubt, favor precision about what's Docker-specific vs. universal to the OCI model, and teach the rootless/pinned/least-privilege path as the default, not the advanced option.
