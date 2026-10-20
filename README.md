# Amptec AV Device & Module Library

A curated, version-controlled collection of professional audiovisual (AV) **Device Type** and **Module Type** definitions for Amptec's NetBox environment.

**Repository:** [amptec-support/AV-library](https://github.com/amptec-support/AV-library)  
**Maintainer:** Amptec Engineering  
**Branch used by the importer:** `master`  
**Status:** Library under active development; definitions must be checked before use in production designs.

## 1. Purpose and responsibilities

The library describes **reusable physical hardware**: manufacturers, models, orderable variants, ports, optical connectors, power connections, modules and rack characteristics.

| System | Responsibility |
| --- | --- |
| **GitHub AV-library** | Version-controlled definitions of hardware **types** (not individual assets) |
| **NetBox** | Actual assets, serial numbers, site/rack placement, physical cabling, IP addresses, VLANs and intended connectivity |
| **Zabbix** | Operational monitoring, metrics, availability, events and alerts |

A future NetBox–Zabbix integration is envisaged, but **is not part of the current library implementation**. Never put customer-specific inventory or secrets in this repository.

## 2. Scope and principles

- Include professional AV equipment from all relevant manufacturers, **including discontinued products**.
- Build the library **manufacturer by manufacturer**, then **series by series**. Luminex is the first focus.
- Research the complete product family and its manufacturer-confirmed SKUs before generating YAML.
- Create **separate Device Types for meaningfully distinct fixed physical hardware variants**, notably different fixed external optical connector formats, PoE/non-PoE hardware, different power hardware and different physical port arrangements.
- Do **not** create new Device Types solely for firmware, licenses, customer settings or **user-replaceable transceivers**.
- Do not claim undocumented hardware details. Leave unknown fields out, record open questions in `research/`, and validate before marking a definition as verified.
- Prefer the correct NetBox field to prose in `description`. **Do not repeat information already conveyed by the component name, type, label or dedicated field.**

The objective is **accurate physical documentation**, not the smallest number of files or the largest possible catalogue.

## 3. Repository structure

```text
AV-library/
├── README.md
├── device-types/
│   ├── Luminex/
│   ├── Netgear/
│   └── ...
├── module-types/
│   ├── Luminex/
│   └── ...
└── research/
    └── Luminex/
        ├── inventory.csv
        ├── INVENTORY.md
        └── sources.md
```

- **One YAML per distinct physical hardware variant**, using a meaningful, stable filename, for example `GigaCore-20t-10G-P500.yaml`.
- Keep exact manufacturer model names, unique stable slugs and validated manufacturer part numbers (`part_number`). File names and naming examples are **not** proof of a manufacturer's SKU.
- Keep source links, uncertainty and verification status under `research/`. Research files do not belong in the importer device/module-type directories.
- For many closely related variants, generate YAML from a shared specification or script when practical, then **review each resulting file**.
- Existing files must not be silently replaced by definitions that omit previously known fields.

## 4. Choose the correct NetBox component

**Physical position is not the same as NetBox component type.** An Ethernet jack on the front of a switch is still an **Interface**, not a **Front Port**.

| NetBox component | Appropriate use | AV example |
| --- | --- | --- |
| `interfaces` | Active data/network endpoints | Ethernet, management Ethernet, SFP/SFP+, fiber network port |
| `front-ports` / `rear-ports` | **Passive** pass-through connectivity | Fiber patch panel, passive connector panel |
| `console-ports` | Local serial or similar console input | RS-232/serial management console |
| `console-server-ports` | Ports serving other devices' consoles | Serial console server |
| `power-ports` | Physical power **inputs** | AC input, external redundant DC input |
| `power-outlets` | Physical power **outputs** | AC power-through, PDU output |
| `module-bays` / `module-types` | Genuinely modular, installable hardware | Replaceable PSU or I/O card where applicable |
| `device-bays` | Bays accommodating separate complete devices | Supported chassis/subdevice arrangements |
| `inventory-items` | Appropriate non-module physical components | Certain accessories or non-modular spare parts |

Do not invent Front/Rear Ports to represent the front/rear location of an active interface. Do not fake DMX, audio or video connectors as Ethernet interfaces. Where native NetBox fields cannot faithfully represent an AV connector, document the limitation and decide a consistent approach before publishing.

## 5. Interface naming and descriptions

- Prefer the **actual port numbering printed on the hardware** (`Port 1`, `Port 9`, etc.) where confirmed.
- Record actual network medium/speed in the `type` field and use `poe_mode` and `poe_type` where applicable.
- Descriptions must be **short, factual and non-redundant**: physical side, external connector family and relevant optical medium only when not represented elsewhere.
- Do not repeat port numbers, RJ45 compatibility of EtherCON, generic SFP explanations, link duplex/TX/RX facts, or the number of fibers normally used by a duplex link.
- Do not put PoE capability in `description` when it can be represented by the dedicated PoE fields.

Examples of acceptable descriptions:

| Component | `description` |
| --- | --- |
| Front copper interface | `Front EtherCON` |
| Rear copper interface | `Rear EtherCON` |
| Fixed front optical connector | `Front Fiberfox 2ch MMF` |
| Fixed front optical connector | `Front Fiberfox 4ch MMF` |
| Fixed front optical connector | `Front Neutrik opticalCON DUO SMF` |

An example PoE-capable Ethernet interface:

```yaml
interfaces:
  - name: Port 1
    type: 1000base-t
    description: Front EtherCON
    poe_mode: pse
    poe_type: type4-ieee802.3bt
```

This PoE example applies **only when the specific hardware supports IEEE 802.3bt Type 4**. Verify the standard per variant. Omit `poe_mode` and `poe_type` on non-PoE and non-PSE ports; do not blindly copy PoE settings onto optical ports.

## 6. Fixed optical connectors and cabling

Keep **fixed external connector variants as separate Device Types** when relevant to installation or cabling: Fiberfox 2ch/4ch, Neutrik opticalCON DUO/QUAD, etc. The device type and interface description must identify the real physical connector.

For the currently discussed **Luminex GigaCore 10t configurations**:

- **Fiberfox 2ch:** two fiber positions, used for **one duplex optical network link** (TX/RX).
- **Fiberfox 4ch:** four fiber positions, of which **two are used for one duplex optical network link** in these configurations; do **not** create a second network interface just because the connector has four positions.
- Count **independent network ports**, not fiber strands or connector positions. Two actual optical network ports still require two interfaces; verify how many external connectors the exact SKU provides.
- A fixed internal optical transceiver connected to a rugged panel connector is **not automatically a user-installable NetBox Module**. Model a module only when the hardware is genuinely modular and the representation is appropriate.
- Optical medium (MMF/SMF), interface speed, physical connector family and the actual transceiver are **different properties**. Do not conflate them.

**Future cabling:** NetBox can associate cables with network interfaces, but it does **not automatically enforce compatibility between every proprietary rugged connector**. A Fiberfox/opticalCON connection may also involve a multi-fiber assembly that maps to one or more duplex links. Before modelling real cables, test how to represent the **physical cable assembly** and each independent logical connection without double-counting the same physical cable. A description alone is not a connector validation rule.

## 7. Power and PoE modelling

- Use the valid **NetBox connector identifiers** (for example, `neutrik-powercon-true1`, not `powercon-true1`). Check values against the installed NetBox version before importing.
- An **input** is a `power-port`; a genuine feed-through **output** is a `power-outlet`.
- For an outlet electrically fed by a specified input, use `power_port` to link it where the importer/schema supports this mapping.
- Keep validated power-budget fields such as `maximum_draw` and `allocated_draw` where present. They represent different quantities; do not invent `allocated_draw` without a basis.
- **Do not discard existing `maximum_draw` values** when adding a new connector. Confirm whether published power figures describe a single power input, the whole device, PoE output budget or input power inclusive of downstream loads.
- For redundant input feeds, **do not automatically duplicate the full device maximum draw on every input**. Document any unresolved allocation between supplies in research notes; NetBox should not imply double consumption.
- Account separately for downstream loads powered through a feed-through outlet when dimensioning circuits, PDUs and UPSes.

### Luminex examples agreed during setup

| Device | AC input | AC output | Separate redundant DC input |
| --- | --- | --- | --- |
| **GigaCore 10t** | PowerCON TRUE1 | No | No |
| **GigaCore 20t non-PoE** | PowerCON TRUE1 | PowerCON TRUE1 | No |
| **GigaCore 20t PoE** | PowerCON TRUE1 | PowerCON TRUE1 | Yes, `SECOND PSU` for external RPSU 5580 |

For example, the GigaCore 20t PoE physical components can be described as follows, **while preserving previously verified `maximum_draw` values on the existing power inputs**:

```yaml
power-ports:
  - name: Power In
    type: neutrik-powercon-true1
    # Keep the variant's previously verified maximum_draw here.

  - name: Second PSU
    type: other
    description: RPSU 5580 DC input
    # Do not guess the DC input's maximum_draw.

power-outlets:
  - name: Power Out
    type: neutrik-powercon-true1
    power_port: Power In
```

This is a **component-layout example**, not a complete replacement for any existing YAML. In particular, do not remove `maximum_draw` from an existing definition when copying these snippets. Record and verify the AC/DC redundancy behavior and supported power draw separately for each hardware variant. The external **RPSU 5580** should eventually have its own Device Type so that its connections can be documented.

## 8. Research, validation and quality gates

Track each planned model or SKU in the research inventory using:

| Status | Meaning |
| --- | --- |
| **Research** | Model found; sources or physical details still incomplete |
| **Ready for validation** | YAML prepared using traceable sources; needs review/import test |
| **Verified** | Physical details checked, YAML/schema validated and successfully inspected in NetBox |

Preferred evidence:

1. Official manufacturer datasheet, manual, ordering guide and hardware drawings.
2. Official archived documents for discontinued hardware.
3. Photos of labels, front and rear panels, and connector details; other secondary sources only when clearly identified and checked.

**Validation checklist for every Device Type:**

- Exact model, orderable variant, SKU, unique slug and file name.
- Rack height/width/full-depth, weight and other documented physical properties.
- Full port count, manufacturer labels, speed, interface media, PoE mode/type and side placement.
- Fixed external optical connector type, MMF/SMF, and number of **independent optical network links**.
- All AC/DC power inputs, outputs, redundant supplies, module bays and console ports where actually present.
- Power draw fields preserved and independently reviewed for units and meaning.
- No duplicate or unnecessarily verbose descriptions.
- YAML syntax and NetBox field/choice compatibility checked.
- Test **Load → Import → inspect component templates and a test device**. A successful import does not by itself prove the hardware definition is complete or correct.

For revisions, **compare old versus new**: never lose valid ports, PoE settings, `maximum_draw`, dimensions or part numbers inadvertently.

## 9. GitHub and NetBox workflow

**During initial setup, Amptec intentionally works directly on `master`.** Separate feature branches and pull requests are **not required at this stage**; this can be revised once the library becomes business-critical or multiple contributors are editing concurrently.

1. Identify manufacturer and product family; add model/SKU to the inventory.
2. Confirm official documents and hardware-specific differences.
3. Create or revise the appropriate YAML file(s), retaining correct existing data.
4. Validate the YAML and schema; review physical components and descriptions.
5. Commit directly to `master` during the setup phase.
6. In NetBox, click **Load** to refresh the importer's catalogue, then **Import** the selected types.
7. Inspect created NetBox component templates and test-device ports; correct YAML and repeat if necessary.
8. Update verification status and research sources. Retire outdated drafts and avoid duplicate Device Types.

**Installed importer:** `netbox-device-module-type-importer` v0.0.3 with NetBox 4.7.3 during initial setup. The plugin contains **hard-coded `master` GitHub references**. Keep `master` available; review compatibility if upgrading the plugin/NetBox. It reads GitHub through GraphQL using the configured `repo_owner`, `repo` and a server-side credential.

**Important operational details:**

- **Load** refreshes the plugin's **staging catalogue**; **Import** creates the selected type definitions in NetBox. Neither operation creates the actual customer device inventory.
- The importer may retain catalogue entries removed from GitHub. Do not assume changing the repository clears the old staged records.
- Updating a YAML in GitHub and clicking Load does **not guarantee that previously imported NetBox Device Types, interface templates, or existing device interfaces are updated**. Inspect the importer's update behavior and perform any required controlled corrections.
- Back up NetBox data before destructive bulk operations. Keep GitHub tokens and credentials out of YAML, README files, commits, screenshots and shared terminal output; rotate accidentally exposed credentials immediately.

## 10. Amptec user guide

**Find a type:** search by manufacturer, model, SKU and connector variant. Marketing names can conceal significant physical differences.

**Request a new type:** provide the exact manufacturer, model, order code, official datasheet/manual, and preferably clear photos of the front, rear and model label. State whether PoE, power or optical connector variants exist.

**Report a correction:** link to the YAML, explain which component is wrong or missing, and provide a source or photo. Flag uncertain details; do not replace them with guesses.

**Create a real device in NetBox:** select the correct Device Type, then enter the actual serial number, site, location, rack position, IPs, cables, VLANs and fitted hardware on the **device**. Never insert that site-specific information into the reusable GitHub type.

**When using optical cables:** check the physical cable/connectors manually; the interface type alone does not guarantee Fiberfox/opticalCON compatibility.

## 11. Planned rollout

1. **Luminex:** first GigaCore 20t and 10t hardware variants; next **GigaCore 10i**, followed by other current/Gen2 switches.
2. Add earlier GigaCore generations and discontinued products using archived documentation.
3. Continue with **LumiNode**, **LumiCore**, **LumiSplit** and relevant Luminex power accessories (including RPSU 5580).
4. Expand to other AV manufacturers using exactly the same modelling and verification rules.
5. Later assess NetBox-to-Zabbix inventory synchronization; keep monitoring concerns out of Device Type YAML.

## References

- [Amptec AV-library](https://github.com/amptec-support/AV-library)
- [NetBox documentation](https://netboxlabs.com/docs/netbox/)
- [NetBox community Device Type Library](https://github.com/netbox-community/devicetype-library)
- [Installed Device and Module Type Importer project](https://github.com/andy-shady-org/netbox-device-module-type-importer)
- [Luminex](https://www.luminex.be/)

---

**Core rule:** describe what is physically present, in the appropriate NetBox field, with traceable evidence and without redundant descriptions. Keep type definitions distinct from live customer inventory and monitoring.
