---
layout: post
title: "OnRotation: Technical Architecture of a Phoenix LiveView Meal Planning App"
date: 2026-02-14
categories: [engineering, elixir, phoenix]
tags: [phoenix, liveview, elixir, ecto, architecture]
author: Nick Stalter
excerpt: "A technical walkthrough of the architecture behind OnRotation—household-scoped contexts, LiveView-heavy UX, a JSON API for mobile, and the patterns that keep it maintainable."
---

[OnRotation](https://onrotation.app) is a weekly meal planning app built with Phoenix and LiveView. I've written about the product story before; this post focuses on the technical architecture: how the app is structured, how data flows, and the patterns we rely on to keep it simple and maintainable.

## Tech Stack

- **Phoenix 1.8** with **LiveView**
- **Ecto + PostgreSQL** for persistence
- **Tailwind CSS v4** for styling (CSS variables, no config file)
- **Req** for HTTP (no HTTPoison/Tesla)
- **Resend** for transactional email (magic links, invites)
- **Sentry** for error tracking

Quality tooling: **Credo**, **Dialyzer**, **SoBelow** (security), all wired into `mix precommit`.

## Architecture Overview

The app follows a context-based structure. Core domains:

- **Accounts** – users, households, invites, magic-link auth
- **Meals** – meal library, curated meals, meal packs
- **Plans** – weekly plans and plan entries
- **ShoppingLists** – per-household shopping list with plan-derived items
- **Blog** – public blog with Earmark-based body processing

Almost all domain data is scoped by **household**. A user belongs to one household; meals, plans, and shopping lists are all keyed by `household_id`. That keeps queries straightforward and authorization simple.

## The Scope Pattern

We use an `OnRotation.Accounts.Scope` struct to represent the caller's context:

```elixir
defstruct user: nil, household: nil
```

`phx.gen.auth` gives us `current_scope`, not `current_user`. The scope carries both `user` and `household`, so controllers and LiveViews can consistently pass `current_scope` into contexts. Context functions take `scope` or `household_id` as the first argument and scope all queries by household.

This avoids leaking user-only logic into contexts and keeps "who is acting" and "on which household" explicit.

## Routing and Pipelines

We use separate pipelines and live sessions for different auth levels:

- **Public** – marketing, blog, sitemap, robots (browser_and_seo)
- **Auth routes** – log-in (magic link), registration
- **Authenticated** – plan, meals, shopping list, feedback (live_session :authenticated)
- **Blog admin** – create/edit blog posts (require_blog_author)
- **Meal pack admin** – manage curated meal packs (require_meal_pack_admin)
- **Admin** – metrics dashboard (require_admin)

The **API pipeline** (`/api/v1`) accepts JSON and uses Bearer token auth. It's used by the Android app; web users stay in LiveView.

## Authentication

We're privacy-focused: no passwords, magic links only, and Simple Analytics for basic analytics instead of tracking-heavy alternatives. No cross-site tracking, no ad networks.

Web users sign in via magic links. The flow:

1. User enters email.
2. We create or find the user and send a tokenized link (Resend).
3. User clicks the link; we validate the token and establish a session.

For the mobile API:

1. Client calls `POST /api/v1/auth/magic-link` with email.
2. User receives the link and opens it in a browser; we redirect to a page that calls the API to exchange the token for a Bearer token.
3. Client stores the Bearer token and sends it on subsequent requests.

Both flows end up with a session token; web uses cookies, API uses `Authorization: Bearer <token>`.

## Data Model

The main entities:

- **Household** – one per account; owns meals, plans, shopping list
- **Meal** – name, recipe_url, main_ingredients, extras, tags, complexity (1–3)
- **Plan** – one per week per household (`week_start_date` = Monday)
- **PlanEntry** – links plan to meal, with `day_index` (0=Mon … 6=Sun) and position for multiple meals per day
- **ShoppingListEntry** – per-household; entries come from plan merge or manual "extras"

Meals, plans, and shopping lists are always queried with `household_id` in the `WHERE` clause.

## Shopping List: Plan Merge and Categorization

The shopping list has two sources of items:

1. **Plan-derived** – parsed from `main_ingredients` and `extras` of meals in the current week's plan
2. **Manual extras** – user-added items (e.g. paper towels, milk)

`merge_plan_into_list/2` takes the current week's plan, parses ingredients into segments (comma/semicolon), normalizes them into `item_key`s for grouping, and upserts into `ShoppingListEntry` by `(household_id, item_key)`. Existing rows are never deleted by the merge—only added or updated.

Categorization is done by `ShoppingLists.Categorizer`: whole-word keyword matching against produce, meat/seafood, cheese/dairy, pantry, and a default "items" bucket. Priority order matters (e.g. "egg noodles" matches pantry before cheese) so we can sort the list in a sensible shopping order.

## LiveView and UI

Core features (plan, meals, shopping list) are LiveView pages. We use:

- **Streams** for large, appendable lists to avoid memory growth
- **Functional components** (CoreComponents) for forms, buttons, layout
- **Colocated JS hooks** when we need client behavior (e.g. theme toggle in localStorage)

All LiveView templates start with `<Layouts.app flash={@flash} ...>` and receive `current_scope` from the `live_session` so they can pass it into context calls.

## Design System

Colors and typography live in `DESIGN.md` and CSS variables (`var(--onrotation-primary)`, etc.). Light/dark themes use `data-theme="light"` and `data-theme="dark"`; we also support "system" via `prefers-color-scheme`. Theme choice is persisted in localStorage and applied before paint when possible to avoid flash.

Transactional emails share the same layout and brand colors, with inline styles for email client compatibility.

## Blog and Content

The blog is a separate context (`OnRotation.Blog`) with published posts, tags, and an author-scoped admin. Body content is stored as Markdown in the database and rendered with Earmark at request time. A `BodyProcessor` runs after Earmark to replace {% raw %}`{{meal:N}}`{% endraw %} shortcodes with rendered meal cards—so authors can embed curated meals in posts, and logged-in users can add them to their library in place.

The blog admin is a LiveView (`BlogAdminLive`) behind a `require_blog_author` plug. Authors get a list view, create/edit with autosave draft, tag management, and a preview that matches the public layout. Public routes are controller-based: index (optionally filtered by tag), show by slug, and an RSS feed at `/blog/feed.xml`. The sitemap includes blog posts for SEO. Slugs are derived from titles; tags are many-to-many and filterable on the index.

## API for Mobile

`/api/v1` exposes resources for meals, plans, and plan entries. The API uses the same context functions as the web app; the only difference is auth (Bearer tokens) and response format (JSON). A plug assigns `household_id` from the authenticated user so controllers can call contexts like `Meals.list_meals(household_id, opts)` without extra logic.

## Quality and Consistency

We enforce consistency with:

- **One `alias` per line** – no `alias Foo.{Bar, Baz}`
- **`@moduledoc` and `@spec` on public functions**
- **No `maybe_` prefix** – use descriptive names instead
- **`mix precommit`** – format, Credo, SoBelow, Dialyzer, tests

The `AGENTS.md` and `.cursor/rules` files document these conventions for humans and AI-assisted workflows.

## Summary

OnRotation is a Phoenix app where:

- Household-scoped contexts and a `Scope` struct keep authorization and data access clear.
- LiveView drives the web UX with streams and functional components.
- A small JSON API under `/api/v1` serves mobile with the same business logic.
- Magic links power both web and mobile auth.
- A token-based design system and precommit checks keep the codebase consistent and maintainable.

If you're building something similar—household/multi-tenant, LiveView-heavy, with a mobile API—this architecture is a solid starting point. The main lesson: invest in scoping and context boundaries early; they make everything else easier.

**[Try OnRotation at onrotation.app](https://onrotation.app)** — start with our Starter Pack and plan your week in minutes.
