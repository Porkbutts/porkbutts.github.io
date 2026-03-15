---
author: Adrian Teng-Amnuay
pubDatetime: 2026-03-15T12:00:00Z
title: "An agent that writes its own tools"
featured: false
draft: false
tags:
  - agentic-ai
  - claude
  - playwright
description: "I needed a CLI for a web app with no public API. I had Claude Code intercept network calls via Playwright while it navigated the CRUD flows itself. It captured the API, then wrote a CLI from what it found. The interesting part isn't the technique. It's what it means when agents can build their own integrations."
---

I volunteer for a small nonprofit. We publish events to a bunch of platforms: Wix, Discord, Google Calendar, Facebook. I've been automating that with Claude Code skills and MCP servers so I can just give Claude the event details and it handles the rest.

One of the platforms we use for RSVPs doesn't have a public API. No docs, no SDK. Just a web UI. So every time we needed to create an RSVP page, someone had to go click through forms manually. That broke the automation chain.

I wanted a CLI that could create events on this platform programmatically. The platform wasn't going to give me one. So I figured I'd reverse engineer their API.

## The original plan

My initial idea was pretty simple. Claude Code has access to a Playwright MCP server that can control a browser. I'd log in, navigate the site myself, and have Claude inject a fetch interceptor to record every API call happening in the background. Basically using Claude as a smart network logger while I click around.

I logged into the site in the Playwright browser. SMS auth, so that part had to be me. And then once I was in, I was about to start clicking through the create event flow and I realized... I'm about to type in a bunch of test data into form fields. Claude can do that. It can see the page via Playwright's accessibility snapshot, it can click buttons, it can fill in text. Why am I the one doing this?

So the plan shifted. Claude would inject the fetch interceptor AND navigate the UI. I just watched.

## How the interception works

Before navigating anywhere, Claude injects JavaScript that monkey-patches `window.fetch`. Every request and response gets logged to a `window.__capturedRequests` array: URL, method, headers, body, response. It's not fancy. It just captures everything the site's frontend sends to its backend.

Then Claude walks through a flow. Clicks "Create Event," fills in a title, picks a date, types a description, submits. After the flow completes, it reads back the captured requests. Now it has the exact endpoints, the auth headers, the request body shape. Everything you'd get from Chrome DevTools, but captured programmatically.

I had Claude repeat this for create, update, image upload, delete. Each flow reveals different endpoints. The site turned out to use Firebase and Firestore under the hood, with a mix of custom API endpoints and direct Firestore REST calls. Image uploads were a two-step process: upload the file to a dedicated endpoint, then PATCH the Firestore document to link the image metadata to the event. That's the kind of thing you wouldn't guess from the UI.

For some features, Claude didn't even need the interceptor. To catalog available themes and visual effects, it just browsed the selection UI and pulled the names from the DOM and CDN asset URLs. To figure out how the "make it public" toggle worked, it captured the Firestore document before and after flipping the switch and diffed the fields.

## Where it got messy

The first flow, creating an event, went smoothly. Claude injected the interceptor, navigated the form, captured the requests, and had a clear picture of the create API. Clean.

It got worse as the session went on. By the time we were working on edit and delete flows, Claude had started to forget how to set up the interception properly. It would navigate to a page without injecting the interceptor first, or it would inject it but then not read back the results. I had to keep re-explaining: inject first, then navigate, then read back.

Part of this was context. Playwright is context-heavy. Every snapshot of the page, every screenshot, every JavaScript evaluation eats tokens. I had to compact the conversation two or three times, even on a 1M context window. Each compaction meant Claude lost some of the working knowledge from earlier in the session. So I'd re-explain the interception method, Claude would get it right for one flow, and then we'd compact again and I'd re-explain again.

It worked. But it was more like supervising a capable but forgetful coworker than pressing a button.

## From captured requests to CLI

Once Claude had the full set of intercepted requests across all the CRUD flows, it wrote a Python CLI using only stdlib. No external dependencies. The CLI handles auth (Firebase phone auth with token refresh), event CRUD, and image uploads, including the two-step upload-then-link process that the interceptor revealed.

I'm glossing over this part because honestly it was the least interesting. Claude had the request/response pairs. Writing code that replays those requests with different parameters is the easy part. The hard part was getting the recordings in the first place.

## What this is actually about

The more I build "agent as personal assistant" workflows, the more I see the same pattern. What makes agents powerful is their ability to interface with external systems. Today that happens through APIs, CLIs, MCP servers, or in the worst case, browser automation.

There's a clear hierarchy there. An API call is fast and reliable. Browser automation is slow and fragile. You always want to be as far up that hierarchy as possible.

What's interesting about this case is the agent almost bootstrapped itself up the hierarchy. It started at the bottom, using browser automation to navigate a site with no API. It captured the API calls that the browser was making. Then it wrote a CLI that makes those calls directly. The next time it needs to create an event on this platform, it doesn't need Playwright at all. It just runs the CLI.

The agent wrote its own tool to avoid needing browser automation in the future.

All it actually needed from me was two things: log in (because SMS auth requires a human with a phone) and the instructions for how to set up the fetch interceptor. Everything else, the navigation, the capture, the analysis, the CLI, Claude handled.

There are still places where humans stay in the loop. Logging into systems. Reviewing high-stakes writes like sending an email or making something public. But the surface area of "things only a human can do" keeps shrinking. Techniques like this, where an agent builds its own tools from observing a system, could close agentic loops faster and faster.
