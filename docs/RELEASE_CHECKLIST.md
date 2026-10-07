# Release checklist (before making the repository public / citing it in the paper)

- [ ] Replace `<YOUR-USERNAME>/<YOUR-REPOSITORY>` in `CITATION.cff` and README, and insert the final URL in Section 9 of the paper.
- [ ] Run notebook 03 and commit `splits/lavdf_splits.json`.
- [ ] Re-run `python scripts/stats_tests.py` and `python scripts/make_figures.py`; confirm the numbers match the paper.
- [ ] Search the repository for personal paths, e-mail addresses or tokens (`grep -rn "/home/" .`).
- [ ] Confirm no data, features (`.npz`) or checkpoints (`.pt`) are committed (`git ls-files | grep -E "npz|pt$|mp4"`).
- [ ] Check the licences of both datasets before sharing anything derived from them (features, checkpoints).
- [ ] Tag a release (`v1.0`) and archive it on Zenodo to obtain a DOI; cite the DOI in the paper.
- [ ] If the journal uses double-blind review, publish through an anonymised mirror until acceptance.
