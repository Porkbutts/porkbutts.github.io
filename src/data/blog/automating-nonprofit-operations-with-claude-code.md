---
author: Adrian Teng-Amnuay
pubDatetime: 2026-03-05T12:00:00Z
title: "Automating a nonprofit's operations with Claude Code skills and MCP servers"
featured: false
draft: false
tags:
  - agentic-ai
  - claude
  - mcp
description: "I volunteer for a JET alumni nonprofit and built Claude Code skills and custom MCP servers to automate their event publishing across four platforms and monthly newsletter creation. What used to take hours of copy-pasting now takes minutes."
---

I volunteer for JETAASC, the JET Alumni Association of Southern California. It's a nonprofit community of former JET Programme participants. Like most small nonprofits, there's a lot of operational work that falls on a handful of people. Event announcements need to go out on the website, Discord, Google Calendar, and Facebook. A monthly newsletter needs to be assembled and sent to the mailing list. Past events need to be archived on the blog. All of it is manual, repetitive, and time-consuming.

I figured this was a perfect use case for Claude Code.

## The problem

Every time we have an event, someone has to publish it to four different platforms. Each platform has its own format. Wix needs a blog post with rich content and tags. Discord needs a scheduled event with a cover image. Google Calendar needs a calendar entry. Facebook needs a formatted post with specific details. The information is the same every time, just shaped differently for each destination.

The newsletter is more cumbersome. Every month, someone logs into Mailchimp and builds the email by hand in their GUI editor. There are some reusable content blocks, but you're still dragging sections around, formatting text, uploading images, and making sure everything looks right across sections. Event recaps, upcoming events, announcements, job postings, member spotlights. It takes 20-30 minutes even when you know what you're doing, and most of that time is fighting the editor, not thinking about content.

## What I built

I built a set of Claude Code skills and custom MCP servers that automate both workflows. The whole project is [on GitHub](https://github.com/Porkbutts/jetaasc-events/tree/main).

### Multi-platform event publishing

The event publisher is a Claude Code skill. I tell Claude the event details once: title, date, time, location, description, cost, RSVP link, and a flyer image. Then I pick which platforms to publish to and Claude handles the rest.

For Wix, it uploads the flyer, auto-selects tags based on the event content, creates a draft blog post using Wix's Ricos JSON format, shows me a preview, and publishes only after I approve. For Discord, it creates a scheduled event with the cover image attached. For Google Calendar, it creates a calendar entry. For Facebook, it generates formatted copy I can paste in, since Facebook doesn't have a useful API for this.

One thing worth noting: Wix offers an official MCP server, but it's basically a wrapper around their entire API documentation. Every time the agent needs to do something, it has to wade through all of that. So instead, we went through the exercise of publishing a blog post manually once, extracted the specific APIs, resource IDs, and content format we actually needed, and baked that into the skill's reference docs. Now Claude knows exactly which endpoints to hit and how to structure the content without re-reading bloated docs every time.

### Newsletter automation

The newsletter skill walks through each section of our monthly email: announcements, event recaps, spotlights, upcoming events, and job opportunities. I provide the content for each section, and Claude handles the rest.

I provide event details and section contents, and Claude organizes everything, makes sure the formatting is consistent, interpolates it all into our HTML email template, and publishes it as a draft campaign in Mailchimp. Then it shares a link to the draft for us to review before scheduling it to go out. [Here's an example](https://us1.campaign-archive.com/?e=__test_email__&u=c83f204740850ff66ba2d6475&id=0f75f86cee) of a newsletter built entirely with the skill.

## The MCP servers

Wix has an official MCP server, and Google Calendar is covered by my [GSuite MCP server](https://github.com/Porkbutts/gsuite-mcp-server). But I couldn't find official ones for Discord or Mailchimp, and I don't trust random third-party MCP servers. So I used Anthropic's [mcp-builder](https://github.com/anthropics/skills/tree/main/skills/mcp-builder) skill to generate MCP servers for just the APIs I needed: Discord guild scheduled events and Mailchimp campaigns, content, and image uploads.

## The iteration loop

The hardest part wasn't writing the skills. It was getting them right. A skill looks good on paper, but the only way to know if it actually works is to run it and see what comes out.

I ended up with one Claude session writing the skill, and I'd launch separate Claude sessions to test it. When the output wasn't what I expected, I'd copy the transcript back into the skill-writer session and explain: here's what happened, here's what I wanted instead. Then update the skill and test again. I drafted a lot of dummy events and newsletters this way.

What struck me is how much the work has shifted. The actual building, writing the skill, wiring up the MCP server, that part is basically a commodity now. Claude handles it. The real skill is in defining what you want clearly enough that Claude can execute it, and then assessing whether the output is right or not. And when it's not right, articulating what needs to change. It feels less like engineering and more like product design and QA. You're the person who knows what good looks like, and Claude is the one who builds it.

## Why this matters

The total infrastructure here is a couple MCP servers, a few skill files, and a Mailchimp HTML template. No databases, no deployment pipelines, no SaaS subscriptions. JETAASC didn't need a multi-channel publishing platform. It needed someone to wire up the APIs and describe the workflow in a way Claude could follow.

This is the part that I think people underestimate about agentic AI. It's not just for software companies with engineering teams. Any individual or small organization with repetitive operational work can benefit from this. The barrier isn't technical complexity. It's knowing that the tools exist and how to put them together.
