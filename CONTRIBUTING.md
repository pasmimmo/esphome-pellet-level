# Contributing to D1 Pellet Level Monitor

Thank you for your interest in contributing to **D1 Pellet Level Monitor**!

---

## 🛠️ Dual-Configuration Architecture Rule

This repository maintains two synchronized configuration formats:
1. **Modular Configuration**: `d1-livello-pellet.yaml` + `packages/*.yaml`
2. **Standalone Monolithic Configuration**: `d1-livello-pellet_standalone.yaml`

> [!IMPORTANT]
> **Parity Requirement:** Any change made to `packages/*.yaml` or `d1-livello-pellet.yaml` **must** also be reflected identically in `d1-livello-pellet_standalone.yaml` so both configurations pass CI validation.

---

## 🧪 Local Validation Before Submitting a PR

1. Copy `secrets.yaml.example` to `secrets.yaml`.
2. Lint all YAML files:
   ```bash
   yamllint -c .yamllint.yml .
   ```
3. Validate both ESPHome configurations:
   ```bash
   esphome config d1-livello-pellet.yaml
   esphome config d1-livello-pellet_standalone.yaml
   ```

---

## 📝 Commit Message Conventions

We follow [Conventional Commits](https://www.conventionalcommits.org/):
* `feat:` new sensor, filter, or Home Assistant capability
* `fix:` bug fix in lambda math, I2C config, or automation
* `docs:` documentation or ADR updates
* `ci:` GitHub Actions or linting adjustments

