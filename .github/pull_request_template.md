## 📋 Description

Briefly describe the changes introduced by this Pull Request.

---

## ✅ Checklist & Parity Verification

- [ ] Changes in `packages/*.yaml` and `d1-livello-pellet.yaml` are synchronized 1:1 in `d1-livello-pellet_standalone.yaml`
- [ ] `yamllint -c .yamllint.yml .` passes with zero errors
- [ ] `esphome config d1-livello-pellet.yaml` compiles cleanly
- [ ] `esphome config d1-livello-pellet_standalone.yaml` compiles cleanly
- [ ] No private credentials (`secrets.yaml`) are included in the commit
- [ ] Documentation (`README.md`, `docs/`) has been updated if applicable

