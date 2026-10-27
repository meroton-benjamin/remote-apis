# Platform Lexicon

This lexicon defines standard platform property names and values that servers
MAY support in the `Platform` message to facilitate interoperability. New
values can be added by submitting a PR against this repository, which requires
signing the [Google CLA](https://opensource.google/documentation/reference/cla).
If signing the Google CLA is undesirable, you may submit an issue instead.

## Platform properties

Platform properties are requirements. A server that does not accept a given
property name SHOULD reject the execution request, and a server that accepts a
property is expected to honor it. Platform properties are part of the action
digest, so changing them changes the action cache key.

The following standard property `name`s are defined:

* `OSFamily`: This describes the operating system family. Multiple values
  are not allowed and an exact match is required.

  The following standard values are defined:

    - `aix`
    - `freebsd`
    - `linux`
    - `macos`
    - `sunos`
    - `windows`

  Additional values may be defined by the server. For other POSIX systems
  the recommendation is to use the output of `uname -s` in lower case.

* `ISA`: This describes the instruction set architecture including
  instruction set extensions and versions. Multiple values are allowed. If
  multiple values are specified, they are AND-ed together: the worker is
  required to support all of the listed values.

  The following standard instruction set architecture values are defined:

    - `arm-a32` (little endian)
    - `arm-a32-be` (big endian)
    - `arm-a64` (little endian)
    - `arm-a64-be` (big endian)
    - `arm-t32` (little endian)
    - `arm-t32-be` (big endian)
    - `la64v100` (little endian)
    - `power-isa-be` (big endian)
    - `power-isa-le` (little endian)
    - `rv32g` (little endian)
    - `rv64g` (little endian)
    - `sparc-v9` (big endian)
    - `x86-32`
    - `x86-64`

  The following standard instruction set extension and version values are
  defined:

    - `arm-neon`
    - `arm-sve`
    - `arm-vfpv3`
    - `arm-vfpv4`
    - `armv6`
    - `armv7`
    - `armv8`
    - `x86-avx`
    - `x86-avx2`
    - `x86-avx-512`
    - `x86-sse4.1`
    - `x86-sse4.2`

  Additional values may be defined by the server. Vendor-neutral names are
  recommended.

  Clients SHOULD NOT request instruction set extensions or versions without
  requesting an instruction set architecture.

  Examples with multiple values:

  ```json
  // (Platform proto)
  {
    "properties": [
      {
        "name": "ISA",
        "value": "x86-64"
      },
      {
        "name": "ISA",
        "value": "x86-avx2"
      }
    ]
  }
  ```

  ```json
  // (Platform proto)
  {
    "properties": [
      {
        "name": "ISA",
        "value": "arm-a64"
      },
      {
        "name": "ISA",
        "value": "armv8"
      },
      {
        "name": "ISA",
        "value": "arm-sve"
      }
    ]
  }
  ```

  ```json
  // (Platform proto)
  {
    "properties": [
      {
        "name": "ISA",
        "value": "arm-a32"
      },
      {
        "name": "ISA",
        "value": "armv7"
      },
      {
        "name": "ISA",
        "value": "arm-vfpv4"
      }
    ]
  }
  ```

## Platform hints

Unlike platform properties, platform hints are optional and advisory. A server
MAY ignore any hint it does not recognize or cannot satisfy, and MUST NOT
reject an action solely because it carries an unknown hint. Hints are intended
for use cases such as resource preferences (e.g. "this action would benefit
from at least 8 GiB of RAM") and routing preferences (e.g. "prefer a worker
with a persistent worker for this hash", or "prefer a worker that recently
fetched a similar input root").

Hints are supplied using the `PlatformHints` message, which may appear in two
places:

* `Action.platform_hints` is part of the action digest and therefore affects
  the action cache key. Use it only when a different hint value may
  legitimately require a different cached result, or when the client
  explicitly wants to split the cache per hint value.
* `ExecuteRequest.execution_hints` is not part of the action digest and does
  not affect the action cache key. Use it for scheduling and routing hints that
  never change the result of the action.
