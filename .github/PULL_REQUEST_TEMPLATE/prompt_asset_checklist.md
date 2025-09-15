# Prompt Asset PR Checklist

- [ ] Camera matches token (e.g., `CAMERA_TON_ISO`); horizon/verticals sane
- [ ] Scale check (character proxy fits doors/steps/props)
- [ ] Lighting token respected (shadow dir/length coherent)
- [ ] Alpha edges clean (no dark halo), background plates free of baked text
- [ ] Filename pattern correct and `.prompt.json` included
- [ ] Unity import: sRGB on, compression appropriate, mipmaps off for plates used as UI
- [ ] LFS pointers verified (`git lfs ls-files` shows the asset)
