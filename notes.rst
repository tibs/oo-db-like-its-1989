=======================================================
Let's design an object-oriented database like it's 1989
=======================================================

.. contents::

Memories and made up stuff - the design (maybe) of an object oriented database
for use by a Geographic Information System (GIS).

Some stuff I remember, some stuff I confabulated, some stuff I just made up.

https://www.tonyibbs.co.uk/lsl-timeline.html#gis-development-1988-onwards
and thereafter

  "Gothic" - satisfyingly solid with nice twiddly bits

  When we came up with this as an internal development name, we were promised
  it would never escape and be used in production. Oh well.

  Amusingly, when I was working in Glasgow University (still mostly for the
  same company and on this GIS), I worked in offices inside Victorian
  neo-Gothic buildings.

I haven't had sight of the codebase I'm (vaguely) remembering since 2003, so I
don't think I can possibly give away anything useful!

(I mean, I'd love to be able to see the code for the bi-directional references
again, but so it goes).


Vector versus raster GIS
========================

Obviously vector is the correct choice...


Objects
=======

Objects have:

* an id

  for "reasons", we'll make the id an object id (a single value identifying
  this object) and a "version" (which will be an integer field that can go up
  to a sufficiently large number). We'll ignore the version field for a while.
  (but to my memory, we did introduce the space for it early on - this will
  have been from experience with other things, I think?)

* a class
* zero or more values
* zero or more methods

Methods can "see" the values of the object.

Single user
===========

Given the application space, it wasn't obvious that we *needed* multiple user
access to the database. And it simplifies things an enormous amount.

Of course, we now know how successful sqlite has been over the years.


Object storage and paging
=========================

Objects stored on disk.

Keep "currently in use" objects in memory, page them out to disk if necessary.

  Tell the story of the scarily good IBM GIS that was written in Smalltalk so
  was limited in the size of map it could represent by available memory (in
  the 1990s!) and how that meant we won at least one contract.

  And note that paging seemed so "obvious" to me that I never considered not
  doing it. Of course, our test case *was* Sheffield 101 (is that the right
  sheet number) which is pretty densse.

  (I'm assuming the Smalltalk GIS could have done this as well, they just
  didn't think "but what if the map was **bigger**",)

An index to find them
=====================

We'll need an index so we can take an object id and find its location on disk
(that is, in the database file or files - I'm going to assume a single file,
but I can't remember if we ever split into multiple files for a single
database).

I believe we used a B-tree at first - I don't remember if this got changed.

We *did* do some very large scale tests to show that the database worked with
very large sizes - the sort of sizes that could only be done by "stringing
together" lots of physicial disks (at the time) into a single virtual disk. My
memory wants to say "petabyte scale", but that feels unlikely even for the
early 2000s?

Remember a version number
=========================

After working on a few transfer formats and some file formats, you begin to
learn to always put a version number at the start of the file, specifying the
version of the file format / layout.

Or so you'd think.

Because if you don't, then when the inevitable version 2 comes along (normally
much earlier than you expect) your code has to instrospec the file to see if
it's version 1 or not, by looking for the *absence* of a version number
(essentially).

Hopefully this happens before the first relase to the world.

Also, in the same line, it's nice to put a format identifier at the start.
This will make ``file`` happier (if you're on a Unix), and also generally
makes the file or database more identifiable as what it is, without needing to
rely on having a particular file extension or (again) introspecting the data.

(Perhaps reference KBUS here?)

Schemas and inheritance
=======================

Hmm.

Collections
===========

This was before languages like Python. So what sorts of collections to support
was not obvious.

I did some research (including reading up on Smalltalk quite a bit).

I think we ended up with:

* a list or array type - I can't rememember the details of this, but I think
  it was a linked list, and I think we definitely wanted to be able to operate
  on each end equally
* Sets
* Bags
* Maps (dictionaries/hashes)

For references, we wanted to be able to do the standard arrangements:
one-to-one and one-to-many

Why GIS is odd
==============

Two dimensional. But may represent (significant portions of) a sphere.

  **Note** find someone very good to write your coordinate handling libraries.
  They may end up inventing very rigorous testing mechanisms as well.

Topology plus topography. They're linked, somehow. But if you just use the
topology (no actual coordinates) then you've got a graph database, which is cool.

A line may have many many coordinates defining it (think fjords, or the whole
of the UK coastline).

An object may have zero or a lot of values (at least up to a hundred, and
maybe more)

Representation is a thing - we draw maps. And they look different at different
scales. (Methods can and will help with that, as objects can "talk" to each
other and negotiate representation.)

Slivers. And *this* is why vector is better than raster (<smile>).

Contours. Hmm.

  At the time, RDBs didn't handle values which might vary hugely in size
  (coordinates) or rows that might have large numbers of columns that were
  only occasionally used. We're thinking Ingres and Oracle (Ingres was
  generally our preferred choice, because it was *much* easier to install and
  manage, and I think also cheaper?)

  Check: Ingres was the ancestor of PostgreSQL? Read
  https://en.wikipedia.org/wiki/Ingres_(database) ...

  Of course, the received wisdom at the time was that a database needed a
  department to manage it, so it was quite hard to convince companies to take
  on another, weird, sort of database, which would presumably need even mmore
  management (even though it didn't, in fact).

  Also, it was perceived as making a data silo in this weird other database,
  even though we did provide means of communication (which I remmeber nothing
  about, unfortunately).

  Nowadays, such concerns seem a bit quaint, with so many different database
  and database adjacent technologies.

Why GIS?
========

Mapping - actual representation of maps

But also map analysis, and *in particular* the first was to be for route
planning.

And that includes things like "if its raining, what weight of vehicle can
safely traverse this field, given its characteristics" - hence methods are
needed, because that's an answer whose calculation needs external factors to
be takeb into account (arguments).

Topology: nodes, edges, faces
=============================

Explain what they are!

References
==========

How we link objects together. Makes everything work.

Bi-directional

* explain *why*
* explain how that was an unpopular decision in the industry
* explain how I was right all along (mwah hah hah hah)

I was intensely validated when I looked, some years later, at the Neo4J
documentation and saw that they also regarded bi-directional referneces as "obvious".

Topology and directionality
===========================

* node knows the edges attached to it, in a predictable order (clockwise?), so
  one can go "next, next" at them
* edge knows its start and end node, and its left and right face
* given that, we can deduce an order for the edges surrounding a face

Where does the geometry live?
=============================

* On the nodes and edges. Minimalistic. (Edges need shape, so they need to
  have "internal" coordinates). We don't need coordinates on faces, but
  perhaps *might* do so for some reason (efficiency of some sort?), or might
  make it so the face can directly reference the coordinates on the edges
  (sort of like compilers will share string fragments).

  I will argue that an edge has *all* its coordinates, even though the start
  and end coordinates match the corresponding nodes. But if so, care must be
  taken to maintain that.

  Nodes do need their own coordinates, because they might not be associated
  with any edges.

* Also on the points and lines. Clearly not *needed*, but might be useful for
  efficiency? Again, if doing this, consider if the coordinates can be shared.

I honestly can't remember if we stored coordinates anywhere other than on the
nodes and edges.

Other coordinates
=================

Methods can be used to do things like calculate representation - more
coordinates. And  caching them may make sense. So that's another sort of value
that stores coordinates.

Also, if one is calculatign representation, "imaginary" objects may be
calculated - for instance when buildings amalgamate to make a built-up area.
So perhaps those "imaginary" objects will want caching (somewhere - it's not
obvious where as the result is shared between multiple "real" objects). Again,
I don't remember how any of this was actually done.

Geographic data: points, lines, areas
=====================================

An area can be made of many faces. They can be disjoint.

A line can be made of many edges. Sometimes they can be disjoint, too (well,
at least maybe).

So there's a decision to be made - can a point be made of many nodes?

An index to find (spatial) things
=================================

Once you've got things with coordinates, you want to be able to search for
them in space.

The natural way is then to compute the bounding box for an entity, and index
that, rather than trying to be too clever.

The initial spatial indexing used quadtrees (I thought this was fascinating at
the time), although I believe it quite soon moved on to something more
sophisticated, which I know I didn't understand at the time.

I'm going to assume that the technology has probably become mature in this
area (no, that's not meant to be a pun), but it's not something I've ever
followed.

  (The nice thing about quadtrees is that they don't get deep very fast. The
  nearest I've ever done to an academic paper was a note in a journal that
  showed that representing all the vector data in the OS(GB) sheet 101
  (Sheffield) only need a quadtree of a smallish depth, something around 4 or
  thereabouts. Which was regarded as counterintuitive at the time, I think.)

Objects in memory
=================

We were writing in C (there wasn't an obvious other choice - I had done
research on this!) so we reference counted the objects in memory (you need to
free them when they're no longer needed). With macros to try and help - but
reference counting in C is never friendly, as it's easy to forget who is
responsible for managing reference increment/decrement.

If we'd been able to use Python, Ruby or Java (all not invented when we
started) then we could have used their mechanisms.

Emacs and LaTeX, oh my
======================

Around the time we started using C I also started using Emacs (well, XEmacs -
give a reference) and TeX.

So naturally I designed a standard header comment format for our C functions,
and wrote an Emacs macro to automatically create LaTeX API documentation.

Which was unnecessarily cruel, as it locked our developers into using Emacs as
well. This wasn't so uncommon for companies to do back then (determine what
editor their programmers would use), but it's not common/recommended
now-a-days for good reason. It would have been much better if I'd written the
documnentation extraction tool in something else (or ported it to some other
programming language when our team expanded).

On the other hand, when we came to want to write Java interfaces to our GIS
library (Python would have been a better fit, but none of our customers had
heard of Python), one of my colleagues was able (in pure brilliance - I
thought it would be impossble) to autocreate JNI bindings for more than 90% of
our C functions, based on those same C function header comments.


Other things...
===============

...


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

So - do references have attributes?
===================================

**Big** argument

I always thought "no", with the *possible* exception (practicality over
purity) of *direction*.

Consider a postal route (a line object that refers to other line or edge
objects). A postal route may go one way along a street, and then come
back the other way.

You can always simulate references with attributes by adding in extra objects.
But that might be regarded as clumsy.

The argument over this could get quite heated...
