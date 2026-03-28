# DVC Lab Report: Versioning a Sentiment Dataset

## 1. Objective
The objective of this lab is to create a reproducible DVC project named `sentiment-dvc`, version a small sentiment dataset, prepare the repository for GitHub and DagsHub integration, and document how to retrieve and switch dataset versions on another machine.

## 2. Environment and Setup
Local environment used for the completed local work:
- Operating context: local terminal workspace
- Git: `2.53.0`
- Python: `3.12.13`
- DVC: `3.67.0`
- Dataset path: `data/raw/sentiment.csv`

Project structure used for submission:
```text
sentiment-dvc/
├── .dvc/
├── data/
│   └── raw/
│       ├── .gitignore
│       ├── sentiment.csv
│       └── sentiment.csv.dvc
├── screenshots/
│   └── .gitkeep
├── .dvcignore
├── .gitignore
├── README.md
└── report.md
```

## 3. Work Completed Locally
Completed in this environment:
1. Created the required project folders and files.
2. Initialized Git and DVC.
3. Added dataset version 1 and committed its DVC tracking file.
4. Updated the dataset to version 2 and committed the new DVC tracking file.
5. Added Git tags for both dataset states.
6. Prepared exact commands for GitHub and DagsHub setup.
7. Wrote this report and the project README.

Not completed in this environment because external credentials or browser actions are required:
1. Creating the GitHub repository.
2. Creating or importing the DagsHub repository.
3. Generating a DagsHub token.
4. Pushing Git commits to GitHub.
5. Pushing DVC data objects to DagsHub.

## 4. Procedure
### 4.1 Create the Project and Dataset
The project was created with the required structure, and the initial CSV file was saved as:

```csv
text,label
"I love this movie",positive
"This was terrible",negative
"Pretty good overall",positive
"Waste of time",negative
```

### 4.2 Initialize Git and DVC
Commands used locally:
```bash
git init -b main
dvc init
git add .dvc .dvcignore .gitignore screenshots/.gitkeep
git commit -m "Initialize Git and DVC"
```

Actual initial commit created locally:
```text
feaec47 Initialize Git and DVC
```

### 4.3 Track Dataset Version 1
Commands used:
```bash
dvc add data/raw/sentiment.csv
git add data/raw/.gitignore data/raw/sentiment.csv.dvc
git commit -m "Track sentiment dataset v1"
```

Actual version 1 commit created locally:
```text
bb3654c Track sentiment dataset v1
```

Version 1 pointer summary:
```yaml
outs:
- md5: 7bda551b49fe980c83ae5d85b146f1cc
  size: 125
  hash: md5
  path: sentiment.csv
```

### 4.4 Update the Dataset to Version 2
Dataset version 2 was created by adding four more labeled examples:

```csv
"Absolutely fantastic acting",positive
"The plot was engaging",positive
"I would not recommend this",negative
"Too long and boring",negative
```

Commands used:
```bash
dvc add data/raw/sentiment.csv
git add data/raw/sentiment.csv.dvc
git commit -m "Track sentiment dataset v2"
```

Actual version 2 commit created locally:
```text
e1bee55 Track sentiment dataset v2
```

Version 2 pointer summary:
```yaml
outs:
- md5: 8e3cd116c6b0c1e34e6108456c0b8107
  size: 266
  hash: md5
  path: sentiment.csv
```

### 4.5 Add Helpful Version Tags
The following local tags were created to simplify version retrieval:
```bash
git tag dataset-v1 bb3654c
git tag dataset-v2 e1bee55
```

## 5. Relation Between Git History and DVC Dataset Versions
This is the key concept of the lab.

Git does not store the large dataset file directly. Instead, Git versions the lightweight pointer file `data/raw/sentiment.csv.dvc`. That pointer file records the content hash and size of the dataset version. DVC stores the real file contents in the DVC cache and, after configuration, in remote storage.

Therefore, changing dataset versions is a two-step process:
1. Use Git to move to the commit or tag that contains the correct `.dvc` pointer file.
2. Use DVC to materialize the real dataset file that matches that pointer.

Practical consequence:
- `git checkout dataset-v1` changes the pointer file to the version 1 hash.
- `dvc checkout` restores the version 1 dataset from local DVC cache if it exists locally.
- `dvc pull` downloads and restores the version 1 dataset from remote storage if the local cache does not contain it.

## 6. Corrected Mistakes From the Original Assignment
The original PDF contains several issues or omissions. They are corrected below.

### 6.1 DagsHub Remote Format in the PDF Is Outdated
The PDF suggests:
```bash
dvc remote add -d dagshub https://dagshub.com/YOUR_USER/sentiment-dvcdata.dvc
```

Current DagsHub documentation instead shows an S3-compatible DVC remote. The generic form is:
```bash
dvc remote add -d dagshub s3://dvc
dvc remote modify dagshub endpointurl <DAGSHUB_REMOTE_URL>
```

Where:
```text
<DAGSHUB_REMOTE_URL> = https://dagshub.com/<DAGSHUB_USERNAME>/<DAGSHUB_REPO>.s3
```

This is the most important correction in the assignment.

### 6.2 Credentials Should Not Be Stored in Tracked Git Files
The PDF suggests adding DagsHub credentials with commands that would place secrets into repository configuration unless handled carefully.

Correct approach:
```bash
dvc remote modify --local dagshub access_key_id <DAGSHUB_TOKEN>
dvc remote modify --local dagshub secret_access_key <DAGSHUB_TOKEN>
```

Reason:
- `--local` writes to `.dvc/config.local`
- `.dvc/config.local` is ignored by Git through `.dvc/.gitignore`
- secrets must not be committed to the repository

### 6.3 The First Git Push Needs an Upstream Branch
The PDF uses `git push` without setting upstream.

Safer first push:
```bash
git remote add origin <GITHUB_REPO_URL>
git push -u origin main
```

### 6.4 Switching Dataset Versions Needs a DVC Step After Git Checkout
The PDF asks to switch versions but does not make the Git-plus-DVC relationship explicit.

Correct workflow:
```bash
git checkout dataset-v1
dvc checkout
```

On another machine, or if local cache is missing:
```bash
git checkout dataset-v1
dvc pull -r dagshub
```

### 6.5 Retrieving Data on Another Machine Requires DVC Pull
Cloning the Git repository alone is not enough, because Git only downloads the `.dvc` pointer file, not the real dataset contents.

Correct additional step:
```bash
dvc pull -r dagshub
```

### 6.6 The Push Sequence in the PDF Is Incomplete
A more sensible full sequence is:
1. Create and push the GitHub repository.
2. Create or import the matching DagsHub repository.
3. Configure the DVC remote in `.dvc/config`.
4. Commit the DVC remote configuration.
5. Push the Git commit that contains `.dvc/config`.
6. Push DVC data objects with `dvc push`.
7. Push Git tags if they are used.

Recommended commands:
```bash
git remote add origin <GITHUB_REPO_URL>
git push -u origin main

dvc remote add -d dagshub s3://dvc
dvc remote modify dagshub endpointurl <DAGSHUB_REMOTE_URL>
dvc remote modify --local dagshub access_key_id <DAGSHUB_TOKEN>
dvc remote modify --local dagshub secret_access_key <DAGSHUB_TOKEN>

git add .dvc/config
git commit -m "Configure DagsHub DVC remote"
git push origin main
dvc push -r dagshub
git push origin --tags
```

### 6.7 Separate Code Repo and Data Repo Is Optional, Not Required
The PDF implies a GitHub code repository and a separate DagsHub data repository.

Practical correction:
- The cleanest setup is usually to create the GitHub repository, then import or mirror that repository into DagsHub and use the DagsHub repository's DVC storage.
- This keeps Git commits, `.dvc` files, and DagsHub's data UI aligned in one place.

This point is an inference from DagsHub's current repository-level DVC storage workflow and its documentation that instructs users to copy DVC configuration commands from the target repository's Data tab.

## 7. How to Connect the Repository to GitHub and DagsHub
### 7.1 GitHub Commands
Create an empty repository on GitHub first. Then run:
```bash
git remote add origin <GITHUB_REPO_URL>
git push -u origin main
git push origin --tags
```

### 7.2 DagsHub Commands
Create or import the repository on DagsHub first. Then open the repository page and copy the DVC commands from `Remote -> Data -> DVC`.

Generic commands:
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

Placeholder meanings:
- `<GITHUB_REPO_URL>`: GitHub HTTPS repository URL, for example `https://github.com/<GITHUB_USERNAME>/sentiment-dvc.git`
- `<DAGSHUB_USERNAME>`: DagsHub username
- `<DAGSHUB_REMOTE_URL>`: `https://dagshub.com/<DAGSHUB_USERNAME>/<DAGSHUB_REPO>.s3`
- `<DAGSHUB_TOKEN>`: DagsHub access token

## 8. How to Retrieve the Dataset on Another Machine
### 8.1 Standard Terminal Workflow
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

Expected result:
- Git downloads the repository and the `.dvc` pointer files.
- DVC downloads the real dataset contents into `data/raw/sentiment.csv`.

If `.dvc/config` in the cloned repository does not yet contain the `dagshub` remote, run the DagsHub setup commands from Section 7.2 first.

### 8.2 Google Colab Workflow
In Colab, run the following in a bash cell or prefix each command with `!`:
```bash
git clone <GITHUB_REPO_URL>
cd sentiment-dvc
pip install "dvc[s3]"

dvc remote modify --local dagshub access_key_id <DAGSHUB_TOKEN>
dvc remote modify --local dagshub secret_access_key <DAGSHUB_TOKEN>

git checkout dataset-v2
dvc pull -r dagshub
python - <<'PY'
import pandas as pd
print(pd.read_csv('data/raw/sentiment.csv'))
PY
```

Expected result:
- The notebook environment downloads the chosen dataset version.
- The printed dataframe shows the version checked out by Git and restored by DVC.

## 9. How to List and Switch Between Dataset Versions
### 9.1 List Available Versions
Using tags:
```bash
git tag --list
```

Example output:
```text
dataset-v1
dataset-v2
```

Using commit history for the DVC pointer file:
```bash
git log --oneline -- data/raw/sentiment.csv.dvc
```

Example output:
```text
e1bee55 Track sentiment dataset v2
bb3654c Track sentiment dataset v1
```

### 9.2 Switch to Version 1
If the dataset already exists in the local DVC cache:
```bash
git checkout dataset-v1
dvc checkout
```

If the dataset is not in the local DVC cache:
```bash
git checkout dataset-v1
dvc pull -r dagshub
```

Expected result:
- `data/raw/sentiment.csv` is restored to the 4-row version.

### 9.3 Switch to Version 2
If the dataset already exists in the local DVC cache:
```bash
git checkout dataset-v2
dvc checkout
```

If the dataset is not in the local DVC cache:
```bash
git checkout dataset-v2
dvc pull -r dagshub
```

Expected result:
- `data/raw/sentiment.csv` is restored to the 8-row version.

## 10. Commands and Example Outputs
### 10.1 Local Git History
Command:
```bash
git log --oneline --decorate --all
```

Actual local output:
```text
e1bee55 (HEAD -> main, tag: dataset-v2) Track sentiment dataset v2
bb3654c (tag: dataset-v1) Track sentiment dataset v1
feaec47 Initialize Git and DVC
```

### 10.2 Data Tracking Files
Command:
```bash
cat data/raw/sentiment.csv.dvc
```

Example interpretation:
- The `md5` value changes when the dataset changes.
- The `size` field changed from `125` in version 1 to `266` in version 2.
- Git tracks this `.dvc` file, not the CSV itself.

### 10.3 Current Dataset State
Current `main` branch state corresponds to `dataset-v2`.

Current dataset rows:
```csv
text,label
"I love this movie",positive
"This was terrible",negative
"Pretty good overall",positive
"Waste of time",negative
"Absolutely fantastic acting",positive
"The plot was engaging",positive
"I would not recommend this",negative
"Too long and boring",negative
```

## 11. Screenshot Checklist
Insert screenshots in the final submission at the following points:

[Screenshot 1: project structure in the terminal]

[Screenshot 2: `git init -b main` and `dvc init`]

[Screenshot 3: generated `.dvc` files and `.gitignore` files]

[Screenshot 4: `dvc add data/raw/sentiment.csv` for version 1]

[Screenshot 5: commit `Track sentiment dataset v1`]

[Screenshot 6: updated CSV content for version 2]

[Screenshot 7: commit `Track sentiment dataset v2`]

[Screenshot 8: GitHub repository creation or repository page]

[Screenshot 9: DagsHub repository page and copied DVC remote commands]

[Screenshot 10: `git push -u origin main`]

[Screenshot 11: `dvc push -r dagshub`]

[Screenshot 12: `git tag --list` or `git log --oneline -- data/raw/sentiment.csv.dvc`]

[Screenshot 13: switching to `dataset-v1` and restoring data with DVC]

[Screenshot 14: retrieving the dataset on another machine or in Google Colab]

## 12. Conclusion
The lab demonstrates that DVC data versioning depends on Git history. Git selects the correct tracked pointer file, and DVC restores the matching real dataset from cache or remote storage. The repository is complete locally, but GitHub and DagsHub pushes still require manual account-based steps.

## 13. References
Official sources used to correct the assignment:
- DagsHub DVC integration docs: https://dagshub.com/docs/integration_guide/dvc/
- DVC documentation: https://dvc.org/doc
