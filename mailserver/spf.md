# SPF: Sender Policy Framework

SPF is one of the three components that try to fix email. It allows an
email domain to define where its message are sent from.

All the details can be found in
[RFC 7208][https://tools.ietf.org/html/rfc7208]. The following is a TL;DR
kind of version of how to configure SPF for a mail domain.

The SPF definition for a mail domain is provided via exactly one TXT DNS
resource record for the domain (i.e., the owner of the record is the mail
domain itself).

The content of the TXT record starts with `v=spf1` followed by a sequence
of space separated terms which are either _directives_ or _modifiers._

## Directives

Directives start with an optional modifier followed by a mechanism. The
are evaluated in order. The first matching directive produces the result.

The possible results are expressed by the qualifier which is the first
character of the directive. The qualifiers are:

`+`
: ‘pass’ – sending is authorized;

`-`
: ‘fail’ – sending in not authorized;

`~`
: ‘softfail’ – sending is probably not authorized, other facts need to be
  considered for a definite statement;

`?`
: ‘neutral’ – no assertion is made.

If no qualifier is present (i.e., the directive starts with any other
character), `+` is assumed.

The rest of the directive is a mechanism that describes how the term is
matched. The mechanism are:

`all`
: match everything; typically used as a final term to refuse everything
  else as `-all`;

`include:<domain>`
: recursively evaluate the SPF record for `<domain>`;

`a`
: matches if the sending IP address is included in the A and AAAA records
  for the mail domain;

`a:<domain>`
: matches if the sending IP address in included in the A and AAAA records
  for `<domain>`;

`mx`
: matches if the sending IP address is an A or AAAA record for the host
  given in the MX record of the mail domain;

`mx:<domain>`
: matches if the sending IP address is an A or AAAA record for the host
  given in the MX record of `<domain>`;

`ip4:<network>`
: matches if the sending IP address is a IPv4 address contained in the
  given network.

`ip6:<network>`
: matches if the sending IP address is a IPv6 address contained in the
  given network.

`exists:<expr>`
: allows constructing a domain name for an A record query and matches if
  any A records exist there. See the RFC for details.

## Modifiers

Modifiers describe additional rules. They should be at the end of the
record’s content but that is not enforced. Their order does not matter.

Modifers are:

`redirect=<domain>`
: if no directives match, expand `<domain>` as per the macro rules from
  the RFC and try there;

`exp=<domain>`
: if the result of processing is ‘fail,’ expand `<domain>` as per the
  macro rules and use the TXT records published there as the explanation
  text.

