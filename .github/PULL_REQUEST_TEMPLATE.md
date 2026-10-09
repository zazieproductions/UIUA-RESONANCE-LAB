## Summary

<!-- One-paragraph description of what this PR changes and why. -->

## Type of Change

- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature that changes existing behavior)
- [ ] New preset
- [ ] New glyph
- [ ] Performance improvement
- [ ] Accessibility improvement
- [ ] Documentation update
- [ ] Chore / tooling / refactor

## Related Issue(s)

<!-- Closes #xxx -->

## Testing

Verified the following (check all that apply):
- [ ] Manual checklist in `docs/TESTING.md` passes for the affected areas
- [ ] Tested on Chrome/Firefox/Safari (list which)
- [ ] No console errors/warnings introduced
- [ ] FPS remains at budget (60 @ 64 resolution, ≥48 @ 128)
- [ ] All slider interactions and preset switches work
- [ ] Audio toggles cleanly without clipping or crashes

For preset additions:
- [ ] `render()` outputs are bounded approximately in [-1, 1]
- [ ] `synth()` outputs are bounded in [-1, 1]
- [ ] Tonal presets loop without audible click at the 2-second seam
- [ ] Added entry to `docs/PRESETS.md` with equations

For glyph additions:
- [ ] Verified against uiua.org/docs (real primitive, not invented)
- [ ] Added entry to `docs/GLYPHS.md`

## Screenshots / Recordings

<!-- For visual or audio changes, include media here. -->

## Performance Impact

<!-- Note any benchmark deltas vs. main (frame time, heap size, etc.). -->

## Documentation

- [ ] Updated `docs/` for any architectural or API changes
- [ ] Updated `CHANGELOG.md` (for user-facing changes)

## Additional Notes

<!-- Anything reviewers should know. -->
