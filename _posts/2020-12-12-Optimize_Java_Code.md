---
layout: default
author: Cameron McKenzie
title: How to optimize Java code
blurb: Here's a look at how to optimize Java code and improve JVM performance
---

<div class="embed-responsive embed-responsive-16by9">
<iframe width="560" height="315" src="https://www.youtube.com/embed/f2aNWtt0QRo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>


# Java code optimization

When we talk about memory efficiency
we're kind of talking about two things
one is using the least amount of memory
possible when you're writing your
application so keeping it small but at
the same time trying to minimize the
amount of work the garbage collection
has to do so there's a cost when you
allocate objects and when garbage
collection frees them up so you want to
minimize that as well as minimizing your
your memory footprint and the to in some
ways compete with each other the best
way to avoid object allocation and
freeing is to cache stuff and sometimes
that means having a higher footprint so
it's a trade-off between the two so it's
always a decision you have to make and
hopefully this presentation gives you a
lot of information on how to make that
decision so before I carry on who has
been to an IBM presentation so far this
conference okay then you guys don't need
to read this for everybody else if
you've got a camera phone you're going
to need to take a photo sign it and send
it back to IBM to say that you've read
this and you understand that everything
I've said is probably wrong because I'm
not very reliable
so me I work for IBM I've been there for
13 years spending all that time working
in what we called the Java Technology
Center so I've been working on JDK
development on IBM's versions of Java
and I've had quite a range of job roles
they started off supporting customers
doing consultancy doing kind of bug
fixing kind of work since then I've
moved on to doing what we call
serviceability so how to do monitoring
and diagnostics I do the way that we get
deployed into the cloud I do customer
requirements and basically the external
interface to the JVM in the JDK from IBM
so a lot of the content that's in here
is things that we've gone out to
customer sites and we've worked with
customers on trying to make their
applications more efficient my contact
details at the bottom including LinkedIn
and my email address and there's a
SlideShare account at the bottom and
that's where the slides will be
hopefully by the end of today but
probably tomorrow so the session itself
we're going to start off aiming to
explain how memory gets allocated when
you write some Java code so you use the
new operator to create an object how
much memory is actually used for that
object then we're going to talk about
garbage collection and allocation and
freeing of objects and how to try and
minimize that minimizing that means
caching so we're then going to talk
about the Java collections so the ways
of storing objects and then finally
we're going to talk about an easy
mechanism to try to find areas of your
application where you've got
inefficiencies in terms of memory usage

## Optimize Java Memory Usage

so starting off memory usage Java itself
so you've got an application just a main
class that doesn't really do anything
all it does is it creates a new integer
object in which you put the int value 10
so in in 32 bits in size so on a 32-bit
platform how big do you think is the
piece of memory that Java puts on the
Java heap for that int value of 10 that
you've got in your integer so who thinks
it
32-bits the same size as the int okay
three or four people who thinks it's 48
bits a one and a half times the size of
the int value anyone a couple who thinks
it's 64 bits so twice the size now we've
got a few more and who thinks it's 96
bits so three times the size that's
that's probably about a third of you and
now there's a whole number of people who
didn't put up their hands go and admit
it so you've either seen me say this
before or you're a little bit lazy but
the answer is actually up to five I'm
cheating it's four times the size so
when you create just a simple in value
and it goes into an integer the amount
of memory Java uses to store that is
four times the size of an int so it's
quite a lot of memory just to to
allocate a little bit of data so the
obvious question is why is that so when
you put that ten value into the integer
we created an object and Java objects
have got metadata that describe your
data right so first of all I need to
have something this object that tells me
it's an integer because you created an
integer so what I've got is called a
class pointer and that points to the the
integer class so the way that I know
what type of object I've got is the
first bit of data inside it points to
the class for that type so it would
point to a java.lang string or an
integer object or or you know whatever
its type is so I've then got a second
bit of metadata which is flags
so this stores a number of bits of
useful information it has usually a
shape bit it has a hash code and so on
so there's another 32 bits worth of data
that tells me some other stuff about the
object you typically have a lock word as
well so the fact that you can
synchronize on Java objects 

<img alt="Optimize Java Code" class="img-fluid" src="/assets/eclipse-jmc-garbage-collection.png"/>


we have to
have a bit of data that lets you do that
so you've often got three words 96 bits
worth of header that's just to have a
Java object before you put your data in
it now I will point out for some of the
more technical people who worked in JVM
implementations I'm saying typical
the amount of data does change from gel
desertions and it actually changes on
different types of objects we try and do
things to to merge looks and flags where
we can typically it's around 96 bytes
worth of data just for the Java object
and then you start putting your
information into it now if what you've
got isn't a and then you have your your
value of 10 now if what you've got is an
array object so for a raise you again
have a class pointer that says it's an
array of Java an array of integer you
then have flags in the same way except
this one's gonna have a shape bit set
that tells me it's an array object I'm
going to have a lock word as before but

## Java array performance optimization


I have to have an additional field which
is the size of the array so I know that
this is either a one element array or a
ten element array or a thousand element
array and this actually is the maximum
size of an array is the maximum size of
that size number so it's you know 32
bits worth of data tells me how long my
array can be and then I have an int
value on the end so to store an int in a
integer objects as a four-to-one
overhead to put a single int into an
integer array it's a five to one
overhead so arrays a slightly larger now
if you move to 64-bit you have words of
data which are 64 bits wide rather than
32-bit so what we have previously was 96
bits of overhead and then your data on
64-bit all of those double in size so
we've got one small value of 10 again
but all of our overhead is doubled in
size and again it's the same thing for
arrays so where we previously out 128
bits of overhead that's now doubled to
256 bits of overhead which means our
overhead to store an INT in an integer
object becomes 9 to 1 so on 64-bit
everything just uses more memory.


### Java Heap Optimization

so moving from 32 to 64 bit you need to
make your Java heap bigger and you have
to accept that it's going to cost you
more memory now there is a mitigation
for this
is anybody in the room heard of
compressed oops or compressed references
okay so what this does is it says where
I have a 32-bit value on 32-bit to point
to my class on 64 bits I can actually
use a 32-bit pointer as long as the
class is at the bottom end of memory as
long as it's within a small range of
memory I can still use 32 bits to
reference it so for small heaps and
those are heaps under 32 gigabytes on
64-bit you can run with compressed dupes
or compressed references and it stops us
using those 64 bits we use 32 instead so
it looks like this whereas on 64-bit
without compressed references we use 192
bits of overhead or 256 bytes of
overhead we shrink back to 32 bits for
Java objects and for a raise so it goes
back to what you had on 32-bit so
basically what we're saying is if you're
using 64-bit Java with Java heaps under
32 gigabytes use compressed references
or compressed hoops so this is the table
of what all of those values inside an
object looks like if you store an int if
you store a boolean if you store a long
then those are the values now the
interesting bit is between 32 and 64-bit
and everything's the same
apart from objects in the bottom right
so when you move to 64 bits your object
pointers.

## Object pointers and performance optimization


 so if you've got an object with
a reference to a string inside it the
amount of memory for that reference to
the string doubles as you move to 64-bit
and again that shrinks back down to 32
bits if you use compressed references so
again use compressor efference --is okay
so that was just allocating a simple
integer object right we had our single
int inside 128 bits worth of memory and
there was a four-to-one overhead now
what happens if I create a string so
when I create a string I've got my three
words of
either had my 96 bits of Earth overhead
then inside a string object there's
actually a field in there called hash
there's another field called count
there's a field called offset so that's
basically the length of the string and
the offset into the string and then I
have a field called value and value is
actually a reference to a character
array and that character array will
store your data so if you create a new
string with my string in it you actually
get two Java objects created one that's
the string itself and the other one
that's a character array that contains
your data which is my string so if you
total up the overhead again in this case
we're using 480 bits worth of memory in
order to store the seven or so
characters that is my string so you've
got an overhead of 3.75 to 1 for this if
you add a string with a single character
in it your overhead would actually be 24
to one so if you have to store just a
single character store as a char on its
own don't wrap it in a string object
because you'll save a lot of memory ok
so the summary on Java memory usage is
when you create objects you're creating
significantly more data so more memory
than the data that you put into it so if
you put an int into an integer there's a
lot more for the integer than for the in
any object you create is going to be
bigger than the data than your store
inside it if you move from 32 to 64 bit
turn on compressed references or make
your heap size bigger because all of
those objects get bigger as you move
from 32 to 64 bit and in more recent VMs
we've tried to optimize this we try and
reduce the three words of header down to
two in some cases yes
so the question is why is compressed
references not a default option for
64-bit it is for some vendors for some
releases so it's best to check that it's
enabled for your release I mean IBM does
further Java 7 we did for our update in
one of our updates for Java 6
I know Oracle didn't for Java 6 but it
might be there now in seven I think they
might have I'm not sure
so check because it's it's yeah it's
important ok next object allocation and
reclamation so or allocating objects in
garbage collection so when you run your
application you generally end up
creating a large number of objects which
are temporary right you do a calculation
you add two numbers together and you get
a result often there's temporary objects
created when you do this whenever you
run a transaction through a jve system
there's a huge number of objects which
are created as you go through that
transaction which are thrown away as
garbage so there's lots and lots of
objects being created and destroyed as
you go and that's why there's
generational garbage collection right it
works on the theory that lots of objects
are created and died very young so
there's a nursery collection to clean
them out very very quickly
so the problem with that is that garbage
collections really attacks on your
application the more garbage collection
you have to do the slower your
application runs because your
application stops for garbage collection
to run and that time when it's stopped
running garbage collection your
applications not not continuing to
execute now GC generally can take up to
about 10 percent of your your processing
time in an ideal system it's more like
1% or less but in a badly tuned system
it can be 10% also and we kind of
commonly see that so you can gain some
performance by making sure you create
and destroy objects left off of them but
if the overhead of GC is 10% the maximum
performance boost you can get by
minimizing this
and reducing overhead of GC is 10%.


### JVM Optimization and Moore's Law

I
think Moore's law explains that you know
CPU speeds double every 18 months so
that's going to dwarf what you can do
through GC but what will cover what you
can do and why it's why it's useful so
let's start with string concatenation
something that probably everybody does
so if I've got some code that says I've
got string s1 with string 1 in it
I've got string s2 with string 2 in it
and I can say string s 3 equals s1 plus
s2 so I'm using the plus operator to
concatenate string 1 and 2 into a third
string now when you run the compiler
first it takes that plus operator and
actually converts it into a string s 3
equals new string builder so it creates
a string builder gets the value of the
string for s1 appends s2 to that and
then creates a new string with the
results and you can actually see this if
you run Java C against it and then run a
D compiler or run Java P the pro be able
to tell you that this is what's actually
happening under the covers so if we look
at what happens in terms of allocating
objects so we start off and we've
allocated nothing and we've reclaimed
nothing
so we first do string s1 equals string 1
so I create a string object and that
points to a character array containing
string 1 so that first line of code
creates two objects for me I then do the
second line of code which is ok string
s2 equals string 2 so I've got a string
object in a character array I've now got
four objects created the next thing I do
is I have to do this new string builder
so that gives me a string builder object
and the character array so I then take
my string 1 and I'm doing value of so
that goes in there as part of the
constructor I'm appending string 2 to it
and I do this by copying data so I've
got a string builder that now has string
1 string 2 inside it and then I have to
do the to string so I allocate my string
s3 with a character array and I put that
data into it
so at this point I've allocated eight
objects in order to do three lines of
code now once this is finished it's only
the bit at the right hand side that I
want so the rest of it can go away.

### Java String performance optimization

You
can get rid of string one I can get rid
of string two I can get rid of the
stringbuilder and I've allocated eight
objects and cleaned up six objects in
order to do a very very simple bit of
code so you burn through objects very
very quickly when you're writing code if
you don't think about trying to make
sure you don't burn through objects now
there's actually something that the JV
JVM does on your behalf here called
string interning so in order to prevent
you having to create and destroy all of
those objects the JVM decides to handle
strings slightly differently so what it
does is it stores unique strings what
those basically defined inside quotes
inside an internal data collection
because we know they're going to get
used several times and string keys are
often used inside hashmaps to do lookups
so we do string comparison as much
cheaper to keep a copy of them so you
only ever reference your your your data
straight structure of strings you don't
create multiple copies of them so in
that case we don't actually need to
allocate s-one and s-two because we've
already got a copy of it in memory in
this side the JVM now even more than
that in the compiler what it can do is
because you have string 1 it's defined
and string - it's defined and you've got
a plus operator it will actually work
hang at compile time what the total is
install that as a result because it's
not it knows you're using constant
values if you've done and look up to get
the string it wouldn't be able to do
that so basically the JVM is minimizing
the amount of allocations you've got to
do by caching on your behalf so the way
you minimize allocation and destruction
is to cache data so the summary is
whenever you write code you actually
create and destroy a huge amount of
objects and you do so very very quickly
that's why generational GC is there to
collect only young newly
created objects frequently and the side
effect of having to do that is a
performance tax on your application
write the time spent in GC the GC
overhead is basically a tax on your
application so by reducing the amount of
allocations by reusing objects you
reduce that garbage collection tax and
it is possible that that will actually
reduce your overall memory usage because
you're not creating the same data
several times over you have one copy of
it stored in memory
okay so object caching so when you cache
data you're going to put it into a
collection it's the usual way of doing
it and there's a whole range of
different collections available to you
some of the more commonly used ones are
on the slide there so there are many
more than those there's the new
collections in Java seven more of them
coming in eight and we heard this
morning at the community keynote that
Goldman Sachs are writing their own
collection classes and so on but hash
set hash map hash table linked list
ArrayList these are probably the most
commonly used collections when people
are writing code and they've got a range
of different function right hash map
gives you much more function an
ArrayList does and unsurprisingly it's
got an increase in size so the more
function you have in the collections of
normally the bigger it's going to be in
terms of memory so using the right
collection usually gives you a way of
minimizing your memory usage so what
we'll do is we'll actually have a look
at some of those in detail so hash map
so hash map is an implementation of the
hash of the map interface so this gives
you key value pairs in memory where you
can look up a value against a key and it
gives you the stored value for it now
the amount of memory that uses so first
of all when you create a new hash map
and just do new hash map without putting
any data inside it what you do is you
get a hash map object
and that references an array of hashmap
entry objects and its default size is 16
so you've got a hash matter object and a
16 element array so two objects
allocated as a result of calling new
hash map it's total size is 128 bytes so
you've got a hash map of 16 entries in
it by default when you create it you can
specify its size but if you don't is it
16 and it's going to use 128 bytes even
before you store any memory inside it
each time you put data inside it you
have to create one of these hash map
entry objects to store your key value
pairs so to begin with we've got our
hash map object and we've got our 16
element array of hash map entry objects
so we've got our two objects now as I
said when you want to put data into it
you create a hash map entry object so
when you do put it creates one of these
for you inside that entry object you'll
have an int which is its key hash you'll
have an object for next an object for
key and an object for the value and your
key value pair goes into the key and
value fields so that's an extra 32 bytes
of overhead just for the hash map entry
object and then you put the key in value
into it so there's a lot of overhead and
in fact if you have a 10,000 entry hash
map there's an overhead of 360 K just
for the hash map the array of hash map
entry objects and the hash map entry
objects before you even start looking at
your key value pairs which is your real
data so that's you know a third of a
megabytes worth of overhead before
you're storing your data so what that
looks like is this and what I've done is
the objects which are overhead are in
blue so we've got a hash map and our
array of hash map entry objects and as
you do a put we create a hash map entry
object that goes into that array and
that's got a reference to a key and a
value which is your real data
so when we add another one we get a
second entry in the array so another
hashmap entry object a key and a value
inside it so at this point we've got
four objects of overhead and for objects
worth of data and again you put your
third one in and the same thing happens
and it rose now if you've got what's
called a hash collision so your key
values are too close to each other in in
type what happens is you end up starting
to get a linked list so this is where we
have a next pointer from the hash map
entry so if you've got a hash collision
it will point to the next hash map entry
object so we've now got to a point where
we've got six objects of overhead and
eight objects of value so the more data
store and hash map the more it becomes
efficient in terms of memory usage more
data to overhead ratio okay next linked
lists so a linked list is an
implementation of the list interface and
it's a traversable list of memory so
when you create a linked list doing just
new linked lists you get a linked list
object and like we did before you get a
child object which is your linked list
entry now there's no array here which
means we're not having in a separate
array object is just the linked list
object which is size 48 it's default
capacity is one entry so you're not
having 16 created for you even though
you're not putting anything in it so
it's it's correctly sized so linked
lists are actually far more memory
efficient than hash maps are so to begin
with we have our linked list object and
we need that's all you've got when you
create a new linked list now in the same
way for hash map when you want to store
data we have a wrapper object that your
data goes into we have a linked list
entry object which has three values
inside it previous/next and your data
the entry so one bit of data
two bits of overhead so every time you
put an entry into a linked list there's
24 bytes of overhead plus your data
and for a 10,000 entry link list the
overhead is 240 K so it was 360 for a
hashmap it's 240 for a link list so link
list some more memory efficient they're
not more efficient in terms of
performance because with a hash map you
should be able to go to your hash map do
a lookup of his key and it only takes
one step down the hierarchy to find that
key entry with a linked list you may
have to traverse the entire list so
linked lists take less memory but it can
take longer to find your entry inside
the linked list so what that looks like
is this you have your linked list object
which is just over head so that's blue
and as you add values into it you get a
linked list entry object and your value
so for the first entry our overheads 2
to 1 for a hash map it was 3 to 2 and as
you put more entries in it the overhead
gets better because we have more green
to blue ratio ok next array lists so an
array list is really just an array and a
wrapper object which knows how to manage
and access data from that array so when
you create a an array list at doing just
new ArrayList it creates an array of
size 10 so for hash map its default size
with 16 for linked lists its default
size is zero and you put objects into it
for ArrayList the default size is 10 and
that means when it's empty its size is
88 bytes
now I have to say I've got no idea who
work tank what the default value should
be for hash map it's 16 for a hash table
I think it's like 12 for an ArrayList
it's 10 for other things it's 16 for
some it's 8 there's a huge range of
default sizes and the only way to know
what it is really is to look it up in a
book somewhere with someone stored it
will go to the source code there's no
consistency but again because when you
put some
into an ArrayList there is no overhead
of having like a hashmap entry object or
a linked list entry object the overhead
of a 10,000 entry ArrayList is actually
only 40 K so it's 360 K for a hashmap
240 K for a linked list and 40 K for an
ArrayList so if you can use an ArrayList
is significantly smaller than the other
two collection types and that's because
in terms of a structure it looks like
this you have your ArrayList and then
you just have a huge array of your
entries so there's very little blue to
green ratio so you lots of data not a
lot of overhead okay so other types of
collections string buffer going back to
our add one string to another it uses a
string builder string builders and
string buffers are really other types of
collections because you've got an array
of characters
that's your collection and it manages
that are that collection for you so it's
very much like an ArrayList
unfortunately an ArrayList starts off
with 10 at a 10 entries in it when it's
empty a string buffer has 16 so again
it's another example if you need to know
what the default size is so creating
string buffer with nothing in it has 16
characters that you can use is empty
sizes 72 bytes and the overhead is is
just 24 bytes for a string buffer its
overall overhead is the same as for an
ArrayList it's 40k for a 10,000 entry
string buffer and again it looks pretty
much like an ArrayList so again it's an
example of you can have a more efficient
collection if you want to but and string
buffers basically look like collections
okay so there's our our range of default
capacities hash set 16 hash map 16 hash
table 11 ArrayList 10 string buffer 16
so there's a range of them and there's a
range of empty sizes as a result and
there's a range of 10k overhead so the
overhead if you've got 10,000 entries
you'll notice basically the three which
are hashing and therefore have fast
access
your data are big linked lists sit in
the middle
given the overhead and given that
they've got linear access rates you go
to your link list and you have to walk
down it until you find your data they're
often not worth using and then you've
got a Rea list and string buffer which
are much much smaller if you can store
your data in those formats so hash
collections versus others I said the top
three work were very big and in terms of
overhead they are nine times more
overhead than an ArrayList so array
lists are much more memory efficient if
you can use them but as I said the
access time for a hash table or a hash
map or a hash set should be linear it's
there's only one set of operations
required to actually get to your data
with a linked list you may have to
traverse all of your data with an
ArrayList you may have to know which
slot it's in ahead of time so there's a
range of different collections you can
choose you may want to use hash map or
hash set but you should use them because
you know it's the right thing for you to
use and we actually do a lot of analysis
of people's applications looking for
gaps in in memory usage and you'll see
this later but what typically happens is
when someone writes code they always use
the same collection type right people
get used to writing new hash map or new
hash table particularly hash table or
hash map you're either one or the other
you either hash table guy or you're a
hash map guy and that's just what
happens now I said that each of these
collections has got a default size and
the problem with that is if I was to
create a new string buffer which I said
has a default size of 16 and what I put
into it is my string my string is much
less in size than the 16 entries so I've
actually got my I think it's like nine
characters of my string but I've got 16
entries in order to put that nine
characters in it so I've got seven
additional character entries which I'm
not using so in this case I'm using
extra 112 bytes worth of memory for
nothing right this just sat there is
reserved used but empty memory now
that's actually makes sense in terms of
a string buffer because you're supposed
to be adding things to your string
buffer that's what you're using it to
build a bigger string but what happens
is so I've got my string buffer with
default size of 16 and I've got my
string in it using nine characters now
what happens if I want to start adding
more content into my string so I want to
put in my string of text so I've got my
string of and as I start adding the
characters for text I've got the problem
that I've hit 16 it's full and I want to
put an extra character in so when I add
the T for my my string of text what
happens is it doubles in size it goes
from being 16 entries to 32 entries for
me to put my one extra character in so
I've now gone from having 9 spend spare
count
sorry 7 spare characters when I had my
string to having 15 spare characters
with my string of text in so it's
doubled the size of my collection in
order to put a 1 extra care character in
now that doesn't sound like too much of
a problem it in all honesty right you
think about it and it's like okay 15
characters that's what I don't know
maybe 200 bytes it's not really a huge
deal but think about taking big soap
messages which are going to send
somewhere and if you've got a big string
for a soap message that's 16 megabytes
in size and I add one extra character in
it I'm going to allocate a 32 megabytes
string in a string buffer in order to
put it in there so I'm now wasting 15
Meg of empty space because it's doubled
in size and again in the same way that
the default size for collections is 16
16 11 1 10 16 you know it seems to just
be I've got a magic 8-ball and it's
whatever it said they've done the same
thing for the expansion algorithm most
of the hash sets double in size
apart from hash table
that doubles and adds one linkless grow
linearly so that's good if I put an
extra item in I only pay for my extra
item I'd never have to to pay for empty
space ArrayList they grow by 50% and
string buffers they double so again the
only way to really know how it's going
to grow is to look it up in a book or if
you've got access go and look at it in
the code so there is a trade-off of
memory footprint versus performance
right we see that with using hash
collections versus you know array lists
or all the like using hash tables they
are nine times the size of an ArrayList
but you've got much faster access to
your data so it may well be that 98% of
the time a hash collection is actually
what you want to use rather than an
ArrayList if you can use an ArrayList is
much much smaller but really the
important thing is you should be making
that decision because you know it's the
right collection to use not because your
hash map guy and that's what you do the
other problem is when you put more data
into an array it's going to double in
size or get add 50% so it's important to
make sure that you've sized it correctly
to begin with so there's no spare space
and if you have an idea of how big that
collection is going to need to be in
total if you're only ever going to store
six items in this collection create it
at size six now you want to try and size
you're correct your collection correctly
for what your application actually does
ok so finally application analysis so
we've said there's an overhead to all of
your data right objects are much bigger
than the data you put in it and we've
said that creating temporary objects
causes garbage collection a problem so
really you do want to try and cache data
where you can and where you are caching
data we want to make sure you're using
the right collection and that you've
sized it correctly so there's no empty
space and that we try and avoid it
doubling size or growing by 50% when you
put extra items in so
we want to be able to identify things in
your application
which don't meet those rules where it's
got lots of empty space in it is a good
starting point or where is the wrong
collection type and you can use a lower
head collection type inside your
application so the the interesting thing
is that I said that the overhead of a
hashmaps like 360k for ten thousand
entries so that's actually what a third
of a megabyte is not a huge amount of
data so am I just talking through this
because you know I can and it's not that
interesting it's not going to really
help you so we actually went through an
application and try to quantify the
amount of wasted memory and the number
of collections that are being used so we
ran the IBM WebSphere application server

### Tomcat performance optimization

I work for IBM so I have to do
everything with my technology it's like
the rule if I came in here and talked
about Tomcat I'd probably get fired so
we did this with website application
server with plants by WebSphere which is
a sample application that gets shipped
with it and this was done on WebSphere 7
so it's a couple releases old we're now
on 855 I ran a load test with five users
and that had heat usage of 206 Meg so
there wasn't a big massive application
but it's using two hundred Meg's worth
of memory so what we found was there was
two hundred sixty-two thousand instances
of hash map being used at that point in
time and the total memory used by the
hash by the hash table and the hash
table entry objects was 220 6.5 Meg that
doesn't include the useful data stored
inside it that's just the overhead of
the hash table objects and the hash
table entry objects next we found that
there was twenty thousand five hundred
instances of weak hash map and that was
totaling an overhead of twelve point six
Meg there was ten and a half thousand
hashmaps
there was nine and a half thousand array
lists one on our thousand hash sets one
thousand two hundred and fifty vectors
over a thousand linked
299 treemaps and a grand total of three
hundred and six thousand collections on
the Java heap with the total overhead of
forty two point nine Meg so when we
looked at this the first thing we said
was the guy who wrote plants by Webster
he's a hash-table guy but then we were
scared because it's sort of like sixteen
percent of the live data on the heap is
just the overhead of the collections and
not the data inside it which is the
useful thing so we're wasting you know
forty two point nine megabytes to manage
our data rather than have the data
itself so we thought there's probably
some gains to be had here so the first
thing we did was we we broke out a clips
memory analyzer does anybody know
Eclipse memory analyzer okay so what it
does is it will take an H proof dump
from from hotspot on IBM we use
operating system core files so a core
file on Linux mini dump on Windows and
we load that into memory analyzer
instead but it achieves the same thing
so we load that in and inside memory
analyzer it's got a few functions that
let you go through all of the memory
from your java application and take a
look and see what it's doing and in
particular it's got an option to look at
Java collections and inside Java
collections one of the interesting ones
is called collection fill ratio and this
tells me how full my collection is is it
empty is it 10% full 20% fills 30
percent full and so on so I can choose
to run collection fill ratio against all
of the Java collections on the heap now
it does give you some filter options so
I know the guy who wrote this code was a
hash table guy so I'm only going to look
for hash tables so I can say only find
Java util hash table objects on the heap
and give me a report about how full they
are so of my 1,200 127,000 hash tables
1,200 and 127 thousand of them
were empty they had a fill ratio of zero
so some would have been creating hash
tables not storing any data in them so
they done just hash table my hash table
equals new hash table and they'd
forgotten to do hash table put or they
haven't got any data to put into it yet
so that's roughly 50% of the hash tables
are completely empty we then had 95
thousand of them which were less than
20% fill so if their size is 16 that
means we've got less than what three
entries in it 39 thousand of them were
less than 40 percent full and then we
had a couple which actually had a
reasonable amount of data in them and in
terms what memory analyzer tells you
shallow heap is the overhead of the hash
map entry objects themselves and the
retain heap is that object in everything
it points to afterwards so everything it
references so the retained heap full of
the empty ones that tells me how much
memory I could save if I never allocated
those 127 thousand empty hash tables
okay so from that I can actually select
a row so I can select the row where
they're all empty they've got to fill
ratio of 0 and I can say list objects
with outgoing or incoming references and
going references will show you the list
of the hash table objects and what's
inside the hash tables so you can see
its contents but we know they're empty
so they're content to nothing useful if
we do it with incoming references we get
to see which objects are pointing to the
hash table so basically what code owns
the hash table and when I do that I can
see that I've got a hash table is being
referenced from a memory session and
that seems to be true in a lot of cases
so I now know it's the memory session
which has got the empty hash table in it
so it's a hash table that's being
created to store session data but
there's no session data and we went
through and we actually did this empty
analysis for all of the collections on
the heap and we find from hash table of
the 262,000 of them
48% of them were completely empty
four-week hashmap 99.5% of them were
completely empty so almost every single
week hashmap was empty for hashmap it
was 71% ArrayList it was 48 percent hash
set 55% and so on so on average we're
looking at over 50% of those collections
were completely empty so in the jdk team
we know that sometimes you don't
actually know what data is going to be
put into your collection if you're
creating a framework and you've got an
API for people to store data you don't
know how much they're going to store so
we want to see whether we can do things
at the jdk level that's going to
minimize the overhead of memory and try
and be more intelligent ourselves and we
cache map was actually a really good
example we said 99.5% and empty so
almost all of them are completely empty
and when you get a weak hash map it's
kind of a unique collection because
inside your weak hash map you get an
array of weak apps map entries size 16
so it's exactly the same as a hash map
right 16 array when it's empty and an
owning weak hash map object but you've
also got a reference key and that
reference queue is used to get rid of
objects when they're not being used
anymore like weak hash maps are designed
to be self addicting now you only
actually need that reference key to do a
victim data if there's some data inside
it if there's no data to it for us to
evict we don't need a reference to you
to do that and the reference queue is
actually quite big the size of a
reference queue is 560 bytes and of a
total 688 bytes for the weak hash map so
we did what we thought was the sensible
thing we did lazy allocation of the
reference key it's not until someone
puts something in the weak hash map do
we choose to create the reference queue
in order to be able to take things out
of the hash map and that saved 10.9
megabytes in our example and we were
using what 210 Meg's so we saved 5% just
from a single change to say that's
lazily allocate the reference
the moment someone puts data into my
collection so finally techniques for
minimizing memory lazy allocation of
collections is probably the first and
best thing that you can do so there's no
point creating a collection until you've
got some data to put in it avoid all of
those empty collections by just not
creating them where you would do your
first hash table put you can create your
hash table and then put something in it
until you're putting data in there's no
need to have the hash table at all it's
very very common to have it in the in
the initializer or the constructor of
1/8 of an owning object rather than when
it's actually required so secondly if
you're only storing one object you could
actually just store that object and not
put it in a collection because there's
only one of them and that saves you the
overhead of the wrapping objects so in
the hash map that saves you the hash map
object and the array of hash map entries
so you just you're down to single
objects thirdly size your collections
correctly so if you're only ever going
to store two entries inside a collection
like a hash map you can create it with a
size and the reason for doing that is
you're not going to have if you're only
going to put two in there you're not
going to have 14 empty entries in your
array which you're never going to use so
you get to save memory there item number
four avoid expansion of collections due
to the doubling or the extra 50%
algorithm if there's a chance it's going
to grow you want to try and minimize the
doubling and wasting a huge amount of
spare space afterwards that's not always
easy to do and in all honesty it would
be really nice if the collections had a
pluggable expansion algorithm in so you
could say I know that most of my
collections are going to be under half a
Meg and if they're going to be over half
a Meg then grow them in extensive 1 Meg
at a time rather than continuing to
double but that's not there so it's kind
of if you know there's a maximum size is
going to grow to maybe Alec
getting at the maximum size is the right
thing to do and finally at the bottom
and I didn't cover this but I probably
add a couple of slides in four we said
that if you've got a string buffer
that's size 16 you've got 16 characters
in it and you want to add one more it
doubles in size that's fine but what
happens if you remove entries from a
collection once it's grown if I remove
entries does it shrink the answer's no
so it always stays at the high-water
mark the largest size it's ever expanded
to and if you want to make it smaller
the only way to do that is to create a
new collection at the smaller size and
copy the data over so you have to resize
that collection yourself it's not done
for you yes yeah so the question is when
you've got array type structures like an
ArrayList or a string buffer and it
wants to double in size will grow by 50%
how does it do that
so yes what it does is it creates a new
array at the new size and copies the old
data over so it's a it's a create and
copy it doesn't dynamically extend as
such yes
performance 


### Java collection class optimization

so the question is if you're
using a hashmap
should you really size it correctly for
the number of entries because that gives
a greater chance you're going to have
hash collisions and have that linked
list for access time so you're
completely correct as I said there's a
trade-off between memory and performance
so for hash Maps yes there's an argument
you should have extra capacity your
alternative is to implement your own
hash algorithm for if you know what your
data is so there's ways of minimizing
that if you want to but yes you're right
if you correctly size it you've got a
much better chance of having a hash
collision as you're putting entries in
yes
so I didn't get the first bit
so that the question is often some
people return an empty collection rather
than null because then you don't have to
handle the nullpointerexception
so you could just change your design
pattern okay yeah you're right
but you could equally do that lazily so
rather than creating your collection
when you initialize the wrapper that's
going to return the collection you wait
until someone actually asks for it and
allocate it at that point in return
empty you still have an empty collection
and then the second thing is if you know
there's a good chance it's going to be
empty because that happens then maybe
the right correct size for it is one or
you know created it at a very small size
anything else okay yes
so the question is the the slide I
showed about the Webster application
servers at the current situation so that
was WebSphere seven that we did the
analysis on the weak hash map changed
that we did went into WebSphere eight
since then there's been eight and 855 so
we've moved on a lot but and yes we do a
lot of work now to reduce our memory
footprint so usually when IBM releases a
product we have performance criteria we
have to meet before we can release one
of them is that you have to be faster
than you were the previous release
performance has to get better we now
have two others that we have to do
startup has to be faster so each release
our application server has to start up
faster than the previous one and our
third one is the memory footprint has to
be smaller so each release even with the
new extra features going in we have to
go faster start faster and be smaller

### Eclipse Memory Analyser and Optimizer


it's not a fun job yeah so the question
was I showed Eclipse memory analyzer
tool is there other tools which will do
that that works across implementations
across JVM vendors so firstly memory
analyzer does work for every single
vendor it works for hotspot it works for
the IBM JDK it may not work for J rocket
actually I don't know if J rocket
implements the H Pro format heap dump
but J Rockets going away anyway so but I
do know that Oracle inside Mission
Control is starting to build a tool
called
J overhead or something and that tries
to actually implement exactly the same
thing looking for empty collections but
generally across the board memory
analyzer is the one that's being used
and it's not just by IBM it's an eclipse
project we just contribute to it
you say anything
so creating is a null entry yeah I've no
idea so the question is can you create a
new hash map as null maybe we should
talk afterward so I've got time to think
about the question okay so the summary
is a significant overhead to your
application and particularly shared data
every time you do a new operator there's
data being created by the JVM in order
to manage your memory applications
almost always have the wrong collection
type in use because people are the hash
table guy they've got collections which
are either empty or near empty and by
being careful about the structure that
you choose the collection that you
choose the size that it is and actually
caching where you can cache can improve
your memory efficiency and if you want
to try and do that for your application
memory analyzer tools the thing I would
recommend to do that with and that's the
URL for there's an IBM web page that
explains all the sessions that we're
doing there's a link off there which
lets you download the slides for all of
the IBM presentations it effect goes
through to my SlideShare account for my
presentation so if you try and find me
on SlideShare you will find the
presentation