---
title: New beginnings
date: 2021-12-23T00:15:00-07:00
draft: false
tags: [career, engineering]
description: A start at documenting getting the hang of a new job
---

After almost 5 years at [Tendril / Uplight](https://uplight.com), I've finally moved on and just started a new job at [Tackle](https://tackle.io) as a Senior Software Engineer on a team working on data ingest. It's a super exciting time to be joining Tackle, as they just [secured Series C funding](https://techcrunch.com/2021/12/21/newly-minted-unicorn-tackle-plans-to-use-100-million-series-c-to-accelerate-the-future-of-software-sales/).

I didn't leave Uplight on the best of terms, which is potentially a story for another time. Suffice to say, everyone there was beyond helpful and supportive, and I feel terrible about the way I left things. I'm so grateful that they took a chance on me in the first place, as a junior developer fresh out of code school, as well as the opportunity to advance from a junior engineer to a senior engineer and tech lead of a small team. I learned just an unbelievable amount there. There are some amazing folks working there, their mission is wonderful, and I truly wish the company and employees all the best.

It's been a long time since I started a new job, and the transition from being "the person who can dive into any legacy project and fix it" to being "the person who has absolutely no idea what you're talking about" has been a little rougher than I anticipated. Needless to say, impostor syndrome is back with a vengeance as well. Not that it ever went away, but I was comfortable enough to keep it at bay most of the time. That said, I'm trying to embrace having fresh eyes, and questioning things that folks take for granted.

Tackle is growing at an astronomical rate, going from ~50 employees to ~150 just this year, and planning to double again in 2022. That said, it still feels like a very early stage startup. There's a lot of work to do, which is not at all a jab at the folks who built it. They built what they needed to, and there's no sense in building software that will scale to millions of requests per hour if you're only dealing with hundreds, especially when there are so many moving parts involved. Patterns and architecture ought to evolve as the company does, so time spent optimizing some service that very well might not even be in use in 2 years doesn't seem like time well spent. All that said, there are a few key areas that I've been thinking about that will need to be improved soon.

## Tightening up loose access control
New engineers have access to basically everything on day 1, violating the principle of least privilege. Creating and enforcing access controls is a bit outside of my wheelhouse, but it'll probably be a pretty big cultural shift for the engineering team, so getting started early and rolling it out incrementally seems wise.

## Better transactional database monitoring
The company has few enough data so far that a single RDS Postgres instance is pretty reliably serving the as the data ingest destination and the API source for everything. This will be unsustainable longer term, and I haven't seen any sort of health monitoring around it. One red flag is that the maximum used transaction IDs haven't dropped for as far back as CloudWatch metrics go. It's nowhere near transaction ID wraparound, but that would be catastrophic, so at the very least, getting monitoring and alerting up is important. Longer term, the DB probably needs to be able to scale horizontally, or we should better utilize the existing data lake.

## Data ingest observability
This is a big, exciting topic, and one that I hope to write a lot more about in the future. As a baseline, [Barr Moses](https://twitter.com/BM_DataDowntime) wrote [a great introduction](https://www.montecarlodata.com/what-is-data-observability/) to the topic. The starting point here is freshness. Currently, ingests are all scheduled via cron, and each client's raw data endpoint is attempted regardless of whether or not we actually expect data to be present. This makes it really hard to know if there was some failure to fetch some client's data, or if no data was actually expected in the first place. There are too many clients to manually set an expected ingest cadence, so freshness metrics will have to rely on either rudimentary statistics from the DB, or machine learning. The goal is to get away from the current state of being surprised that some client's data hasn't been ingested in months, and instead have some easy to understand metrics that can be exposed to customer success folks. There's a lot more to say about this, but I'll save that for later.

## Signal to noise ratio of alerts
This is a common issue, but one that needs to be addressed at some point. I've been at this job for a week, and I'm already getting alert fatigue and ignoring alerts. At some point, useless alerts need to be destroyed, and important ones need to be elevated. This kind of thing will have to be a team effort involving subject matter experts from the source of each alert.

Overall, these are fun and interesting problems to have. I don't think any of them are unusual for such a young startup, and I'm super excited to start getting my hands dirty. I'm going to try to keep this updated and write about some of these topics more in depth as they come up. Hopefully someone, somewhere will find it helpful!