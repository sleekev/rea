# CLAUDE.md

## 🔴 Sleek EV policy — REA use counts as sending code outside the company

Any REA analysis (CLI or MCP) puts its results into an AI agent's context, which is sent to the model provider. Sleek EV treats every REA run as **code leaving the company**.

- **NEVER run REA on Sleek EV source code** or anything built from it: our repositories, builds, binaries, mobile APKs/IPAs, web bundles, or firmware we develop.
- For our own code, use normal local repository tools only.
- REA is allowed only on third-party artifacts we are authorized to inspect (vendor firmware, vendor apps, legacy binaries without source), after checking the vendor's license or contract.
- If a target might be ours, or its origin is unclear: **stop and ask the owner** before calling any REA tool.
- Never commit analysis results or internal details to this public repository.

ห้ามใช้ REA กับ source code ของบริษัท และของที่ build จากโค้ดของเรา (repo, APK, firmware, binary, web bundle) เพราะถือเป็นการส่งโค้ดออกนอกบริษัท · ใช้ได้เฉพาะของ vendor ที่เรามีสิทธิ์ · ไม่แน่ใจ = หยุดแล้วถามก่อน

See [SLEEK-USAGE.md](SLEEK-USAGE.md) for install and environment rules.

@AGENTS.md
