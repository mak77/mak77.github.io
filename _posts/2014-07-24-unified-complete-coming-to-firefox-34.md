---
layout: post
title:  "Unified Complete coming to Firefox 34"
subtitle: "The new autocomplete implementation is almost ready"
date:   2014-07-24 11:23:00
categories: [mozilla]
---

The awesomebar in Firefox Desktop has been so far driven by two autocomplete searches implemented by the Places component:

1. history: managing switch-to-tab, adaptive and browsing history, bookmarks, keywords and tags
2. urlinline: managing autoFill results

Moving on, we plan to improve the awesomebar contents making them even more awesome and personal, but the current architecture complicates things.

Some of the possible improvements suggested include:

* Better identify searches among the results
* Allow the user to easily find favorite search engine
* Always show the action performed by Enter/Go
* Separate searches from history
* Improve the styling to make each part more distinguishable

When working on these changes we don't want to spend time fighting with outdated architecture choices:

* Having concurrent searches makes hard to insert new entries and ensure the order is appropriate
* Having the first popup entry disagree with the autoFill entry makes hard to guess the expected action
* There's quite some duplicate code and logic
* All of this code predates nice stuff like Sqlite.jsm, Task.jsm, Preferences.js
* The existing code is scary to work with and sometimes obscure

Due to these reasons, we decided to merge the existing components into a single new component called UnifiedComplete (toolkit/components/places/UnifiedComplete.js), that will take care of both autoFill and popup results. While the component has been rewritten from scratch, we were able to re-use most of the old logic that was well tested and appreciated. We were also able to retain all of the unit tests, that have been also rewritten, making them use a single harness (you can find them in toolkit/components/places/tests/unifiedcomplete/).

So, the actual question is: which differences should I expect from this change?

* The autoFill result will now always cope with the first popup entry. Though note the behavior didn't change, we will still autoFill up to the first '/'. This means a new popup entry is inserted as the top match.
* All initialization is now asynchronous, so the UI should not lag anymore at the first awesomebar search
* The searches are serialized differently, responsiveness timing may differ, usually improve
* Installed search engines will be suggested along with other matches to improve their discoverability

The component is currently disabled, but I will shortly push a patch to flip the pref that enables it. The preference to control whether to use new or old components is browser.urlbar.unifiedcomplete, you can already set it to true into your current Nightly build to enable it.

This also means old components will shortly be deprecated and won't be maintained anymore. That won't happen until we are completely satisfied with the new component, but you should start looking at the new one if you use autocomplete in your project. Regardless we'll add console warnings at least 2 versions before complete removal.

If you notice anything wrong with the new awesomebar behavior please [file a bug in Toolkit/Places](https://bugzilla.mozilla.org/enter_bug.cgi?product=Toolkit&component=Places) and make it block [Bug UnifiedComplete](https://bugzilla.mozilla.org/show_bug.cgi?id=995091) so we are notified of it and can improve the handling before we reach the first Release.
