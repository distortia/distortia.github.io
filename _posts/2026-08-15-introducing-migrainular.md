---
layout: post
title: "Introducing Migrainular"
date: 2026-08-15
categories: [product, launch]
tags: [migrainular, phoenix, svelte, pwa, privacy]
author: Nick Stalter
excerpt: "A private, local-first migraine diary you can fill out in about 20 seconds—even on a bad day. Phoenix API, Svelte PWA, no ads, not medical advice."
image: /img/migrainular-home.png
---

I have chronic migraines. Day in, day out. Typically, I am not in the mood to open a spreadsheet, or an app with 1000 fields to enter. I want to tap a number, maybe tap a medication, and lie back down.

That is the whole product. I spend 20 seconds or less filling out my diary entry.

<figure>
  <img class="shot-wide" src="/img/migrainular-home.png" alt="Migrainular home: a private diary, 20 seconds or less, three steps if it hurts right now">
  <figcaption>The home screen. No walkthrough after you sign in.</figcaption>
</figure>

[Migrainular](https://migrainular.com) is a private migraine diary. It lives on your phone as a Progressive web app(PWA). Days and attacks save on the device first. Sync is optional, when you choose, so you can restore on another device or print something for a clinician. It is a personal tracker—not a diagnosis, not a treatment, not medical advice.

I already owned the name. There was an older Phoenix app under it that stored a handful of fields and was not a diary I would actually use during an attack. I started over.

## The constraint

A useful attack log has to be completable in about **20 seconds**. Pain level is the only required field. Everything else is a chip you can skip.

If you have logged before, **Same as last time** copies the previous attack—pain, location, aura, meds, symptoms—so you can glance and save. Notes, timing, and weather stay with this attack, not the last one.

<figure>
  <img class="shot" src="/img/migrainular-entry.png" alt="Diary entry form: only pain level is required, Same as last time, type and location chips">
  <figcaption>Pain is the only required field. Everything else is a chip you can skip.</figcaption>
</figure>

The day card (sleep, water, food, caffeine, workout, CPAP AHI) is a different job. That is "when you have more energy." It is not on the way to Save.

<figure>
  <img class="shot" src="/img/migrainular-today.png" alt="Today screen with Add diary entry, Same as last time, weather, and the day card">
  <figcaption>Today: log an attack at the top. The day card can wait.</figcaption>
</figure>

I have been ruthless about this. If a new field would make someone in pain think, it does not belong on the form.

## What you can log

**Attacks:** pain 0–10, type, location, aura, skippable chips for nausea and light/sound/smell sensitivity, when it started, whether it is still going, and medications.

**Medications** are a library in Profile, not a pharmacy. You list what you take. Rescue meds can show as one-tap chips on an attack. A monthly preventive can stay on the list without appearing every time you log. Deletes sync across devices.

**Days:** sleep, water (glasses or ounces), food, caffeine, workout, and—if you use a machine—**CPAP - AHI** (events per hour), not a vendor 0–100 score.

**Weather** is optional. OpenWeather is proxied through Phoenix so the API key never sits in the app. Barometric pressure gets stamped onto entries when you are online. Offline, you still save.

**Trends** are on-device charts: pain over time, overlays for sleep, caffeine, barometer, and AHI. **Export** is CSV plus a printable provider summary, including your current medication list.

Sign in is a magic link. No password. Logout clears the diary on that device; sign back in and pull from your account. There is a **Clear this device** control if you lent someone a browser.

## Why a PWA, not LiveView

I spend a lot of time in Phoenix LiveView—[OnRotation](https://onrotation.app) is built that way, and I have written about making LiveView feel [offline-ish](/phoenix-liveview-offline-ish-list-with-localstorage-and-a-follow-up-event). For a meal plan on the couch, a reconnect cache is enough.

For a migraine log, it is not. The save button has to work with no network, in airplane mode, with a flaky cell connection, after the tab has been killed. The client owns the write: IndexedDB first (Dexie), then a boring last-write-wins sync when you are online.

So Migrainular is:

- **Svelte 5** PWA in `web/` (Vite, service worker, Add to Home Screen)
- **Phoenix** JSON API for magic-link auth, sync, and the weather proxy
- **Postgres** on Fly, same neighborhood as OnRotation
- Dark by default. Low glare. No reminder nags.

Diary contents never go to analytics. If Simple Analytics is on, it is anonymous pageviews with Do Not Track respected. In-app feedback posts to Discord without the diary attached. Delete account deletes the server copy.

## Try it

Migrainular is live at [migrainular.com](https://migrainular.com). Sign in with email, add it to your home screen if you want, and the next time it hurts: pain, save, done.

