# Data Directory

- `raw/`: private source images; ignored by Git.
- `processed/`: private derived images; ignored by Git.
- `manifests/`: versioned metadata and hashes; safe examples may be committed.
- `eval_refs/`: held-out reference images; ignored by Git.

Use repository-relative paths in manifests. Never place private image data in
Git, including through Git LFS, without explicit project approval.
