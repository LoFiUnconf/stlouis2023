# CRDT Implementation session notes

## Where session attendees are coming from:
<note: missed three people at the start>
- Very little knowledge
- Work on CRDTs for construction industry
    - How to break a large CRDT into smaller ones
- 4 years of experience, trying to wrap my head around what a CRDT is and get a visualization
- I make a hybrid text visual editor, a
- I found a ways to not do CRDT, I want to find better ways to do CRDTs
- Worked a bit with auto merge, want to make things simpler

## Initial Discussion

CRDT stands for Conflict Free (Resolution ?) Data Type
Sets <note: add only sets> are an example because you can add items into it in any order and you cannot get conflicts
In Martin’s talk at StrangeLoop he talked about CRDTs for text
- Vector Clocks
- How do you not use UUIDs
What CRDTs are we interested in? - Set? Text? Register? Tree?

Taking highest version number
- Time is relative 
- Represent time as the number of changes you’ve made in the document with your id

## Questions

### If you have concurrent updates to text, how do you know what comes first?
- Peers each have their own id
    - Something you need to do beforehand is assign globally unique peer IDs
    - Peer ids are just really randomly generated strings that are extremely likely to be unique

### How do you go from a flat thing to JSON?
- If you have an event log, and events have parent events
    - Each peer has linear events

![Photo on 2023-09-23 at 5 42 PM](https://github.com/LoFiUnconf/stlouis2023/assets/11082236/36166234-6196-4127-b9b9-1ba1f2e81d11)
Glared out part: “Now need to linearize this tree”


### How is the data sent across peers
- Everyone has their own system
    - Have to write the networking as peer to peer or as client server
    - Have to decide on a protocol for how to share changes
- It doesn’t matter if you use TCP, UDP, HTTP, sockets, etc.
- CRDTs are network agnostic

There are state-based CRDTs and Delta-based CRDTs

### Why build your own?
- If you have two conflicting fields 
- There’s a distinction between implementing your own CRDT vs composing CRDTs to make your own data structure
Performance impact?
- Cost to document size, more object ids,

### How large do documents get? How many operations can you do? - One person has seen millions of writes
- Martin Kleppman made document compression
- Automerge recommends clearing history at some point

Adding branches, in-memory compression, history deletion.

### Do branches cause problems with lampport vector clocks?
- There’s a sequence, actor, concurrency id to help with branching
    - Increment concurrency id if you make a change locally that needs a new branch
    - Switch between 4 versions of the same actor
    - This solves the vector clocks issue
- One person’s solution is to do a separate feed for every branch
- Merckle search tree if you have one document with so many 

### What is a branch?
- A specific set of change the represent a state of a document
    - In git you can merge completely different things
- Use case for branches?
    - Upwelling text editor needs branches
        - Multiple people editing the same document, each on their own branch, can see each other changes and accept them or whatnot
    - Drawing program where you can see other people drawing
        - Ephemeral branch analogous to a layer in photoshop
        - Make local changes to my branch and then choose when you want to merge
            - Then delete my local branch to hide my changes

### What is a clock? Can you do it without clocks?
- It is a logical clock
    - It is a way of representing a happens-after/before/concurrently relationship between events

If you have an append only data structure then you can’t have a CRDT if you don’t have an ordering of events



