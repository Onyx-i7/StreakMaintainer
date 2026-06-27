# StreakMaintainer

<div align="center">

**A lightweight GitHub Actions utility designed to automate contribution heartbeats and maintain activity matrix continuity.**

[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Maintenance](https://img.shields.io/badge/Maintained-yes-green.svg?style=flat-square)](https://github.com/Onyx-i7/StreakMaintainer/graphs/commit-activity)

</div>

---

## Overview

StreakMaintainer is a minimal GitHub Actions workflow that automatically commits to your repository on a daily schedule. This keeps your contribution graph active without manual intervention, making it ideal for maintaining contribution streaks on private repositories while keeping your GitHub activity graph consistent.

### Key Features

- **Ultra-Fast Execution**: Completes in under 15 seconds, consuming minimal GitHub Actions minutes
- **Privacy-First**: Works with private repositories while still recording contributions on your public profile
- **No CI Triggers**: Uses `[skip ci]` to prevent unnecessary workflow runs
- **Zero Dependencies**: No external tools, packages, or services required
- **Fully Customizable**: Adjustable schedule, commit message, and tracked metrics

---

## Technical Architecture

The workflow operates on a deterministic cron schedule inside GitHub's isolated Ubuntu runner environment:

```
┌─────────────────┐
│  Cron Trigger   │ (Daily at 12:00 UTC)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Ubuntu Runner  │ (GitHub-hosted, ephemeral)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Update Ledger  │ (pulse.log with timestamp)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Git Commit     │ (with [skip ci] flag)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Push to Repo   │ (Contribution recorded)
└─────────────────┘
```

### Workflow Configuration

```yaml
name: Streak Maintainer Engine

on:
  schedule:
    - cron: '0 12 * * *' # Triggers daily at 12:00 UTC
  workflow_dispatch:

jobs:
  heartbeat:
    runs-on: ubuntu-latest
    steps:
    - name: Core Checkout
      uses: actions/checkout@v4

    - name: Generate Chrono Ledger
      run: |
        echo "Matrix pulse: $(date -u +'%Y-%m-%dT%H:%M:%SZ')" > pulse.log

    - name: Cryptographic Commit & Push
      run: |
        git config --local user.email "${{ github.actor }}@users.noreply.github.com"
        git config --local user.name "${{ github.actor }}"
        git add pulse.log
        git commit -m "chore: automated contribution heartbeat [skip ci]"
        git push
```

---

## Quick Start for Personal Use

The simplest way to use StreakMaintainer for your own GitHub account is to fork this repository. Follow these steps:

### Step 1: Fork the Repository

1. Navigate to this repository: `https://github.com/Onyx-i7/StreakMaintainer`
2. Click the **Fork** button in the top-right corner
3. Select your personal GitHub account as the destination
4. Wait for the fork to complete (usually takes 5-10 seconds)

### Step 2: Configure Repository Settings

After forking, you need to enable the necessary permissions:

1. Go to your forked repository (e.g., `https://github.com/YOUR_USERNAME/StreakMaintainer`)
2. Click on **Settings** in the top navigation
3. Navigate to **Actions** > **General** in the left sidebar
4. Scroll down to the **Workflow permissions** section
5. Select **Read and write permissions**
6. Click **Save** at the bottom of the page

### Step 3: Enable the Workflow

GitHub disables workflows in forked repositories by default. To enable it:

1. Go to the **Actions** tab in your forked repository
2. You should see a message: "Workflows aren't being run on this forked repository"
3. Click **I understand my workflows, go ahead and enable them**
4. The workflow is now active and will run according to the schedule

### Step 4: Enable Private Contributions (Optional)

If you want these commits to appear on your public contribution graph:

1. Go to your GitHub profile page: `https://github.com/YOUR_USERNAME`
2. Scroll to the contribution graph section
3. Click **Contribution settings** (dropdown above the graph)
4. Select **Private contributions** from the menu

Your automated commits will now appear as green squares on your contribution graph.

### Step 5: Verify the Workflow

To confirm everything is working:

1. Go to the **Actions** tab
2. Click on **Streak Maintainer Engine** in the left sidebar
3. Click **Run workflow** > **Run workflow** to trigger it manually
4. Wait for the workflow to complete (should take ~15 seconds)
5. Check the **pulse.log** file in your repository to see the timestamp

---

## Customization Options

### Adjusting the Schedule

Modify the cron expression in `.github/workflows/streak.yml` to fit your timezone:

```yaml
on:
  schedule:
    # Every day at 6:00 AM UTC
    - cron: '0 6 * * *'
    
    # Every 12 hours
    - cron: '0 */12 * * *'
    
    # Every Monday at 9:00 AM UTC
    - cron: '0 9 * * 1'
```

**Cron Format Reference:**
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6, Sunday to Saturday)
│ │ │ │ │
* * * * *
```

### Customizing the Commit Message

```yaml
- name: Cryptographic Commit & Push
  run: |
    git config --local user.email "your-email@example.com"
    git config --local user.name "Your Name"
    git add pulse.log
    git commit -m "chore: daily maintenance [skip ci]"
    git push
```

### Tracking Additional Metrics

```yaml
- name: Generate Chrono Ledger
  run: |
    echo "## Daily Activity Report" > pulse.log
    echo "- Date: $(date -u +'%Y-%m-%d')" >> pulse.log
    echo "- Time: $(date -u +'%H:%M:%S UTC')" >> pulse.log
    echo "- Status: Active" >> pulse.log
    echo "- Repository: ${{ github.repository }}" >> pulse.log
    echo "- Commit SHA: ${{ github.sha }}" >> pulse.log
```

---

## Important Considerations

### GitHub Actions Limits

| Account Type | Monthly Minutes | Storage |
|--------------|----------------|---------|
| Free | 2,000 minutes | 500 MB |
| Pro | 3,000 minutes | 1 GB |
| Team | 3,000 minutes | 2 GB |
| Enterprise | 50,000 minutes | 50 GB |

**StreakMaintainer Usage:**
- Execution time: ~5 seconds per run
- Monthly consumption: ~2.5 minutes (assuming daily runs)
- Storage per commit: <1 KB
- Total monthly storage: ~30 KB

### Repository Activity Requirements

GitHub automatically pauses scheduled workflows in repositories that have been inactive for 60 days. To prevent this:

- Ensure your repository has at least one commit, issue, or pull request every 60 days
- StreakMaintainer's daily commits will keep the repository active
- If paused, manually trigger the workflow via the Actions tab

### Email Address for Commits

The workflow uses `action@github.com` as the commit email. If you want commits to be attributed to your account:

```yaml
- name: Cryptographic Commit & Push
  run: |
    git config --local user.email "${{ github.actor }}@users.noreply.github.com"
    git config --local user.name "${{ github.actor }}"
    git add pulse.log
    git commit -m "chore: automated contribution heartbeat [skip ci]"
    git push
```

This ensures commits appear as coming from your GitHub username.

---

## Troubleshooting

### Workflow Not Running

**Problem**: The scheduled workflow isn't executing.

**Solutions:**
- GitHub may delay scheduled workflows by up to 15 minutes during peak times
- Verify the workflow is enabled in the Actions tab
- Check that the repository hasn't been inactive for 60+ days
- Manually trigger via **Actions** > **Streak Maintainer Engine** > **Run workflow**

### Permission Denied Error

**Problem**: `remote: Permission to git denied to github-actions[bot]`

**Solutions:**
- Verify workflow permissions are set to **Read and write** in Settings > Actions > General
- Ensure the repository isn't archived
- Confirm the workflow file is in the default branch (usually `main` or `master`)

### Commits Not Showing on Profile

**Problem**: Commits are happening but not appearing on your contribution graph.

**Solutions:**
- Enable **Private contributions** in your profile settings
- Ensure the commit email matches your GitHub account email
- Wait up to 24 hours for the graph to update
- Verify the repository isn't a fork of another repository (commits in forks don't count unless you're the owner)

### Workflow Runs but No File Changes

**Problem**: The workflow completes but `pulse.log` doesn't update.

**Solutions:**
- Check the workflow logs in the Actions tab for errors
- Verify the file path is correct in the workflow
- Ensure the runner has write permissions to the repository

---

## Security Considerations

### Token Permissions

The workflow uses the default `GITHUB_TOKEN` provided by GitHub Actions. This token:
- Is automatically rotated for each workflow run
- Has permissions limited to the repository scope
- Cannot access other repositories or organization data
- Is destroyed after the workflow completes

### Best Practices

1. **Never commit secrets**: This workflow doesn't require any secrets or API keys
2. **Review workflow changes**: Always review changes to workflow files before merging
3. **Monitor usage**: Check the Actions tab regularly to ensure the workflow is running as expected
4. **Use branch protection**: Enable branch protection rules to prevent unauthorized workflow modifications

---

## Usage Statistics

| Metric | Value |
|--------|-------|
| Execution Time | ~10-15 seconds |
| Monthly Actions Minutes | ~5-10 minutes |
| Storage per Commit | <1 KB |
| Dependencies | 0 |
| Setup Time | <2 minutes |
| Maintenance Required | None |

---

## Contributing

Contributions are welcome. Feel free to:

- Report bugs via [Issues](https://github.com/Onyx-i7/StreakMaintainer/issues)
- Suggest features via [Issues](https://github.com/Onyx-i7/StreakMaintainer/issues)
- Submit pull requests for improvements

Please ensure all changes maintain the lightweight, zero-dependency philosophy of this project.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## FAQ

**Q: Will this work on private repositories?**  
A: Yes, StreakMaintainer works on both public and private repositories. Commits to private repositories will appear on your contribution graph if you enable "Private contributions" in your profile settings.

**Q: Does this violate GitHub's Terms of Service?**  
A: No. This workflow performs legitimate git operations (commit and push) using GitHub's official Actions infrastructure. It doesn't simulate user activity or manipulate the contribution graph through unauthorized means.

**Q: Can I use this on multiple repositories?**  
A: Yes, you can fork this repository multiple times or copy the workflow file to other repositories. Each repository will maintain its own independent schedule.

**Q: What happens if I delete the `pulse.log` file?**  
A: The workflow will recreate it on the next run. The file is simply a timestamp ledger and has no functional purpose beyond providing a commit target.

**Q: Can I change the frequency to multiple times per day?**  
A: Yes, modify the cron expression. However, be aware that more frequent runs will consume more Actions minutes. GitHub's minimum interval for scheduled workflows is 5 minutes.

**Q: Will this work if my repository is empty?**  
A: No, the repository must have at least one commit before the workflow can run. The initial fork provides this commit.

---

## Acknowledgments

- Built with [GitHub Actions](https://github.com/features/actions)
- Inspired by the need for consistent contribution tracking across private repositories

---

<div align="center">

**Maintained by [Onyx_i7](https://github.com/Onyx-i7)**

</div>
