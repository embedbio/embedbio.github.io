---
layout: post
title: "PTA grant agent"
date: 2026-10-29
author: "Amit Indap"
tags:
  - LangGraph
  - non-profit
  - AI
  - PTA
---

I was proud to have secured $4000 in grants last year as PTA president. The ROI was huge, because the applications were pretty straightforward and generally took <1hr to complete compared to doing bake sales or other low margin fundraisers.

The biggest hurdle was finding funding opportunities. I would usually do Google  searches to find leads. Fast forward a year later, I (with help from Claude) wrote an AI agent to do the grunt work for me:

https://github.com/indapa/pta-grants-agent

All you need is a  Gemini and Tavily API key. I haven't tried  it with a locally run model (e.g. Ollama), but will add that feature soon.

I recently ran the program and found several  promising leads after the agent ran for 9 minutes. If I did the same manually, it probably would taken me at least 45min-60min to compile the same list. 