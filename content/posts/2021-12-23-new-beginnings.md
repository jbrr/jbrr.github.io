---
title: New beginnings
date: 2021-12-23 00:15
draft: true
tags: [career]
description: A start at documenting getting the hang of a new job
---

After almost 5 years at [Tendril / Uplight](https://uplight.com), I've finally moved on and just started a new job at [Tackle](https://tackle.io) as a Senior Software Engineer on a team working on data ingest. It's a super exciting time to be joining Tackle, as they just [secured Series C funding](https://techcrunch.com/2021/12/21/newly-minted-unicorn-tackle-plans-to-use-100-million-series-c-to-accelerate-the-future-of-software-sales/).

I didn't leave Uplight on the best of terms, which is potentially a story for another time. Suffice to say, everyone there was beyond helpful and supportive, and I feel terrible about the way I left things. I'm so grateful that they took a chance on me in the first place, as a junior developer fresh out of code school, as well as the opportunity to advance from a junior engineer to a senior engineer and tech lead of a small team. I learned just an unbelievable amount there. There are some amazing folks working there, their mission is wonderful, and I truly wish the company and employees all the best.

It's been a long time since I started a new job, and the transition from being "the person who can dive into any legacy project and fix it" to being "the person who has absolutely no idea what you're talking about" has been a little rougher than I anticipated. Needless to say, impostor syndrome is back with a vengeance as well. Not that it ever went away, but I was comfortable enough to keep it at bay most of the time. That said, I'm trying to embrace having fresh eyes, and questioning things that folks take for granted.

Tackle is growing at an astronomical rate, going from ~50 employees to ~150 just this year, and planning to double again in 2022. That said, it still feels like a very early stage startup. There's a lot of work to do, which is not at all a jab at the folks who built it. They built what they needed to, and there's no sense in building software that will scale to millions of requests per hour if you're only dealing with hundreds, especially when there are so many moving parts involved. Patterns and architecture ought to evolve as the company does, so time spent optimizing some service that very well might not even be in use in 2 years doesn't seem like time well spent. All that said, there are a few key areas that I've been thinking about that will need to be improved soon.

## Tightening up loose access control
New engineers have access to basically everything on day 1, violating the principle of least privilege. 