# Collaborating on Analog_382m

This guide is for teammates working on the Cadence Virtuoso and ADE L project on the UT lab servers. Each teammate uses their own GitHub account, lab account, and clone. Cloning gives you the project's history, the main branch, and the origin remote in one step.

## 1. Get repository access

**Owner:** Open the [project repository](https://github.com/gamenewbie/Analog_382m). Under **Settings → Collaborators**, invite your teammate by GitHub username.

**Teammate:** Accept the invitation while signed in to your own account. Confirm that you can open the repository and see its files on GitHub.

## 2. Authenticate from the lab server

For teammates on someone else's personal repository, SSH is a convenient option. GitHub limits fine-grained tokens for outside collaborators. A token created for the owner's account should stay with the owner.

1. In your own lab terminal, run `ls -la ~/.ssh` to see whether you already have a key pair. If an existing key is registered with your GitHub account, start with the connection test below.
2. To create a key, run `ssh-keygen -t ed25519 -C "your-email@example.com"`, substituting your email. On an account with no existing default key, press Enter to accept the default filename and choose a passphrase. If prompted to overwrite a key, cancel and use the existing key or choose a separate filename.
3. For the default filename, run `cat ~/.ssh/id_ed25519.pub`. Copy the entire public-key line. On GitHub, open **Settings → SSH and GPG keys → New SSH key**, select **Authentication Key**, paste the public key, and save. Keep the private key on your lab account.
4. Run `ssh -T git@github.com`. On first connection, compare the displayed fingerprint with [GitHub's published fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints). A greeting naming your GitHub username confirms authentication. The accompanying message about shell access is expected.

**HTTPS alternative:** If you already use HTTPS credentials with write access to this repository, use the HTTPS clone command below. At the username prompt enter your own GitHub username. At the password prompt paste your own personal access token. For the repository owner, a fine-grained token can be scoped to Analog_382m with **Contents → Read and write**. [GitHub token documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

## 3. Clone once and verify the checkout

From the parent directory where you want your project folder, run **one** clone command:

```bash
# SSH
git clone git@github.com:gamenewbie/Analog_382m.git

# Alternatively, HTTPS
git clone https://github.com/gamenewbie/Analog_382m.git
```

If you already have a separate Analog_382m folder containing your own work, preserve it and clone in another parent directory. A teammate joining an existing repository can skip git init, adding a remote, and merging unrelated histories.

```bash
cd Analog_382m
git remote -v
git branch -vv
git status
```

Expect origin to point to gamenewbie/Analog_382m and main to track origin/main. Set the commit author for this clone, substituting your own details:

```bash
git config user.name "Your Name"
git config user.email "your-github-email@example.com"
```

## 4. Open the shared design in Virtuoso

Start Virtuoso using the usual course environment so the course libraries and device models resolve. In Library Manager, check that **Analog_382m** points to the library in your clone. If it is missing, open **Tools → Library Path Editor** and add the existing library directory to your local Cadence setup. Choose the directory containing the library's cell folders. If there is already an Analog_382m entry pointing to an older copy, update that mapping.

Open an existing testbench schematic. Launch ADE L and use **Session → Load State → Cellview** if the team saved a simulation state. Check that the device models resolve and run the testbench once. For a setup you want to share, use **Session → Save State → Cellview** in the testbench library. A state saved only under your home directory is outside the clone.

## 5. Make and share a change

Coordinate which cell and view each person will edit. Two people can work on different cells at the same time. Git cannot combine concurrent schematic or layout edits to the same binary OpenAccess file.

The following example uses a task branch and a pull request. Replace the example branch name and cell folder with your own.

```bash
git checkout main
git pull --ff-only origin main
git checkout -b update-bias-circuit
```

Edit the design in Virtuoso, **Check and Save**, run the relevant testbench, and close the edited views before switching branches or pulling updates. Review and stage the intended files:

```bash
git status --short
git add -- path/to/changed-cell/
git diff --cached --stat
git commit -m "Update bias circuit and its testbench"
git push -u origin update-bias-circuit
```

On GitHub, open **Compare & pull request**, select **main** as the base, and describe the changed cells and what you simulated. After the team merges the pull request, update your local copy:

```bash
git checkout main
git pull --ff-only origin main
```

## Files to share and common issues

| Situation | What to do |
| --- | --- |
| Choosing files for a commit | Include edited schematic, symbol, layout, saved ADE state, and hand-written stimulus files. Keep generated Spectre runs, CDS.log files, and .cdslck locks out of commits. Check that the simulation directory rule in .gitignore matches your local run directory. |
| Accidentally staged a folder | Run `git restore --staged -- folder-name/` to keep the local files and remove them from the next commit. |
| Permission denied (publickey) | Run `ssh -T git@github.com` on the lab server. Confirm GitHub recognizes the same account that accepted the invitation. |
| HTTPS rejects your login | Check `git remote get-url --push origin`. For an HTTPS remote, retry with `git -c credential.helper= push` on a branch with an upstream and use a valid token when prompted. |
| Push rejected or branches diverged | Run `git fetch origin`, then `git status -sb` and `git log --oneline --graph --decorate --all -12`. If local main has your work and remote main has advanced, create a task branch at your current commit and push that branch for review. Preserve both versions of a conflicting cellview and coordinate which edits to reapply in Virtuoso. |
