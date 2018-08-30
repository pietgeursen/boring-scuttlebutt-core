# Boring Scuttlebutt Core

> WIP (snarky) spec for a stripped down scuttlebutt using as much standard tech as possible.

## What? Why?

I'm frustrated at the difficulty and complexity of trying to implement ssb in other languages. There have been other efforts and none of them have succeeded. I'm also frustrated at the difficulty of building this on mobile.

I come from an electronics design background and part of the job was researching existing chips (Integrated Circuits) and combining them in ways to solve your problem. _If_ you couldn't find any chips that solved your problem, then maybe you'd think about solving the problem yourself with discrete components.

The drawback of doing discrete design is that it takes ages. And you suddenly realise how damn talented those engineers are who make the chips. And maybe you get a bit humble. 

I've been mulling on Rich Hickey's talk about simplicity vs complexity a lot recently. I think that's my main criticism of the ssb stack at the moment. It weaves together (makes complex) many things.  

I see the absolute core part of ssb as:

A sequence of messages by an author where:
 - Every message references the hash of the previous message
 - Every message is cryptographically signed by the author.

And that's it!

No need to weave in:

multiserver addresses
secret handshake
pull-streams
weird-streams
muxrpc
flume-db
some un-specified V8 weirdness
in-process plugins


## Yeah, but what about replication, gossip, security, live streams, authorisation?

Not here. Maybe in different modules if you want. 

As far as security goes, how does the rest of the world do it? Oh TLS. Let's do that. Projects like let's encrypt might be a good starting point.
Security is hard and I'm not that smart. I'd rather use something that really talented engineers have designed. And I want to push it into a black-box that can be upgraded easily as things change and vulnerabilities are found.

## Database choice

> Oh, this looks like a **block chain**! Yay, append only logs! (starts writing ICO)

Um no. 

Sure, it's useful to conceptualise the sequence of messages as a chain. Cool, we don't need that in the data model. Because, you know what it is more than an append only log? It's a whole bunch of messages referencing other messages. We're using it as a social network after all. 

All we have to do to enforce the "append only logginess" is validate any inserted messages reference their parent and are are signed by their author.

We're going to use a relational database. Guess what, they're available in iOS and Android too!

And guess what, a whole bunch of really talented engineers have spent time optimising them. And you can choose what type of db you want. Postgres? sqlite? I don't care, do what you feel.


## Possible REST API

OMG what? A REST api? Oooh yeah.

If you're building a phone app you might just talk directly to your db through your sdk.

### Get all the messages in a feed.

`GET /api/v1/feeds/<pub-key>` 

### Get a specific message in a feed.

`GET /api/v1/feeds/<pub-key>?hash=<message-hash>`

### Add a new message to a feed.

Server validates that the new message references the previous message and verifies the signature on the message.

`POST /api/v1/feeds/<pub-key>`


### Other queries you might want to support

`?type=<post|about|contact|etc>`
`?root=true`

### Other endpoints you might want to support

backlinks
