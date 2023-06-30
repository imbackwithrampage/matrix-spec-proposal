# MSC4027 Custom Images in Reactions

One of the most desired features within the Matrix ecosystem is the ability to
react to messages with custom images. This feature is especially requested by
users who come from Slack and Discord where this functionality is one of the
main ways that the culture of a chat rooms develops.

There is an existing proposal to
[render image data in reactions (MSC3746)](https://github.com/matrix-org/matrix-spec-proposals/pull/3746/),
but it has had little attention recently and has the flaw of not being conducive
to deduplication (either on the client or server). Sorunome proposed a
modification to that MSC to
[use the MXC URI as the key](https://github.com/matrix-org/matrix-spec-proposals/pull/3746/files#r866285147)
which this proposal adopts.

This proposal is meant to replace
[MSC3746](https://github.com/matrix-org/matrix-spec-proposals/pull/3746/) and is
additionally intended to document the behaviour of existing clients and bridges.

Like
[MSC3746](https://github.com/matrix-org/matrix-spec-proposals/pull/3746/), this
MSC does not propose a mechanism for providing a list of available images.

## Proposal

This proposal suggests two changes to events with the `m.annotation` relation.

1. If the `key` of an `m.annotation` relation is an MXC URI of an image, clients
   should render the referenced image instead of the key text.

2. When the annotation's key is an MXC URI, a new (optional) `shortcode` key can
   be added to the content of the event with a textual name for the image. This
   field must be a string and should start and end with the `:` (colon)
   character.

   This shortcode should be shown by clients in situations such as hovering over
   the annotation, as alt-text, or if the client does not support rendering
   images.

Example custom image reaction event content

```json
"content": {
  "m.relates_to": {
    "rel_type": "m.annotation",
    "event_id": "$abcdefg",
    "key": "mxc://matrix.org/VOczFYqjdGaUKNwkKsTjDwUa"
  },
  "shortcode": ":partyparrot:"
}
```

## Potential issues

### Clients rendering the MXC URI as text

The biggest disadvantage is that clients that do not support rendering custom
reactions will render the MXC URI as text. However, this is already problematic
because many bridges and clients already support this MSC, and users likely
already encounter this.

### Un-renderable image referenced in the `key`

The MXC URI could specify an asset that either does not exist, or is not a
renderable image. Clients can opt to render the `shortcode` in these situations,
or some placeholder/error image, or just opt to render the full key.

### Multiple MXC URIs can refer to identical files

Consider two distinct MXC URIs which both refer to a file with identical
contents. If the URIs are used as reaction keys on a given message,
deduplication will not happen as the keys are different. This could cause
confusion as the two reactions would be visually identical despite the keys
being different.

A similar problem exists already in some clients with Unicode emojis because
there is significant inconsistency as to whether skin tone indicators and other
un-renderable characters should be included in the reaction keys generated by
clients. As such, this proposal does not materially worsen the current
situation.

Clients can opt to compare the file contents of the images, but this should not
be required as it would complicate rendering significantly.

Future MSCs may provide a way to share a set of custom reaction images available
in a room/space. This will make it so that users can share custom reaction
images reducing the likelihood of two users uploading the same image under two
different MXC URIs and reacting with them.

## Alternatives

### Use the shortcode as the `key`

This is what was proposed by
[MSC3746](https://github.com/matrix-org/matrix-spec-proposals/pull/3746/). The
problem with this is that there could possibly be multiple distinct images with
the same shortcode. Reactions are only deduplicated based on `key`, so clients
and servers would group these distinct reactions together which is undesirable.

### Put the `shortcode` as a key within `m.relates_to`

Instead of being at the root of the `content` dictionary, the `shortcode` value
could be included within `m.relates_to`. This is the wrong place to put this
value because `m.relates_to` is meant to only contain information pertaining to
the relationship between events, not information about the reaction event
itself.

## Security considerations

### Image is unencrypted

Reaction events are not encrypted, and so the MXC URI referenced by the key
would have to be an unencrypted image. However, this is probably not a problem
for the following reasons:

- Custom reactions are most likely not sensitive information.

- Users are already able to upload unencrypted content into encrypted rooms, so
  this does not introduce any leakage that was not previously possible.

- Clients can add UX to indicate to users that the reaction images are not
  encrypted.

## Unstable prefix

Until this proposal is merged into the spec, the `shortcode` unstable field name
should be `com.beeper.reaction.shortcode`.

An unstable prefix for the `key` in `m.relates_to` is not necessary as the spec
already allows arbitrary data to be used as the `key`. This MSC merely adds
extra meaning to a specific class of key.