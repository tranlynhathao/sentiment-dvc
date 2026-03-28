# sentiment-dvc

## Overview
This project is a small DVC lab that versions a sentiment analysis dataset stored at `data/raw/sentiment.csv`.
Git tracks the project code and the DVC pointer file, while DVC tracks the dataset contents.

Current local dataset versions:
- `dataset-v1` -> 4 labeled rows.
- `dataset-v2` -> 8 labeled rows.

Current local Git history:
- `feaec47` `Initialize Git and DVC`
- `bb3654c` `Track sentiment dataset v1`
- `e1bee55` `Track sentiment dataset v2`

## Local Setup
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install "dvc[s3]"
```

If you are recreating the repository from scratch instead of cloning it:
```bash
git init -b main
dvc init
```

## DVC Workflow Used
```bash
# Version 1
dvc add data/raw/sentiment.csv
git add data/raw/.gitignore data/raw/sentiment.csv.dvc
git commit -m "Track sentiment dataset v1"

# Version 2
# edit data/raw/sentiment.csv
dvc add data/raw/sentiment.csv
git add data/raw/sentiment.csv.dvc
git commit -m "Track sentiment dataset v2"
```

## GitHub Setup
Create an empty GitHub repository first, then run:
```bash
git remote add origin <GITHUB_REPO_URL>
git push -u origin main
git push origin --tags
```

## DagsHub DVC Remote Setup
Current DagsHub documentation uses an S3-compatible DVC remote, not the older direct `.dvc` URL pattern.
Recommended generic commands:
```bash
dvc remote add -d dagshub s3://dvc
dvc remote modify dagshub endpointurl <DAGSHUB_REMOTE_URL>
dvc remote modify --local dagshub access_key_id <DAGSHUB_TOKEN>
dvc remote modify --local dagshub secret_access_key <DAGSHUB_TOKEN>

git add .dvc/config
git commit -m "Configure DagsHub DVC remote"
git push origin main
dvc push -r dagshub
```

Expected placeholder values:
- `<DAGSHUB_REMOTE_URL>` -> `https://dagshub.com/<DAGSHUB_USERNAME>/<DAGSHUB_REPO>.s3`
- `<DAGSHUB_TOKEN>` -> a DagsHub access token copied from your account or repository setup flow

Important:
- Keep credentials in `.dvc/config.local` by using `--local`.
- Do not commit `.dvc/config.local`.
- If you create tags locally, push them with `git push origin --tags`.

## Retrieve a Dataset Version on Another Machine
```bash
git clone <GITHUB_REPO_URL>
cd sentiment-dvc
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install "dvc[s3]"

dvc remote modify --local dagshub access_key_id <DAGSHUB_TOKEN>
dvc remote modify --local dagshub secret_access_key <DAGSHUB_TOKEN>

git checkout dataset-v2
dvc pull -r dagshub
```

If the repository was cloned before the DVC remote was committed to `.dvc/config`, run the DagsHub remote setup commands from the previous section once.

## List and Switch Dataset Versions
List versions:
```bash
git tag --list
git log --oneline -- data/raw/sentiment.csv.dvc
```

Switch to version 1:
```bash
git checkout dataset-v1
dvc checkout
```

Switch back to version 2:
```bash
git checkout dataset-v2
dvc checkout
```

If the target data is not already present in the local DVC cache, use `dvc pull -r dagshub` after `git checkout`.

## Notes
- Git stores dataset versions indirectly by versioning `data/raw/sentiment.csv.dvc`.
- DVC stores the actual file contents in the local cache and in the configured remote storage.
- To restore a historical dataset, Git selects the correct pointer file first, then DVC materializes the matching data.

See `report.md` for the full lab report and screenshot checklist.
