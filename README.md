# cloud-itonami-iso3166-jpn-mic

Open ISO 3166 Agency Blueprint for **JPN-MIC**: Ministry of Internal Affairs and Communications
(総務省, MIC) — a Japan-agency-level LEAF under
the `cloud-itonami-iso3166-jpn` country-level coordinator.

This repository designs a forkable OSS business for an independent
compliance consultant: an already-incorporated operator (typically one
already using `cloud-itonami-iso3166-jpn` for general Japan market entry)
gets a Compliance Advisor + independent **Telecom-Licensing Compliance Governor** to
navigate telecommunications carrier licensing under the Telecommunications Business Act (電気通信事業法) and broadcasting-license rules for an operator delivering a public-sector network, telecom, or broadcasting contract.

This is the final repo in the Japan agency-level sweep started by
ADR-2607040100 — with this blueprint published, all 19/19 Japan central-
government bodies in `kotoba-lang/iso3166` are `:maturity :blueprint`.

## No robotics premise — digital/data service exemption

Agency-specific compliance navigation is a pure data/software service with
no physical-domain work — the same exemption class as `cloud-itonami-6310`
and `cloud-itonami-gtin-*`. `blueprint.edn` sets
`:itonami.blueprint/robotics false` and `:required-technologies` lists only
real capabilities (`:identity`, `:forms`, `:dmn`, `:bpmn`, `:audit-ledger`),
no `:robotics`.

## Core Contract

```text
operator intake + prior filing/compliance history
        |
        v
Compliance Advisor -> Telecom-Licensing Compliance Governor -> compliance draft, or human sign-off
        |
        v
gated filing / registration / compliance-program submission + audit ledger
```

No automated proposal can submit a filing or registration the governor
refuses, suppress a compliance record, or claim a legal conclusion the
governor has not cleared. `:filing/submit` is never in any phase's `:auto`
set — it always requires human sign-off (mirrors `cloud-itonami-M6910`'s
`filing-submit-never-auto-at-any-phase` invariant).

## What this is NOT

- **Not Ministry of Internal Affairs and Communications (総務省) itself, and not the
  government of Japan.** See [`docs/business-model.md`](docs/business-model.md)
  for the boundary with `com-etzhayyim-ooyake`, `matsurigoto`,
  `com-etzhayyim-toritsugi`, `legal-entity.etzhayyim.com`,
  `cloud-itonami-M6910`, and the country-level `cloud-itonami-iso3166-jpn`.
- **Not legal or tax advice.** Every regulatory claim must cite the
  official MIC source and route final filings to
  Japan-licensed counsel or a registered agent where the law requires
  licensed representation.

## Regulatory source register

The citation requirement above has a referent: [`facts.edn`](facts.edn), a
register of the statutes and official pages this repository is allowed to
build a requirement on. A law or page that is not in that table has **no
spec-basis here** — extend the table, never invent an id or a URL.

Re-check it against the live authorities:

```bash
nbb scripts/verify-facts.cljs
```

Three exit codes, and the third is the point:

| exit | meaning |
|---|---|
| `0` | every source re-fetched and matched |
| `1` | a source did not check out — **the register is wrong** |
| `2` | the run could not answer — **not a pass** |

`2` exists because a check that could not run must not return the same value
as a check that ran and found nothing. Three measured hazards on MIC's own
hosts make that distinction load-bearing rather than decorative:

- **`www.soumu.go.jp` serves Shift_JIS and does not say so in the HTTP
  header.** A body read with the default UTF-8 decoding does not throw — it
  returns stable mojibake, so the live top page's title `総務省` reads as
  `������`. Left unhandled that reports every live page in the register as a
  dead citation, or, if an author pins the mojibake, passes forever while
  asserting nothing.
- **On `www.tele.soumu.go.jp`, a dead citation and a blocked client are the
  same bytes.** A real page fetched without a browser User-Agent and a page
  that does not exist both return 403 with an identical 1,727-byte body
  (`sha256 3559970027daab21…`). That host therefore carries no page entries,
  and the byte-identity is re-measured every run so that the day the block
  lifts is noticed rather than assumed.
- **`laws.e-gov.go.jp` cannot be checked as a page at all** — a real law URL
  and an invented one agree on status, final URL and title. Statutes are
  resolved through the law API instead.

Force status is checked as two fields, not one: `特定通信・放送開発事業実施
円滑化法` was repealed in 2024 and still answers with the title and law
number a citer would have written down, while `413AC0000000137` is not
repealed at all yet carries `PreviousEnforced`. Each half has a real
statute as its negative control.

## Capability layer

Resolves via [`kotoba-lang/iso3166`](https://github.com/kotoba-lang/iso3166)
(code `JPN-MIC`, `:parent "JPN"`, cross-referenced to ooyake's
`gov.jpn.mic`). Required capabilities:

- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
