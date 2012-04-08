---
layout: post
title: "Statistics @ Velocity"
date: 2011-06-11 21:12
comments: true
categories:
- velocityconf
- statistics
---

One of the pre-conference workshops was an excellent session by John Rauser from
Amazon. I've never been hugely confident at my abilities to do estimation,
forecasting, capacity planning etc. It's a really difficult thing.

I'd heard that Rauser was an excellent speaker, so I attended his workshop on
statistics. For anyone interested, Rauser also did a shorter keynote later in
the conference, which you can catch on YouTube. A big theme from these
discussions was understanding the reduction that your statistical reporting
makes. Averages, variances, etc are all summary statistics - but how often to we
really think of them as lossy formats?

  

  My main takeaways from both of his talk were to try and understand the
  lossiness of summary stats. I particularly liked his summary statistics of
  Moby Dick. Another major theme in his talks was to make time to do deep dives
  into log data, even if it only means you know how to when something really
  goes wrong.

  

  During the workshop, John discussed how we approach estimating someone's age.
  Given that problem, the instinct reaction is to propose a estimated value: 45,
  58, 60 etc. However, to quote Rauser: we '[shouldn't] give a single number
  unless you're forced to. We should always give a range, ideally with an agreed
  confidence interval such as 90%. I can guess someone's age with 100%
  confidence if I state their age is between 0 and 150, but that's not very
  useful. But by combining additional information and reducing absolute
  certainty I can start to narrow that down. This isn't rocket science, but how
  many times have we been asked "when will this be ready", or "when's the
  delivery date"? And in the face of those questions, how often have we given a
  range with a stated confidence. Perhaps we feel that our project managers
  would laugh us out of the room? Project managers like fixed dates.

However, in complex projects - and IT projects are usually complex - a range is
often more honest, and would often better express the reality of the situation.
Again, this is nothing new - if you have an Agile Coach on your team, they will
probably recommend estimating using ranges or 3-point estimates. However, it's
easy to slip into the default behaviour of giving a single numeric response.

  Throughout the talk, Rauser covered other examples, such as estimating the
  depth of his desk, and discussed how measurement improves our ability to
  estimate with confidence.

  

  As the workshop continued, Rauser set us an estimating task. He asked us to
  provide estimates (ranges, of course!) for a number of questions. The key was
  not the specific answers, but rather that we were aiming for 90% confidence
  estimates... i.e we should have got 18 out of the 20 questions correct. We
  then graphed the results of a portion of the room and found that most people
  averaged between 20% and 40% accuracy. An interesting demo of how humans seem
  to default to be over-optimistic in their estimating.

  

  In the final section of the workshop, Rauser discussed the nature of
  statistics - how school curriculum are still teaching stats the 'old way' and
  how for practical analysis, it's really much better to tackle stats correctly
  using computer modelling. Do we need to learn about how to apply statistical
  tests and derive summary statistics? After all, summary stats are lossy and
  stop you looking at your data? Are they more dangerous in the hands of the
  layman that just getting a computer to process the raw stats? We saw a demo of
  some of the statistical features of excel, and then, a demo of more powerful
  tools like **R** and **Sage**.

  

  In conclusion: Rauser left us with a few final comments:

  * everything is uncertain
  * even counting breaks down at scale
  * estimates need ranges
  * single numbers are decisions

A great session.