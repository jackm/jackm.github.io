---
layout: post
title:  "Hide page elements on Tangerine online banking using uBlock"
date:   2018-04-02 22:12:46
keywords:
  - uBlock
  - Tangerine
  - bank
  - online
  - block
  - elements
  - insights
  - highlights
---

[Tangerine][tangerine-bank]'s client online banking interface is pretty obnoxious with some questionable UX choices.

Most of the page elements, fonts and buttons are excessively large to the point where hardly any useful content will fit on a single screen.
And this is coming from someone who uses 2560x1440 display resolution.
You have to scroll down just to get past the giant header banners and unhelpful pie charts that appear before 

I guess I kind of get what they were trying to do - have one UX design that will work for both desktop users and mobile users.
More often that not however, I find that trying to shoehorn in a one-design-fits-all strategy never really works out that great for anyone.
Desktop users don't quite like it, and neither do mobile users.

Anyways, below are some [uBlock][ublock] filters that will help improve on Tangerine's UX design.

All of the filters are there to either block user analytics tracking or to hide elements from the page that take up way too much screen space.
With these filters, you can see the stuff you care about easier, like transaction entries and not waste your time scrolling down past the huge banners and useless charts.

Just copy and paste these lines into the text box in "My filters" tab within uBlock's settings.

```
! Block Dynatracemonitor APM user tracking software
||tangerine.ca/*/dynaTraceMonitor^

! Hide the giant "insights" banner, bottom "highlights" section, overly large account balance font and the useless pie chart
tangerine.ca##.is-open.c-insights-carousel
tangerine.ca##.c-quick-access
tangerine.ca##.c-account-details__balance
tangerine.ca##.c-transactions-chart
```

[tangerine-bank]: https://www.tangerine.ca/
[ublock]: https://github.com/gorhill/uBlock