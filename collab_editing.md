# hi

I'm Stephen Whitmore (`noffle` on the interwebs)

https://github.com/noffle
https://twitter.com/noffle

# goal: collaborative p2p text editing

this talk is to present some of the problems with
centralized document collaboration, and offer solutions to
the largest challenges to having asynchronous peer-to-peer
editing in realtime

---

# problems with today's centralized document editors

e.g. Google Docs, Etherpad

- dependent on a central service
  - usually fine for Google, each joe's etherpad server less so
  - ironic: both products actually owned by Google

- making backups of documents is usually non-trivial

- having poor / intermittent connectivity is not forgiving

---

# challenges

- how do peers who want to edit the same document find each
  other?

- how do you resolve edit conflicts?

---

# peer discovery

WebRTC!

however, WebRTC requires peers to exchange introduction data
to connect

solution?

---

# peer discovery

(inspired by https://github.com/moose-team/friends)

1. new user enters a document name (UUID)

2. user connects to a signalhub instance

3. signalhub acts as a middle-man to facilitate webrtc
   introductions to other peers interested in the document

4. this forms a webrtc swarm of peers editing the same
   document!

---

# resolving conflicts

- CRDTs
- causal trees
- lamport timestamps

---

# CRDTs

Enter CRDTs!

*conflict-free replicated data type*

or

*converging replicated data type*

---

# CRDTs

data structures specially designed to provide:

*strong eventual consistency*

^ can receive the same set of updates in any order

*monotonicity*

^ no rollbacks need to occur to get consistent state

---

# CRDTs

* add-only counter

* grow-only set

* remove-once set

---

# document editing with CRDTs

wait!

imagine there is a text document being edited by two Alice
and Bob

it has the text `world`

---

# document editing with CRDTs

Alice locally performs

`insert(0, 'hello ')`

but Bob locally performs

`delete(0, 2)`

---

# document editing with CRDTs

If Alice applies her entry first, the document reads:

`hello world`

then Bob's:

`llo world`

---

# document editing with CRDTs

If Bob applies his entry first, the document reads:

`rld`

then Alice's:

`hello rld`

*How can you resolve this?*

---

# causal trees

instead of indexing letters in a document by numeric index,
how about unique IDs?

---

# causal trees

instead of

 | H     | e     | y
 |       |       |
 0       1       2

have

 | H     | e     | y
 |       |       |
 ubq1    ubq2    ubq3

---

# causal trees

now operations are relative to a unique character,
regardless of its position

`insert('ubq1', 's')    // => ubq4`

and

`insert('ubq3', 'f')    // => ubq5`

resolve cleanly, applied in either order.

Deletes work the same way.

---

# causal trees

this forms a tree structure:

               root
            /   |    \
       ubq1    ubq2    ubq3
        /                  \
     ubq4                  ubq5


allowing operations to be shown in the same ordering
regardless of what order they are received in

---

# causal trees

but wait!

what about two users racing to make a child of the same
node?

---

# causal trees

e.g.

`Hell`

Alice wants to append `o` to the end.

Bob wants to append `a` to the end.

Depending on the order they are applied, the result is
either

`Hello` or `Hella`

how can this be resolved?

---

# lamport timestamps

this problem occurs whenever there are multiple siblings
that share a parent in the causal tree.

could timestamps be used here for ordering?

---

# lamport timestamps

no way! time varies from computer to computer, and two users
could publish operations with the exact same time

---

# lamport timestamps

`lamport timestamps` introduce the idea of *logical time*
rather than *wall clock time*.

time only move on a user's machine when an event is
published

---

# lamport timestamps

Alice chooses a random prefix locally, `jmb`.
Bob chooses a random prefix locally, `trk`.

Alice and Bob start their operation clocks at `jmb0` and
`trk0` respectively.

---

# lamport timestamps

Her logical time increases as she performs operations:

`insert('ubq1', 'H')     // => 'jmb0'`
`insert('jmb0', 'e')     // => 'jmb1'`
`insert('jmb1', 'y')     // => 'jmb2'`
`delete('ubq1', 1)       // => 'jmb3'`

Now if Bob does insertions that would conflict

`insert('ubq1', 'P')     // => 'trk0'`

The timestamps can be sorted lexagraphically to achieve a
deterministic ordering on events!

---

# bonus: storage and sharing

incremental revision snapshots and publishing would be
great!

use IPFS or webtorrent to publish and store snapshots to get
revision history and easy sharing of the final (immutable!)
document

https://ipfs.io
https://webtorrent.io

---

# it's over

questions?

---

# references

http://www.ds.ewi.tudelft.nl/~victor/polo.pdf
https://github.com/ritzyed/ritzy/
https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
https://en.wikipedia.org/wiki/Lamport_timestamps

