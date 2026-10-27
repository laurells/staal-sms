# staal-sms

SMS through Termii: a [Staal](https://github.com/laurells/staal) package that gives
programs the `sms.termii.send` capability, built on Staal's general `net.http`
capability. It talks to [Termii](https://termii.com).

## Authority

This is everything the package can reach. `staalc fetch` prints it when the
package is added, and the compiler refuses the package if its code reaches
beyond it.

| Capability | Targets |
| --- | --- |
| `net.http.post` | `https://api.ng.termii.com/api/sms/send` |

## Install

Add the package to your project's `staal.json`, pinned to an exact commit:

```json
{
  "dependencies": {
    "sms": {
      "git": "https://github.com/laurells/staal-sms",
      "commit": "<full 40-character commit id>"
    }
  }
}
```

Then fetch it. This writes `staal.lock` with the SHA-256 of every file, and every
later check verifies the stored copy against it:

```
staalc fetch
```

## Use

The capability's action:

```
sms.termii: send(to: Text @target, text: Text) -> Text
```

A program imports the module and asks for the capability with its own limits:

```
import sms.termii

fn remind(phone: Text, texts: cap sms.termii.send(only: ["2348012345678"], uses: 1)) -> Receipt {
    return texts.send(phone, "See you tomorrow.")
}
```

The operator supplies what the library needs in the run's `grants`:

```json
{
  "grants": {
    "texts": {
      "cap": "sms.termii.send",
      "needs": {
        "http": {
          "cap": "net.http.post"
        },
        "key": {
          "env": "TERMII_API_KEY"
        },
        "sender": "YourSenderID"
      }
    }
  }
}
```

## Notes

`only` lists phone numbers in canonical form: international, without `+`, spaces or dashes (`2348012345678`). The library turns `+234 801 234 5678` into that form before the check.

`sender` must be a sender ID Termii has approved for your account.

The API key is read from the environment variable named in the grant and never appears in a URL or a log.

## Status

Tested end to end against a local server that answers like the service, as part of
the [Staal test suite](https://github.com/laurells/staal/blob/main/tests/packages_test.rs).
Not yet run against the live service.

## License

Copyright 2026 Laurels Ozy Echichinwo. Licensed under the
[Apache License, Version 2.0, with the LLVM Exceptions](LICENSE)
(`Apache-2.0 WITH LLVM-exception`), the same terms as
[Staal](https://github.com/laurells/staal). The licence grants no right to the
name "Staal".
