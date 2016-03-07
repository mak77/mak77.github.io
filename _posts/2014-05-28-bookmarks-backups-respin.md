---
layout: post
title:  "Bookmarks backups respin"
subtitle: "Part of the performance improvements we planned for Places, the history, bookmarking and tagging subsystem of Firefox, involved changes to the way we generate bookmarks backups."
date:   2014-05-28 12:11:00
categories: [mozilla]
---

As you may know, Firefox stores almost everyday a backup of your bookmarks into the profile folder, as a JSON file. The process, so far, had various issues:

* it was completely synchronous, I/O on main-thread is evil
* it was also doing a lot more I/O than needed
* it was using expensive live-updating Places results, instead of a static snapshot
* backups take up quite some space in the profile folder
* We were creating useless duplicate backups
* We were slowing down Firefox shutdown
* the code was old and ugly

The first step was to reorganize the code and APIs, Raymond Lee and Andres Hernandez took care of most of this part. Most of the existing code was converted to use the new async tools (like Task, Promises and Sqlite.jsm) and old synchronous APIs were deprecated with loud console warnings.

The second step was dedicated to some user-facing improvements. We must ensure the user can revert to the best possible status, but it was basically impossibile to distinguish backups created before a corruption from the ones created after it, so we added the size and bookmarks count to each backup. They are now visible in the Library / Restore menu.

Once we had a complete async API exposed and all of the internal consumers were converted to it, we could start rewriting the internals. We replaced the expensive Places result with a direct async SQL query reading the whole bookmarks tree at once. Raymond started working on this, I completed and landed it and, a few days ago, Asaf Romano further improved this new API. It is much faster than before (initial measurements have shown a 10x improvement) and it's also off the main-thread.

Along the process I also removed the backups-on-shutdown code, in favor of an idle-only behavior. Before this change we were trying to backup on an idle of 15 minutes, if we could not find a long enough interval we were enforcing a backup on shutdown. This means, in some cases, we were delaying the browser shutdown by seconds. Currently we look for an 8 minutes idle, if after 3 days we could not find a long enough interval, we cut the idle interval to 4 minutes (note: we started from a larger time and just recently tweaked it based on telemetry data).

At that point we had an async backups system, a little bit more user-friendly and doing less I/O. But we still had some issues to resolve.

First, we added an md5 hash to each backup, representing its contents, so we can avoid replacing a still valid backup, thus providing more meaningful backups to the user and reducing I/O considerably.

Then the only remaining piece of work was to reduce the footprint of backups in the profile folder, both for space and I/O reasons. Luckily we have an awesome community! Althaf Hameez, a community member, volunteered to help us completing this work. [Bug 818587](https://bugzilla.mozilla.org/show_bug.cgi?id=818587) landed recently, providing lz4-like compression to the backups: automatic backups are compressed and have .jsonlz4 extension, while manual backups are still plain-text, to allow sharing them easily with third party software or previous versions.

Finally, I want to note that we are now re-using most of the changed code also for bookmarks.html files. While these are no more our main exporting format, we still support them for default bookmarks import and bookmarks exchange with third party services or other browsers. So we obtained the same nice perf improvements for them.

Apart from some minor regressions, that are currently being worked on by Althaf himself, who kindly accepted to help us further, we are at the end of this long trip. If you want to read some more gory details about the path that brought us here, you can sneak into the [dependency tree of bug 818399](https://bugzilla.mozilla.org/showdependencytree.cgi?id=818399&hide_resolved=0). If you find any bugs related to bookmark backups, please [file a bug in Toolkit / Places](https://bugzilla.mozilla.org/enter_bug.cgi?product=Toolkit&component=Places).
