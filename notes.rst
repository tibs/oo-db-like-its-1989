=======================================================
Let's design an object-oriented database like it's 1989
=======================================================

Memories and made up stuff - the design (maybe) of an object oriented database
for use by a Geographic Information System (GIS).

Some stuff I remember, some stuff I confabulated, some stuff I just made up.

https://www.tonyibbs.co.uk/lsl-timeline.html#gis-development-1988-onwards
and thereafter

I haven't had sight of the codebase I'm (vaguely) remembering since 2003, so I
don't think I can possibly give away anything useful!

(I mean, I'd love to be able to see the code for the bi-directional references
again, but so it goes).






Object Versions
===============

Back when we were designing our object ids, we left a big empty space to hold
a *version*. This is where we have a look at that that might be useful for.

  **Note** that I don't actually remember how versioning was done in the
  original software, so this is a made up (but perhaps plausible) way of doing
  things.

Assumption: all objects start with version number 1 (version 0 will come up in
a little while). Also, the database as a whole gains a *maximum object
version*, which stores the largest version number over all the objects
therein.

  (That *could* start at ``0`` for an empty database.)

Let's imagine that every time we finish editing a set of objects, and save
that change, we also set their version numbers to the *maximum object version*
``+ 1``, and then increment that *maximum object version*.

Furthermore, lets add an extra index, which goes from

  *object id with version set to zero*  **to**  *object id with particular
  version*

The *particular version* should be the **latest** version for that object. So
we'll also need to remember to update *that* when we finish an edit sequence.

Also, amend the original index to go from

  *object id with particular version* **to** *location of that object in the
  database*

Of course, you could equally conflate those into a single index if that feels
better to you - the type of key (version number zero or not) determines the
type of value.

..

  **Note** this would be a good time to backup the original database (you did
  write that backup and restore software earlier, didn't you? If not, now's a
  good time) and then restore it to populate the new indices.

So to find the location of an object in the database, we either:

1. Look up the *object id with zero version*, to get the latest version of
   that object id. Then look that up to find its location.

2. Look up an *object id with a particular version* to directly find a
   specific version of an object.

For references, we make sure to always store an *object id with zero version*
(hmm, could do with a shorter name for that!). To follow a reference, our
default action is thus to look up the most recent referenced object, which is
what we want.

Now, as I said, I can't remember how we actually did version handling, so the
following is an ineffcient flight of fancy - but let's go for it anyway.

A useful thing to be able to do is to say "show me the database as it was at
object version **N**". Now assume that we have lazy copy-on-write for our
*particular version* index

  **Note** Luckily we're not doing implementation, so we can just assume this.
  But of course, lazy copy-on-write datastructures are now used in various
  places, including the Limux kernel, so I assume this is just a matter of
  looking up how to do it <smile>.

and introduce yet another index,

  *object version* **to** *particular version index*

Each time we end an edit sequence, we

1. make a lazy copy of the *particular version* index (this should cost us
   almost no space, because "lazy")
2. update the object values in this new version of the index
3. add that *particular version* index to the new "index index", using the new
   *maximum object version* as its key

For a normal (current) lookup, we use the *maximum object version* number as
the key to get the relevant *particular version* index.

To go "back in time", we just choose the relevant (older) object version
number.

Other things we can do, once we've got this:

* revert changes - go back in time and throw away the later changes
* prune obsolete versions - throw away the entries in the "index index" for
  sufficiently old keys
* reduce the density of changes retained - remove some percentage of older
  "index index" keys, and merge them in with the adjacent (earlier or later?)
  entry.

**However** we will have to give a little thought to deleted objects. Clearly
deleting an object means it shouldn't show up any more, so we probably need to
keep its index entry, but marked as "deleted". That's not a big issue - it's
the same sort of soft deletion we see in a lot of more traditional database.

  If we prune or compact the database, we can decide to entirely throw away
  entries for objects that are no longer reachable.

If our database ever becomes multi-user, this will also make backup a lot
easier, as we'll be able to backup a specific version, even while edits are
happening to other, later (invisible because we're not using their indices)
versions.

(Maybe compare with how BSD-like operating systems have been able to do
backups of their disks because their file systems support this sort of
option - although I don't know when that became common.)
