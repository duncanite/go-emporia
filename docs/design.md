# go-emporia: the design

Research into the Emporia Vue Utility Connect, the unit that reads a utility's smart
meter, done on 2026-11-15 from public sources, to decide what a Go module for it would
be. Nothing is built and nothing here was run against a unit. The decisions at the end
are open.

In short: Emporia offers no local interface and no public API. The unit can be read
locally only after replacing its firmware, which ends its use with Emporia's app. Short
of that, the app's own cloud API is the way in, undocumented by Emporia and wrapped by
several open projects, in Go among them.

## The device

- The Utility Connect plugs into a wall outlet. It joins the meter's Zigbee Smart Energy
  network, once the utility has registered it against the account, and sends what it
  reads to Emporia's cloud over 2.4 GHz Wi-Fi
  ([how it works](https://www.emporiaenergy.com/how-the-vue-utility-connect-works/),
  [setup](https://help.emporiaenergy.com/en/articles/14833139-setting-up-your-vue-utility-connect)).
  Emporia gives 30 seconds as the interval.
- It reads the whole home at the meter. Emporia's panel monitors, the Vue 2 and 3, are
  other products: current clamps in the breaker panel, a circuit each.
- Inside, an ESP32 talks over a serial line to a Silicon Labs Zigbee module, asking it
  for the meter's reading
  ([a reverse-engineering write-up](https://victorchang.codes/reverse-engineering-the-emporia-vue-utility-connect)).

## Ways in

### Emporia's statement

Emporia's [developer access page](https://help.emporiaenergy.com/en/articles/13297125-developer-access-future-integrations),
dated 2026-03-13: "The devices do not expose a local API or direct network interface —
all data flows through the Emporia cloud", and "Emporia does not currently offer a public
API". Matter is listed as not supported, with no timeline.

### Replacement firmware (local)

[esphome-emporia-vue-utility](https://github.com/nekorevend/esphome-emporia-vue-utility)
is ESPHome firmware for the unit's ESP32, maintained. From its README:

- The first flash is over a serial adapter on pads inside the case; later ones go over
  Wi-Fi.
- It gives power in watts and energy in watt-hours, net, imported and exported, and
  polls the Zigbee module itself.
- "Flashing the firmware of the Vue does not break the pairing": the meter's pairing is
  in the Zigbee module, which the flash does not touch. It advises pairing under the
  stock firmware first.
- "The Emporia app will no longer work." It advises backing up the stock firmware to be
  able to return.

The unit then speaks ESPHome's native API on the LAN, which a Go client would read.

### Emporia's cloud, as the app uses it (unofficial)

Described by [PyEmVue](https://github.com/magico13/PyEmVue): `https://api.emporiaenergy.com`,
a login through AWS Cognito, the token in an `authtoken` header, usage by device and by
scale from one second to a year. No limit is published. Emporia can change it without
notice.

### Around Emporia

The meter can be read by another registered Zigbee gateway, and the utility has its own
data-sharing API, which gives interval data after the fact and not live readings. Neither
is this module.

## Two ways in, compared

| | Replacement firmware | The app's cloud API |
|---|---|---|
| Supported by Emporia | no | no |
| Local at run time | yes | no |
| Needs | the case opened, a serial adapter, one flash | the Emporia account |
| Gives | watts and watt-hours, at the interval the firmware polls | usage down to one second |
| Cost | the Emporia app stops working for the unit | depends on an API that may change |

## What this module would be

Not decided. The options, most local first:

1. **A reader of the reflashed unit**, over ESPHome's native API: protobuf over TCP,
   optionally encrypted. The Go work is that client, or the choice of an existing one; the
   hardware step is the owner's.
2. **A client of the app's cloud API**: the Cognito login, the device list, usage.
   [go-emporia-vue](https://pkg.go.dev/github.com/alex-savin/go-emporia-vue) is a young Go
   client of it, to read more than to depend on.

## Open questions

- Whether the unit is the Utility Connect, as supposed here, or a panel monitor.
- Whether opening it and giving up the Emporia app is acceptable.
- What interval is wanted: live watts, or minute history.

## Not verified

- Nothing above was observed on a unit.
- The radio modules' FCC identifiers and the Zigbee profile's version come from search
  excerpts of an Emporia document that could not be read.
- How often the replacement firmware reads the meter by default.
- Whether a meter accepts a second gateway beside the Emporia unit.

## Sources

- Emporia: [how the Utility Connect works](https://www.emporiaenergy.com/how-the-vue-utility-connect-works/),
  [setting it up](https://help.emporiaenergy.com/en/articles/14833139-setting-up-your-vue-utility-connect),
  [developer access and future integrations](https://help.emporiaenergy.com/en/articles/13297125-developer-access-future-integrations).
- [esphome-emporia-vue-utility](https://github.com/nekorevend/esphome-emporia-vue-utility):
  the replacement firmware, what it needs and gives, what it costs.
- [Reverse engineering the Emporia Vue Utility Connect](https://victorchang.codes/reverse-engineering-the-emporia-vue-utility-connect):
  the two chips and the line between them.
- [PyEmVue](https://github.com/magico13/PyEmVue): the app's API.
- [go-emporia-vue](https://pkg.go.dev/github.com/alex-savin/go-emporia-vue): a Go client
  of it.
