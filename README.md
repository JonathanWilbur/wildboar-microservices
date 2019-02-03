# Wildboar Microservices Architecture Standard

This is still a work in progress.

## "Wildboar Services"

The term "Wildboar services" will mean services that are compliant with this
standard.

## Message Serialization

All messages shall be serialized in JSON. Even though binary serialization
formats like Google's Protocol Buffers, ASN.1 Basic Encoding Rules, or
Parquet may be fine for sending messages, the overwhelming majority of data
transferred will come from the message bodies of emails or log messages,
rather than flags or other metadata. This means that using a binary protocol
would afford a protocol almost no data compression.

Also, JSON is probably the most widely supported / understood serialization
format, and in most programming languages, it is supported in the standard
library. If I used another serialization format, I would have to pull in
another dependency.

Finally, JSON is readily accepted by most REST APIs, which makes integrating
components of the Wildboar services far easier.

### JSON Message Structure

#### The Root Element

The root element of each JSON document sent via the messaging queue shall be
an object--not an array. If the data intended for transmission is an array,
it shall still be encoded as a member of the root object; it shall not be
the root object in the message itself.

#### Serializing Identifiers

- UUID-URNs should be serialized as described below.
- IP addresses of all versions shall be serialized as strings.
- MAC addresses and all other related IEEE identifiers shall be serialized
  as strings.
- Object identifiers (as specified in the ITU's X.660) shall be serialized as
  an array of integers.
- URIs, URLs, and URNs shall be serialized as strings.

#### Serializing Time Types

All dates and times MUST:

- Be serialized as strings
- Contain no leading or trailing whitespace
- Comply with ISO 8601:2004 specifications

All services that deserialize dates and times SHOULD:

- Either:
  - Be able to understand ISO 8601 negative or positive years, or
  - Fail gracefully.

#### Serializing Dates

#### Serializing Times

Times by themselves shall never be used. A timestamp must be used instead,
which associates a specific date.

#### Serializing Timestamps

All timestamps MUST:

- Be in UTC, which entails the inclusion of a `Z` at the end of the ISO 8601 timestamp.

#### Unit Tests for JSON-encoding compliance

- The first non-whitespace character of the message is `{`.
- The last non-whitespace character of the message is `}`.
- The second-to-last non-whitespace character of the message is not a comma, `,`.

### Consumer Signatures

The function or method that first accepts a message from the message queue for
processing MUST have a signature that matches a valid AWS Lambda function
written in the same language. If AWS Lambda does not support such a language,
a best attempt MUST be made to match the function or method signature to one
which AWS Lambda would likely use if said language were an option for AWS Lambda.

The reason for doing this is to ensure that consumers can be easily converted
to AWS Lambda functions.

## Identifiers

Applications throughout this environment, where permissible, shall use version
4 UUIDs in the form of a URN as identifiers. In any such uniquely-identified
object, the field or key for which this URN shall be a value shall be named
`id`, with lowercase letters used exclusively, when possible.

An example of UUID-URN looks like this:

```
urn:uuid:10ba038e-48da-487b-96e8-8d3b99b6d18a
```

The following objects MUST be labeled with a UUID-URN, and provide read-only
access to their identifiers to other objects:

- Any object that represents a server
  - This includes raw listening sockets
- Any object that represents a client
  - This includes raw sockets
- Any object that represents a message
- Any object that represents a user or person
  - Identifiers of users or people, such as email addressess, do not apply
- Any object that represents a login session
- Any object that represents a transaction
- Any configuration object or driver

The following objects MUST NOT be labeled with a UUID-URN:

- Lexemes produced by a lexer / Tokens produced by a scanner
- Productions produced by a parser
- Other identifiers or addresses

Any object with a UUID-URN must include its UUID-URN when serialized for
placement in a message queue. When an object with a UUID-URN is mentioned in a
log / event message, the UUID-URN shall be included in the message, even if it
is not transported through the use of a message queue.

The use of a URN both namespaces an identifier and explicitly stating the
specification to which it conforms. To clarify, one could mistake a UUID for a
Microsoft SID, which the URN namespace above prevents by disambiguating.

## Code

If an object includes an `id` field, this field must be the first member of the
object. If an object includes a `creationTime` field, this field must be the
second field if an `id` field is included, or the first field if there is no
`id` field. If the `creationTime` field can be initialized from outside of the
constructor, it MUST be. If the `creationTime` field can only be initialized
from within the constructor, it MUST be initialized first.