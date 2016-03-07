---
layout: post
title:  "Places got async expiration"
subtitle: ""
date:   2012-02-13 18:49:00
categories: [mozilla]
---

Last week, on Friday, i've pushed the last pieces of the new Places Expiration component. This was one of the Firefox Projects intended for 1.9.3 branch, you can find more background on the start of this project in its wiki page.

Some background
---------------

Originally expiration was managed by History component itself on three major steps: after each visit, during idle, at shutdown. This had various drawbacks. First of all it was making navigation experience laggish, so we moved the after each visit step to be after each sync between memory and disk tables. We also reduced idle expiration and shutdown expiration.

The result was better, but we had other issues: we were not expiring enough pages related to the number of visits, and the sync component was now bloated with non-related functionality (And slower). We were also still doing a bunch of stuff at shutdown.

In bug 516940 i cleaned up the shutdown stuff, while increasing separation between History and Expiration, at that point was easier to split it out of History in a separate component.

So, what's new?
---------------

The new component is a JS component, it runs expiration in steps, every 3 minutes, with a simple adaptive algorithm, so that if the last step did not expire enough, the next one will be run later, while if it finds more items than the expired ones, the next step will expire more! This should ensure we don't lag behind with expiration.

It also uses async Storage API, this ensures that we run I/O in a separate thread, so we won't hurt your navigation.

Expiration on idle will run just a single larger step, then it'll stop till you exit idle, this way it won't kill your standby or batteries. Expiration on shutdown runs a larger step, but not too large, in most cases the adapative expiration steps should still ensure we don't expire on shutdown.

What has changed for you?
-------------------------

The new component is able to detect your hardware specs, especially memory size, and adapt expiration to it, this means you don't need anymore to tweak number of days of history, or whatever. For this reason we have removed the number of days field from the preferences panel, you don't need anymore to tell us how much days of history your computer can handle.

What about privacy? Well, we discussed about that obviously and we went to a conclusion that the days field was not giving back any real privacy gain. Sure i could have set it to 6 days, but that would have not protected me since:

* The days pref was an "at least" pref, so for most users it was really a fake-change
* Being expiration async per definition, you can't be sure when pages are phisically expired
* even if you reduce history to 6 days, nobody can ensure you don't have bad entries in these days

Since we have better privacy tools (And we can even build new ones, so feel free to suggest changes and file enh bugs about that) like Clear Recent History, Private Browsing and Forget about this page/site, the choice was pretty clear, we want real privacy, not fake-privacy.

Also hidden expiration preferences have gone, so browser.history_expire_days, browser.history_expire_days_min, browser.history_expire_sites are now replaced by a single places.history.enabled preference. No more need to read preferences manuals just to make the browser feel faster.

What can you tweak? Ideally you don't need to tweak anything, and i suggest you don't touch any pref. Btw, for the sake of information we have two new hidden preferences: places.history.expiration.interval_seconds is number of seconds between each expiration step, while places.history.expiration.max_pages is maximum number of pages that we will retain before expiring. We make our best to have satisfying default values for anyone, current values are built to be pessimistic, we will evaluate how we behave with them, and eventually increase them in future, if we feel that's needed.

What you got finally
--------------------

Faster, easier and more secure expiration with smoother navigation. All of this at the price of fake-privacy. Win win.
