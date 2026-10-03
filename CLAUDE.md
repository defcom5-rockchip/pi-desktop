# Agent instructions

This is the PRODUCT repository for Pi Desktop: user-facing docs and image
releases only. The build system lives in defcom5-rockchip/pi-desktop-recipe
(Armbian userpatches; the retired 2.0.x line used defcom5-rockchip/ubuntu-rockchip);
the video driver in defcom5-rockchip/rockchip-vaapi (read its AGENTS.md
before touching anything video-related).

## Rules
1. **Verified claims only.** Every capability statement in README/KNOWN-ISSUES
   must trace to a hardware-verified result. If it wasn't eyeballed on a real
   Orange Pi 5B, it doesn't ship as a claim. When reality and docs disagree,
   docs change the same day (see the v1.0 browser-decode correction for the
   house precedent: we publish the correction AND what we got wrong).
2. **KNOWN-ISSUES is a feature.** Never delete an issue to tidy the file —
   issues leave via a "Fixed in vX" entry or stay. Write severity honestly.
3. **House voice:** plain, direct, a little dry. "We'd rather tell you what's
   rough than let you find out." No marketing superlatives.
4. **Releases are human-triggered.** Draft notes freely; the maintainer cuts
   the release. Image assets must fit GitHub's 2 GiB cap (xz -9e as needed)
   and ship with a .sha256.
5. Commit as defcom5-rockchip; Claude co-author trailer welcome.
