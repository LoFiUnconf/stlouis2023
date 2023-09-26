This session was the combination of two topics:

- Incremental view maintenance (IVM) which Aaron wanted to hear more about, and
- "cheating" on clients with tricks like interpolation for things like cursor position, which ? of TLDraw wanted to discuss.

A few attendees who spoke several times

- Michael Arntzenius ([rntz](http://www.rntz.net)) of Datafun
- Steve Ruiz of [TLDraw](https://github.com/tldraw/tldraw)
- Aaron Boodman of [Replicache](https://replicache.dev/)
- Predrag Gruevski of Trustfall, who asks some questions
- Paul Butler of Drifting in Space, Jamsocket
- ? who does browser engineering
- ? also from TLDraw
- ? who has implemented a system similar to TLDraw
- a dozen others including scribe [Tom Ballinger](https://ballingt.com/)

I didn't get a great transcript so I figured I might as well reorder some of the points made; I've attempted to attribute things to who said them but have missed a lot of points and a lot of people.

---

Aaron: In order not to drop frames (i.e. never go beyond 16ms when updating state to achieve 60FPS) for in-browser queries not to drop frames, I think we need incremental view maintenance. Tell me about it!

(some discussion of use cases)

Predrag: are there other contexts in which people you don't care about milliseconds?

someone: There are other contexts, like what git does, but you still care about performance a lot.

Someone: One thing I'd caution abut not worrying about performance, is that it really does! Even for text editing where you'd think it doesn't matter!~

Aaron: 60FPS really does matter! I can feel it. Dropped frames feel bad.

Git is an example where it seems not to matter. Git uses incremental view maintenance and it's faster, and it really does matter!

(Most attendees seem interested in incremental view maintenance in the context of clients, UI, and the browser. It helps that the other topic was cheating to make collaborative interactions on the client feel better)

Michael: There are two kinds of IVM:

- tracking dependencies, so that you know when you don't need to recalculate something because nothing it depends on has changed, and
- pushing diffs, for example a sum being maintained by just adding the new number

Michael: to clarify, which ones does TLDraw do? Oh both, that's cool!

(ed: [Convex](https://www.convex.dev/), the dependency-tracking database that I work on, mostly does (1) so this is very cool)

Someone: How does this work, do you have to compile the queries ahead of time?

No! TLDraw implements operators which each can maintain views incrementally. Developers compose these. This way there's no ahead-of-time compilation of these queries

(ed: this is interestingly different than [Noria](https://www.usenix.org/conference/osdi18/presentation/gjengset), which involves compiling queries.)

> _Editorial Sidebar: What does Incremental View Maintenance have to do with materialized views?_

> A database "view" is typically re-evalutated each time its queried (maybe it's cached?) and at query time is always up to date.

> A "materialized view" means it's definitely cached. Updating it might even be in the write path,
> although often it is not and instead needs to be manually refreshed or is updated on a schedule, so is not consistent with other tables
> it is based on.

> It's not explicit in today's discussion that this consistency is required, but I inferred it is.

Hey browser engineer, how do browsers do this?

? first cautions that browsers are limited by maintaining backward compatibility, then describes cool update and render optimizations that I don't quite catch.
Sounds like CSS is super complicated to implement though.
Chromium and Gecko have re-architected rendering trees in the last few years to do... something cool.

?: What are the latency classes you're interested in? 1-2ms, 8ms, 20ms, 1000ms, minutes, etc.

Those have gone to a place where the intermediate pieces of data are not mutable - there were immutable

Back to implementation, say you're creating a general purpose library for working with data structures that does make it easy to incrementally update those data structures.
A nice property is if the system declares both the pure purse derivation and the incremental version you can use property-based testing to compare them.

Aaron: when you came up with some operators that work incrementally, have you reached a point where you don't need new operators?

Steve: Yeah! We haven't written new ones for months.

Someone? do you do

Aaron: I meant "views" as in database views, I'm realizing that some folks are thinking of rendered views in a browser.

(Oh! Lots of folks were confused about this. Most discussion has been relevant to both topics.)

Do you do this in replicache?

Aaron: No not really, it's hard when we're writing a framework, not an application.

Paper: Frank McSherry - [DBSP: Automatic Incremental View Maintenance for Rich Query Languages](https://arxiv.org/abs/2203.16684)

The approach they end up with is "write a datalog!" and express things at datalog queries.
An incremental datalog engine does track these dependencies.

> Editorial Sidebar: What's Datalog?

> It's basically prettier SQL with recursion and better syntax.
> It's a logic programming language that academics tend to use because the semantics are nicer than SQL,
> but research into how to do incremental view maintenance in Datalog probably applies to SQL too.

There's this PostgreSQL thing from Japan that's like ~70% done? it's years away from PostgreSQL core though.

Now moving on to

# "cheating" on clients with tricks like interpolation

What do you do in TLDraw?
We do optimistic updates, we used to let two people to drag the same thing

Humans can implement locks if you communicate intention

We used to cheat and hide it, but now we don't (because it doesn't happen that much)

We don't do totally offline first. We used to throttle diffs

We queue things up.

(replicache doesn't do this, it sends all the intermediate steps)

Unedited notes for part 2:

Reflect: we send operations that are

Aaron: developers just know which operations can be debounced!

120 frames per second

We also batch, that's really common

What's the API that replicache provides?

TLDraw folks: you want both right, you want to send all the up and you want to know
Do you have a sequence of cursor moves intertwined

Stream of moves
These streams of operations are separate streams!

Huh, so I don't get to rely on mutations being transactional with each other!

TLDraw is trying to mark thing as ephemeral in order to get better, more meaningful undo/redo checkpoints.

Cursors really are a different data stream, no undo

The stuff you need to sync is often the undoable stuff.
