---
layout: post
title:  "New download experience post mortem"
subtitle: "The new download experience was released along with Firefox 20, just one month ago."
date:   2013-04-30 7:30:00
categories: [mozilla]
---

While the rapid release trains provide a very good userbase for testing a feature, it's only at the release time that the "real world" feedback allows to prioritize improvements.
Unfortunately, at that time, there's the concrete risk of a disconnection:

* the user sends valuable feedback
* the development team is already busy on something else

Thanks to the open and transparent nature of the Mozilla project, the feedback is never lost, very often it naturally converts to bugs and discussions. On the other side this disconnection may cause a delay before actionable items are created.

Thus I decided, after the release, to collect the most common feedback about the new download experience and setup a meeting with people involved in its development, to work toghether towards a plan.
I went through bugs, input.mozilla.org, comments to IT articles, Mozilla newsgroups, and forum discussions, extracted the most common topics and put them on the discussion table.

I'd like to point out this is not an exotic behavior at all, persons involved in development do read feedback and are active part of the community, there is also a User Research dedicated team. Though, it is worth writing this tale to clarify that what is sometimes confused with "indifference", is instead the unavoidable challenge of satisfying a large, heterogenous and passionate community.

What came out of that is a very concrete list of things we can do. Please forgive me if I don't go too much into details for each item, I'm still in the process of adding required information to bugs. Also notice many more suggestions were evaluated, this is a filtered list.

Priority improvements:

* Notify ongoing downloads when closing the browser (bug 851774) [code]
* Increase number of downloads in the panel (bug 780837) [ux]
* Reintroduce speed for each download (bug 812894) [ux]
* Make multi-selection commands work properly (bug 844606) [code]
* Properly handle removed files in the UI (bug 726451) [code]
* Add back referrer support  (bug 829201) [plan]
* Undetermined progress indicator when only unknown size downloads are in-progress [code]

Backend improvements:

* improve .part files management [plan]
* Firefox thinks failed downloads are instead complete (bug 237623) [plan]
* Remove download compelte toast notification (bug 588314) [code]

Minor changes:

* Drag download URI, not target URI, as text (bug 832994) [code]
* Avoid animations when the progress is visible (bug 857801, bug 856935) [code]
* Ensure subpixel antialiasing on text (bug 857471) [code]
* Allow multi-selection open (bug 845861) [code]
* Fix wrong icons (bug 857584) [code]
* Improve tooltip of the download button with a summary [code]
* Evaluate adding more command buttons to the Library view (bug 862528) [ux]

Now, to the bad news: we are low on resources.  Clearly everyone has many projects to follow, we'll try to dedicate available time to these changes, but we'd like having some help from the community.
So, if you are able in javascript, xul and css, your help will be very welcome.  I've added a status near each item, indicating the current needs, those having [code] are already well defined and can be worked on, the others will be defined in the next days.
