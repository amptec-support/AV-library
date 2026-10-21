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
- Build the library **manufacturer by manufacturer**, then **series by series**. Luminex and NETGEAR M4350 are the first families; apply the same standards to every future manufacturer.
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
│   ├── Netgear/
│   └── ...
├── scripts/                      # Optional maintained NetBox utility scripts
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

Do not invent Front/Rear Ports to represent the front/rear location of an active interface. **Agreed AV exception:** to document the physical DMX cable path in the current NetBox deployment, LumiNode XLR5 and RJ45 DMX connectors may be represented as `interfaces` with `type: other`. These are **not Ethernet** and must not be mistaken for SNMP-monitored network ports. Record the signal/physical connector explicitly (e.g. `Front XLR5 Female`, `Rear RJ45 DMX`). Apply this pattern to other non-Ethernet AV connections only after verifying that native NetBox components do not model the intended cable path more faithfully.

## 5. Interface naming and descriptions

- Prefer the **actual port numbering printed on the hardware** (`Port 1`, `Port 9`, etc.) where confirmed.
- Record actual network medium/speed in the `type` field and use `poe_mode` and `poe_type` where applicable.
- Descriptions must be **short, factual and non-redundant**: **`Front` or `Rear`** (do not alternate with `Back`), external connector family and relevant optical medium only when not represented elsewhere. **Populate `description` on every interface/interface template, including module interfaces.**
- Do not repeat port numbers, RJ45 compatibility of EtherCON, generic SFP explanations, link duplex/TX/RX facts, or the number of fibers normally used by a duplex link.
- Do not put PoE capability in `description` when it can be represented by the dedicated PoE fields.

Examples of acceptable descriptions:

| Component | `description` |
| --- | --- |
| Front copper interface | `Front EtherCON` |
| Rear copper interface | `Rear EtherCON` or `Rear RJ45` as fitted |
| Front SFP+ interface | `Front SFP+` |
| Front SFP28 interface | `Front SFP28` |
| APM414SD | `Front opticalCON QUAD MMF (connector 1)` |
| DMX connector | `Front XLR5 Female` |
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

### Combo interfaces and unique names

A **combo port** is one logical switch port exposed on **two different physical connectors** (e.g. RJ45 and SFP+). For physical cabling, describe both connectors as separate interface objects. They must have **different NetBox `name` values**, because NetBox requires unique interface names on a device. Keep the manufacturer's shared port number in the label or name and explicitly describe that the two interfaces are mutually exclusive. **NetBox will not enforce the combo-port exclusivity** for you. Do not accidentally count these as two independently usable switch links.

Example of the agreed M4350-24F4X principle (exact numbering is illustrative; use actual hardware labels):

```yaml
interfaces:
  - name: Port 25 - RJ45
    label: '25'
    type: 10gbase-t
    description: Front RJ45 (combo with Port 25 - SFP+)
  - name: Port 25 - SFP+
    label: 25F
    type: 10gbase-x-sfpp
    description: Front SFP+ (combo with Port 25 - RJ45)
```

For **module interfaces**, `name` must likewise be unique relative to every other interface on the device. On the M4350-16M4V, the tested APM414-series naming is `Port I1`–`Port I4` rather than `Port 1`–`Port 4`, which would collide with fixed switch ports. Test by installing the module on an actual **test device**, not solely by inspecting its Module Type.

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

### Replaceable power supplies and interface modules

A fixed connector/input on the chassis is a `power-port`; a **truly removable slide-in PSU** is a `module-type` installed in a `module-bay`. Different counts or combinations of replaceable PSUs **do not** justify separate chassis Device Types. The same applies to removable interface expansion cards. The device instance records which specific PSU or card is installed.

- The installed PSU module should own its **actual AC power input**. For multiple PSU slots, use a module-bay position (e.g. `1`, `2`) and a templated input name such as `Power In {module}` to avoid duplicate power-port names after installation. Test the result with two populated bays.
- External NETGEAR APS254W adapters for the **M4350-8M2V** are not chassis plug-in PSU modules; model the switch's DC inputs instead of inventing internal PSU bays.
- The **M4350-16M4V** has fixed PowerCON inputs and a removable **APM414 interface expansion card**; do not miscategorize those PowerCON connectors as slide-in PSU modules. Confirm the precise PowerCON physical variant from manufacturer documentation before claiming it verified.
- M4350 APM414 interface cards include **APM414V** (SFP28), **APM414C** (copper), **APM414SD** (MMF opticalCON QUAD) and **APM414LD** (SMF opticalCON QUAD). For SD/LD, distinguish **two physical QUAD connectors** from **four independent optical links**; map the exact port/connector relationship from official drawings where available.
- NETGEAR M4350 PSU families have different hardware compatibilities. A shared manufacturer is **not proof** that a PSU fits a chassis. Use only manufacturer-confirmed combinations and document them in the relevant module-bay description/research.
- The W number in an APS product name typically refers to PSU output rating, **not** necessarily the whole device's NetBox `maximum_draw` (AC input consumption). Verify connector types (including C14 versus C20) and input ratings individually.
- Where NetBox **Module Bay Types** are supported, they can classify compatible bays/modules; check how the installed importer actually handles them. **Do not assume that merely adding a Bay Type or a description enforces every incompatibility**, especially for unclassified modules. Validate allowed and disallowed installations on a test device.

#### Module Type YAML example

```yaml
manufacturer: Netgear
model: M4350 APM414V 4x25G SFP28
part_number: APM414V
interfaces:
  - name: Port I1
    type: 25gbase-x-sfp28
    description: Front SFP28
  - name: Port I2
    type: 25gbase-x-sfp28
    description: Front SFP28
  - name: Port I3
    type: 25gbase-x-sfp28
    description: Front SFP28
  - name: Port I4
    type: 25gbase-x-sfp28
    description: Front SFP28
```

Every removable part belongs in `module-types/<Manufacturer>/`, not as an extra device type for each possible chassis/module combination.

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
- **Device Type `description` must be at most 200 characters** (NetBox form validation). Keep it concise and factual; put extensive specifications, source references and research notes in `research/` instead. This limit applies to the **Device Type description field**; do not assume every component description has the same limit.
- No duplicate or unnecessarily verbose interface/component descriptions.
- YAML syntax and NetBox field/choice compatibility checked. Before publishing, automatically check the length of every Device Type top-level `description` (e.g. `len(description) <= 200`), including files generated in bulk; otherwise the importer rejects the record.
- Test **Load → Import → inspect component templates → create a test device → install modules → inspect resulting components**. A successful import does not by itself prove the hardware definition is complete or correct.
- Confirm that `description` is present both on the **template** and (after the synchronization step below) on the **actual interface**. These are separate records.
- Verify that module interfaces install without name collisions and that power-ports have unique names after multiple PSUs are fitted.

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
8. Run the **interface description sync** (section 10) on the NetBox instance after new devices/modules are installed, first in dry-run mode.
9. Update verification status and research sources. Retire outdated drafts and avoid duplicate Device Types.

**Installed importer:** `netbox-device-module-type-importer` v0.0.3 with NetBox 4.7.3 during initial setup. The plugin contains **hard-coded `master` GitHub references**. Keep `master` available; review compatibility if upgrading the plugin/NetBox. It reads GitHub through GraphQL using the configured `repo_owner`, `repo` and a server-side credential.

**Important operational details:**

- **Load** refreshes the plugin's **staging catalogue**; **Import** creates the selected type definitions in NetBox. Neither operation creates the actual customer device inventory.
- The importer may retain catalogue entries removed from GitHub. Do not assume changing the repository clears the old staged records.
- Updating a YAML in GitHub and clicking Load does **not guarantee that previously imported NetBox Device Types, interface templates, or existing device interfaces are updated**. Inspect the importer's update behavior and perform any required controlled corrections.
- Back up NetBox data before destructive bulk operations. Keep GitHub tokens and credentials out of YAML, README files, commits, screenshots and shared terminal output; rotate accidentally exposed credentials immediately.

## 10. Interface descriptions: synchronization script (required operational step)

**Observed on Amptec NetBox 4.7.3:** `InterfaceTemplate.description` is correctly loaded for both Device Types and Module Types, but an interface **created on an actual device** (including a removable module interface) **does not automatically inherit that description**. A correctly filled template does not guarantee a description on the device instance. Reinstalling a module does not solve this behavior.

We have tested a local Python/Django script on the NetBox server at:

```text
/opt/netbox/sync_interface_descriptions.py
```

**What the script does:**

1. Reads all Interface Templates with nonempty `description` values.
2. Finds actual device interfaces where `description` is empty.
3. For fixed interfaces, matches by the **device's Device Type + exact interface name**.
4. For fitted module interfaces, matches by the **installed Module Type + exact interface name**.
5. Copies the template description only when a match is found; **existing nonempty descriptions remain untouched**.
6. Prints totals and sample changes. The current script is **manual**, not automatically triggered by new devices/modules.

We verified the process in the current environment: the dry-run identified **21 interfaces**, all **21 matched**, and the subsequent apply operation populated their descriptions successfully. This confirms the mechanism on the tested hardware; it does not automatically validate future hardware YAML.

### How to run (NetBox host `ampnetbox01`)

**1. Dry-run — mandatory first**

Ensure the script contains:

```python
APPLY = False
```

Execute:

```bash
cd /opt/netbox
/opt/netbox/venv/bin/python netbox/manage.py shell \
  < /opt/netbox/sync_interface_descriptions.py
```

Inspect `Interfaces checked`, `Already filled`, `Missing matching template`, `Descriptions to update` and the examples. `None` in the example's device-name column may mean the test device has no name; inspect the record if unexpectedly missing. **Do not proceed if unexpected components or descriptions appear.**

**2. Apply — only after the preview is correct**

Change the existing script temporarily to:

```python
APPLY = True
```

Then run the **same command**. The current implementation uses a database transaction for its writes and avoids overwriting any description that was populated when the script queried the interfaces. **Do not run simultaneous interface edits during apply**; obtain a backup appropriate to your change scope before mass updates.

**3. Verify and reset**

- Refresh the actual **Device → Interfaces** page in NetBox; inspect at least one built-in port and one installed module port.
- Set `APPLY = False` again immediately after the successful run to prevent accidental writes on subsequent executions.
- After importing definitions, **run the sync for any new devices/modules**; updates to a Device Type's YAML or template do not themselves update already deployed devices.

**Scope/limitations:** the script matches **exact interface names** and only fills *empty* values. It does **not** change existing descriptions when the library standard changes, does not rename ports, and does not verify actual connector presence or enforce module compatibility. Keep manual exceptions on existing interfaces intact. If a template is missing or has a different name, inspect YAML and component templates; do not force a broad rename/update. A future approved automation (e.g. an event-driven NetBox integration) can eliminate the manual step, but **is not implemented today**.

**Source control:** maintain a reviewed copy of the script in `scripts/` if it is to become a shared team tool. Currently the tested copy lives **on the NetBox host**, not automatically in GitHub; the README alone does not install it.

## 11. Amptec user guide

**Find a type:** search by manufacturer, model, SKU and connector variant. Marketing names can conceal significant physical differences.

**Request a new type:** provide the exact manufacturer, model, order code, official datasheet/manual, and preferably clear photos of the front, rear and model label. State whether PoE, power or optical connector variants exist.

**Report a correction:** link to the YAML, explain which component is wrong or missing, and provide a source or photo. Flag uncertain details; do not replace them with guesses.

**Create a real device in NetBox:** select the correct Device Type, then enter the actual serial number, site, location, rack position, IPs, cables, VLANs and fitted hardware on the **device**. Never insert that site-specific information into the reusable GitHub type.

**When using optical cables:** check the physical cable/connectors manually; the interface type alone does not guarantee Fiberfox/opticalCON compatibility.

## 12. Planned rollout

1. **Luminex:** continue checking GigaCore Gen2 variants, LumiCore and selected LumiNode models against official hardware evidence; add other products (including older generations) only where needed and sourced.
2. **NETGEAR M4350:** complete batch 1 hardware verification (seven switch models, PSU Module Types and APM414 cards); continue to further batches only after quality checks.
3. Expand to other AV manufacturers using exactly the same physical modelling, description and verification rules.
4. Assess NetBox-to-Zabbix inventory synchronization later; keep monitoring concerns out of Device Type YAML.

## References

- [Amptec AV-library](https://github.com/amptec-support/AV-library)
- [NetBox documentation](https://netboxlabs.com/docs/netbox/)
- [NetBox community Device Type Library](https://github.com/netbox-community/devicetype-library)
- [Installed Device and Module Type Importer project](https://github.com/andy-shady-org/netbox-device-module-type-importer)
- [Luminex](https://www.luminex.be/)
- [NETGEAR](https://www.netgear.com/business/wired/switches/fully-managed/m4350/)

---

**Core rule:** describe what is physically present, in the appropriate NetBox field, with traceable evidence and without redundant descriptions. Use `Front`/`Rear` descriptions consistently, validate modules on real test devices, and run description synchronization after creating devices. Keep type definitions distinct from live customer inventory and monitoring.
