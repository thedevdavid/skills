# Ship checklist

Read when verifying a draft or unsure a packaging rule applies.

## Description

- [ ] Third person; what + when; concrete trigger terms; ≤1024 chars
- [ ] Would fire on real user phrases; would not steal unrelated tasks
- [ ] Optional Skip-when if false positives are likely

## Package

- [ ] `name` matches folder; soft metadata present when shipping
- [ ] `SKILL.md` lean; gotchas inline; refs one-level-deep with “read when …”
- [ ] No tests/fixtures under `skills/<name>/`
- [ ] Scripts only if reinvented; portable deps documented
- [ ] Body host-agnostic; install tips only in README / optional `agents/`
- [ ] Root README catalog updated
- [ ] Density comparable to `skills/prune-tests/` (gates, anti-filler)
