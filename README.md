# Amptec AV Device & Module Library

A curated, version-controlled library of professional audiovisual (AV) equipment definitions for use with **NetBox**.

**Repository:** [amptec-support/AV-library](https://github.com/amptec-support/AV-library)  
**Maintainer:** Amptec  
**Primary consumers:** Amptec engineering, project management and technical operations

## Purpose

This repository provides reusable **device type** and **module type** definitions for AV infrastructure. Its purpose is to make the physical characteristics of equipment consistent across projects: model identity, interfaces, connectors, rack dimensions, power connections and modular components.

It is **not** a customer asset database. Actual devices, serial numbers, locations, cabling, IP addresses, VLANs and site-specific configuration belong in **NetBox**, not in this Git repository.

In the longer term, NetBox may serve as a source of intended inventory and connectivity information for integration with **Zabbix**. Zabbix remains responsible for live monitoring, measurements and alerts. No Zabbix integration is assumed to be active today.

## Scope

- Professional AV equipment across manufacturers, including both current and discontinued products.
- All Luminex product families, including legacy models, are the initial documentation priority.
- One definition per **distinct physical hardware variant** when specifications, fitted connectors, power system or hardware SKU differ meaningfully. Firmware versions or software licences alone do not create new device types.
- Only documented technical facts are published. Missing or uncertain information must not be guessed.

The goal is detailed, accurate definitions, not the largest possible number of YAML files.

## Repository layout

```text
AV-library/
├── device-types/
│   ├── Luminex/
│   ├── Netgear/
│   ├── QSC/
│   └── ...
├── module-types/
│   ├── Luminex/
│   └── ...
├── research/
│   └── Luminex/
│       ├── inventory.csv
│       └── sources.md
└── README.md
```

**Device and module YAML files** must follow the schemas and naming conventions supported by the installed NetBox importer and relevant NetBox version. The `research/` directory is for tracking source material and work in progress; it is not an import source.

## What to capture

For each device type, record as much of the following as is supported by documentation and the NetBox schema:

| Category | Examples |
| --- | --- |
| Identity | Manufacturer, exact model, part number / SKU, hardware variant, description |
| Form factor | Rack units, full/half width, full-depth flag, weight and dimensions where supported |
| Network | Copper Ethernet, SFP/SFP+/QSFP, management and console ports, exact front-panel labels |
| AV I/O | DMX/RDM, audio, video and control connectors, where the data model supports them faithfully |
| Power | AC/DC inputs, redundant power feeds, PoE capabilities, power supply modules |
| Modularity | Module bays and supported module types |
| Documentation | Manufacturer source URL, document revision/date and verification notes in research files |

Use the **manufacturer's actual port names** when possible, to help engineers match NetBox diagrams to the physical chassis. An SFP cage is a device interface; the particular installed transceiver is part of the actual device inventory, not a universally fixed characteristic of the device type.

Do not represent an unsupported AV connector as an Ethernet port or misleading connector type. Capture its specification in research notes and define an agreed modelling approach before publishing it. Physical connectors and logical protocol capabilities are not automatically the same thing.

## Data quality and verification

Each planned model has a status in the research inventory:

1. **Research** — identified, but documentation or specifications are incomplete.
2. **Ready for validation** — YAML drafted and each technical claim can be traced to a source.
3. **Verified** — schema checked, reviewed against authoritative documentation, and tested in NetBox.

Preferred sources, in order:

1. Manufacturer datasheets, manuals, hardware documentation and official product pages.
2. Manufacturer archives and official support material for discontinued products.
3. Other sources only when clearly identified as secondary and independently checked.

Keep source links and any unresolved questions in `research/<Manufacturer>/sources.md` or the inventory. **Never invent** a port count, speed, connector, rack size, SKU or power specification.

## Naming and hardware variants

- Use one YAML per distinctly orderable **hardware variant** if it has meaningful physical differences.
- Keep the manufacturer's model name accurate; use a unique, stable NetBox slug for each variant.
- Record an exact `part_number` only when the manufacturer confirms it.
- Do not create duplicate variants solely for firmware, optional software features or customer-specific configuration.
- File names should be descriptive, consistent and stable (for example, `GigaCore-20t-10G-P500.yaml`); names are examples, not a claim that a particular SKU is verified.
- Match the manufacturer's actual interface and component labels, including numbering and capitalization where practical.

## Contribution workflow

We work **manufacturer by manufacturer** and validate **series by series**:

1. **Inventory:** identify product families, older devices and documented hardware variants. Record sources and gaps.
2. **Draft:** create YAML from confirmed specifications and the supported schema.
3. **Review:** verify names, part numbers, port types, physical labels, power interfaces and modularity.
4. **Validate:** check YAML syntax and schema, then load a small sample through the NetBox Device and Module Type Importer.
5. **Inspect in NetBox:** confirm the resulting device type, interfaces, power ports and module bays are correct. Check for collisions with existing records.
6. **Publish:** merge the reviewed files into `master`, then refresh the importer catalogue and selectively import the approved type definitions.

Use a feature branch and pull request for substantial changes. Avoid committing incomplete device definitions to `master` as though they were verified. When correcting an existing definition, describe the impact on NetBox in the pull request; changes to a device type do not necessarily retroactively update deployed devices.

## Using the library in NetBox

The Amptec NetBox installation uses the **Device and Module Type Importer** and is configured to read:

- GitHub owner: `amptec-support`
- Repository: `AV-library`
- Branch: `master`

The currently installed importer version, **0.0.3**, contains hard-coded `master` references. Keep the repository's `master` branch available until compatibility with another importer version has been tested. Do not assume that merely changing the default GitHub branch will be safe.

**Load** refreshes the plugin's catalogue of GitHub definitions; **Import** creates or updates NetBox records as supported by the plugin. Loading the catalogue does **not** automatically create the corresponding live devices at customer sites.

When changing repository sources, be aware that this importer version does not automatically remove stale catalogue entries absent from the new repository. Do not bulk-delete or overwrite imported inventory without a backup and review.

Never commit GitHub tokens, NetBox credentials, private customer information, or other secrets. Use restricted credentials and appropriate server-side secret management.

## Luminex: first implementation

Luminex is the first manufacturer to be developed comprehensively:

1. Inventory **all product families**, including legacy equipment and separate hardware variants.
2. Prioritize current and Gen2 GigaCore switches for initial validation; then cover earlier generations.
3. Use a well-documented GigaCore family, such as **GigaCore 20t**, for the first reviewed sample set.
4. Extend the same workflow to **LumiNode, LumiCore and LumiSplit**, paying special attention to faithful modelling of DMX/RDM and other non-Ethernet connectors.
5. Keep variants in `Research` when primary documentation is missing; publish only verified records.

The research inventory is the definitive worklist. Any model list in planning notes is preliminary until checked against manufacturer documentation.

## Guidelines for Amptec users

**Before requesting a new model:** search the repository and NetBox for the same manufacturer, exact model and part number. A similar marketing name may conceal a different physical variant.

**When requesting an addition or correction:** provide the manufacturer, full model name, hardware SKU, link to the official datasheet/manual, photos of front/rear labels if available, and a clear statement of what is missing or incorrect.

**When using an imported type:** confirm it matches the specific hardware in front of you. Enter customer-specific information only on the NetBox device and related inventory objects.

**When uncertain:** flag the item for research instead of approximating. Review is more valuable than a fast but incorrect import.

## Maintenance and governance

- Amptec engineering owns technical accuracy and changes to the library.
- Repository changes should be reviewed before merging to `master`.
- Record notable changes in pull requests and preserve evidence for corrections.
- Review importer/NetBox compatibility before software upgrades.
- Verify changes against a test device type before applying them to production inventory.
- Back up NetBox's database before destructive bulk actions; a Git backup alone does not protect the NetBox inventory.

## Useful resources

- [Amptec AV-library](https://github.com/amptec-support/AV-library)
- [NetBox documentation](https://netboxlabs.com/docs/netbox/)
- [Community Device Type Library](https://github.com/netbox-community/devicetype-library)
- [Device and Module Type Importer](https://github.com/andy-shady-org/netbox-device-module-type-importer)
- [Luminex](https://www.luminex.be/)

---

**Principle:** model the physical hardware accurately, preserve traceable sources, and keep reusable type definitions separate from customer-specific inventory and live monitoring.
