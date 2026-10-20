# Amptec AV Device Type Library — v0.1

NetBox-compatible YAML templates for professional AV equipment.

## First batch

- **Netgear**: M4250-26G4XF-PoE+ (GSM4230P) — 30 network interfaces, console and power; based on community definition.
- **d&b audiotechnik**: D80 — 2 RU and two 100 Mbit Ethernet interfaces, as stated by manufacturer. Other audio connectors NOT modelled as IP interfaces.
- **QSC**: Core 110f — 1 RU, LAN A and B at 1 Gbit/s; audio I/O documented but deliberately not represented as Ethernet ports.
- **DirectOut**: PRODIGY.MP — draft in `drafts/`, eight module bays verified; verify chassis size before moving to `device-types/`. It is intentionally **excluded from GUI import** for now.

## GUI import using the plugin you already installed

1. Create a GitHub repository named `amptec-av-device-library` and upload the contents of this folder **to repository root** (not one directory deeper).
2. The plugin uses GitHub GraphQL, and expects `device-types/<Manufacturer>/<Model>.yaml`. Its `repo_owner` must match your GitHub account/org.
3. On `ampnetbox01`, edit `/opt/netbox/netbox/netbox/configuration.py`: in the EXISTING `PLUGINS_CONFIG['netbox_device_module_type_importer']`, set `repo_owner` and `repo` to your repository, keeping the `github_token`. Do not create duplicate `PLUGINS_CONFIG` blocks.
4. Verify with `/opt/netbox/venv/bin/python /opt/netbox/netbox/manage.py check`; then `sudo systemctl restart netbox netbox-rq`.
5. In NetBox's Device & Module Type Importer GUI, select one manufacturer/model, import and verify generated component templates under Devices > Device Types.
6. This importer selects ONE repository at a time. To return to community models, change `repo_owner=netbox-community`, `repo=devicetype-library` and restart services.

**Caution:** plugin 0.0.3 is not confirmed compatible with NetBox 4.7.3 end-to-end, though initial `manage.py check` succeeded. Backup PostgreSQL before any import.

## Scope and quality controls

- YAML fields match the public NetBox Device Type Library format.
- Audio XLR, NL4, AES, GPIO etc. are NOT Ethernet interfaces; precise physical patching requires custom component conventions (or a dedicated plugin) beyond these minimal templates.
- Verify chassis variations, network connector labels, and power specs against the actual hardware before deployment.
- DirectOut is a draft because chassis rack U and fixed ports need further checking.
- Use the NetBox GUI to create actual devices only after verifying imported device types.

Sources: 
- https://github.com/netbox-community/devicetype-library
- https://github.com/andy-shady-org/netbox-device-module-type-importer
- https://www.dbaudio.com/global/en/products/amplifiers/d80/
- https://help.qsys.com/q-sys_9.4/Content/Hardware/Processing/Core_110f.htm
- https://www.directout.eu/product/prodigy-mp/
