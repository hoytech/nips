NIP-XX
======

Event Lifetime Preferences
--------------------------

`draft` `optional` `author:hoytech`

This NIP makes [NIP-33](33.md) `d` tags work on events of any kind, and specifies an `ephemeral` tag that lets users create [NIP-16](16.md) ephemeral events of any kind. It supersedes, but is backwards compatible with, [16](16.md) and [33](33.md). Relays SHOULD advertise all of 16, 33, and XX in their [NIP-11](11.md) `supported_nips` field.

Implementation
--------------

* If an event (of any kind) has one or more `d` tags, the first tag's value is considered a *replacement key*. `d` tags without a value are considered to have a replacement key of `""` (empty string). Upon receipt of an event with the same pubkey, kind, and replacement key as an existing event, a relay SHOULD store/retain the event with the newer `created_at` timestamp and delete/discard the other. If the `created_at` fields are the same, the event with the lowest `id` (first, in lexical order) SHOULD be stored/retained, and the other deleted/discarded.
* If an event (of any kind) has one or more `ephemeral` tags, this event is considered *ephemeral*. Upon receipt of an ephemeral event, a relay SHOULD send it to all clients subscribed to a matching filter, but SHOULD NOT store it.

Interactions
------------

* If an incoming event has both `d` and `ephemeral` tags, any event matching the pubkey, kind, and replacement key SHOULD be deleted, but a relay SHOULD NOT store the new event.
* If [NIP-09](09.md) is supported and a deletion event has a `d` tag, it MAY be replaced as specified in this NIP, and newly referenced `id`s SHOULD be deleted. If a deletion event has an `ephemeral` tag, the referenced `id`s SHOULD be deleted, but the relay MAY discard the deletion event instead of storing it.

Backwards compatibility
-----------------------

Previous NIPs specified various event kinds as having replacement and ephemeral behaviour. For backwards compatibility, relays SHOULD implement the following rules, or an equivalent behaviour:

* Events of kinds 0, 3, 41, 10000-19999, and 30000-39999 (inclusive) have implicit `d` tags with `""` values added to the *end* of their tag list.
* Events of kinds 20000-29999 (inclusive) have an implicit `ephemeral` tag added to their tag list.

Relays MAY or MAY NOT return events with implicit `d` tags to a `#d` filter.

Use cases
---------

* Replaceable events (events with empty-valued `d` tags) are intended for events where it only makes sense to store the most recently issued version, for example a user's kind 0 metadata.
* Parameterised replaceable events (events with `d` tags containing varying values) are intended for events that have multiple different channels, each of which it only makes sense to store the most recently issued version. For example, a user may choose to shard their [NIP-02](02.md) contact list into `family`, `friends`, etc. channels.
* Ephemeral events are intended for transient data that doesn't make sense for the relay to store. For example, "user is typing..." indicators in a chat app, or [NIP-42](42.md) `AUTH` messages.

Security considerations
-----------------------

If a relay supports [NIP-09](09.md) then this NIP does not grant users additional capabilities, since users could always manually delete events at the end of their lifetimes. Additionally, if a relay supports [NIP-40 expirations](40.md), then ephemeral events could be emulated with short-term expirations.

* Relays MAY choose to not implement this NIP for deletion events if they wish to prohibit "undelete" actions.
* When replicating events between relays, care should be taken so that ephemeral events do not cause infinite loops. This could occur accidentally if somebody is replicating from relay A to relay B, and somebody else is replicating from relay B to relay A.
