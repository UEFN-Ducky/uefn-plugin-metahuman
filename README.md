# MetaHuman

MetaHuman for UEFN — create/edit in Creator, Mesh to MetaHuman, UEFN Export assemble, NPC spawn. Bundles the metahuman skill + thin MCP probe/inspect/assemble tools.

Desktop plugin for [UEFN-Ducky](https://github.com/UEFN-Ducky/UEFN-Ducky) (`metahuman`).
Install or update from **Settings → Store** in the app — do not install from a zip by hand.

## Build

```bash
py scripts/build_zip.py
```

Writes `deploy/metahuman-*.ducky-plugin.zip` (scripts/ and deploy/ are not packed).
