# AdFeeder Guide for Account Managers

> ⚠️ **Status**: This document is AI-generated and not yet fully reviewed. Please report any inaccuracies to technical team.

> **Purpose**: This guide explains how AdFeeder works and how you can effectively manage your Google Ads account alongside the automated system.

---

## Table of Contents

- [What is AdFeeder?](#what-is-adfeeder)
- [How AdFeeder Works](#how-adfeeder-works)
- [Understanding Labels](#understanding-labels)
- [Managing Ad Groups Alongside AdFeeder](#managing-ad-groups-alongside-adfeeder)
- [Common Scenarios](#common-scenarios)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## What is AdFeeder?

**AdFeeder** is an automated system that manages your Google Ads campaigns based on data from your source files (CSV/BigQuery). It:

- Creates new ad groups based on your data feed
- Pauses ad groups that are no longer needed
- Re-enables ad groups when they return in your data
- Updates ad group content (headlines, descriptions, keywords)

**Key principle**: AdFeeder is designed to work alongside you, not replace you. You maintain full control over ad groups you want to manage manually.

---

## Critical Naming Requirements

### AdFeeder Only Manages Specific Ad Groups and Campaigns

AdFeeder uses a specific **find pattern** to identify which campaigns and ad groups it should manage:

**Find pattern**: `(d_adfeeder_always_on)`

This pattern is embedded within the full name structure. AdFeeder searches for campaigns and ad groups whose names contain this pattern.

**Example names that AdFeeder will manage:**
- `(c_che)(p_local)(l_de)(e_p-search)(b_goo)(d_adfeeder_always_on)(pc_Restaurants)(f_text)(g_commitment)(t_generic)(i_always)(a_broad)(s_all)`
- `(c_che)(p_local)(l_fr)(e_p-search)(b_goo)(d_adfeeder_always_on)(pc_Hotels)(f_text)(g_commitment)(t_generic)(i_always)(a_broad)(s_all)`

**Example names that AdFeeder will NOT manage:**
- `Zurich Restaurants`
- `Geneva Hotels - Testing`
- `Switzerland Tourism`
- `(c_che)(p_local)(l_de)(e_p-search)(b_goo)(d_manual)(pc_Restaurants)(f_text)(g_commitment)(t_generic)(i_always)(a_broad)(s_all)` (missing the pattern)

**Important**: You may see references to an `_always_on` suffix in documentation or discussions. This is a simplified way to refer to the `(d_adfeeder_always_on)` pattern that appears in the full structured names shown above.

### ⚠️ WARNING: Do NOT Rename Ad Groups or Campaigns

**Never rename ad groups or campaigns** that AdFeeder manages. Renaming will cause:

1. **Duplicate creation**: AdFeeder will create a new ad group/campaign with the original name
2. **Orphaned entities**: The renamed entity becomes unmanaged and may cause confusion
3. **Data inconsistency**: The system loses track of which entity corresponds to your source data

**Example of what happens if you rename:**
```
Original: "(c_che)(p_local)(l_de)(e_p-search)(b_goo)(d_adfeeder_always_on)(pc_Restaurants)(f_text)(g_commitment)(t_generic)(i_always)(a_broad)(s_all)" (managed by AdFeeder)
You rename to: "Zurich Restaurants - Updated"

Result:
- AdFeeder sees the original structured name missing from Google Ads
- AdFeeder creates a NEW campaign/ad group with the original structured name
- You now have two entities with similar purposes
- The renamed "Zurich Restaurants - Updated" is now unmanaged
```

**If you need to change an ad group or campaign name:**
1. Coordinate with your technical team
2. Update the source data with the new name
3. Let AdFeeder handle the transition
4. Never rename directly in Google Ads UI

---

## How AdFeeder Works

### The Diff-Based Approach

AdFeeder compares two states:

1. **Desired state**: What should exist (from your source data)
2. **Current state**: What currently exists in Google Ads

It then computes the minimal changes needed:

| Action | When it happens |
|--------|----------------|
| **CREATE** | Ad group exists in your data but not in Google Ads |
| **PAUSE** | Ad group exists in Google Ads but NOT in your data |
| **ENABLE** | Ad group exists in both but is currently paused |
| **UPDATE** | Ad group exists in both and needs content updates |

### Execution Order

AdFeeder executes changes in this specific order:

1. **PAUSE** first (stop spending on outdated ad groups)
2. **ENABLE** second (reactivate relevant ad groups)
3. **UPDATE** third (refresh content)
4. **CREATE** fourth (add new ad groups)

This order ensures budget is reallocated efficiently.

---

## Understanding Labels

Labels are how AdFeeder tracks which ad groups it manages versus which you manage manually.

### Available Labels

| Label | Purpose | Who applies it | When it's used |
|-------|---------|---------------|----------------|
| `adfeeder_paused` | AdFeeder paused this ad group | AdFeeder (automatic) | When AdFeeder pauses an ad group |
| `adfeeder_active` | AdFeeder is managing this ad group | AdFeeder (automatic) | When AdFeeder creates or enables an ad group |
| `adfeeder_do_not_change` | **Don't touch this ad group** | **You (manual)** | When you want full control over an ad group |

### How Labels Work in Practice

**Scenario 1: AdFeeder pauses an ad group**
```
1. Ad group "Zurich Restaurants" removed from your data
2. AdFeeder pauses it in Google Ads
3. AdFeeder adds label: adfeeder_paused
4. Later, if "Zurich Restaurants" returns to your data
5. AdFeeder sees the adfeeder_paused label
6. AdFeeder re-enables it and removes the label
```

**Scenario 2: You manually pause an ad group**
```
1. You manually pause "Geneva Hotels" in Google Ads
2. AdFeeder sees it's paused but still in your data
3. AdFeeder wants to re-enable it
4. ⚠️ Without protection, AdFeeder will re-enable it
5. ✅ With adfeeder_do_not_change label, AdFeeder ignores it
```

---

## Managing Ad Groups Alongside AdFeeder

### When to Use `adfeeder_do_not_change`

Apply this label when you want **full manual control** over an ad group:

- **Policy violations**: Ad group violates Google Ads policies and must remain paused
- **Manual optimization**: You're testing something and don't want AdFeeder to interfere
- **Special cases**: Ad groups that don't fit the standard automation logic
- **Temporary holds**: You need to pause something for a specific reason

### How to Apply `adfeeder_do_not_change`

**In Google Ads UI:**
1. Go to the ad group
2. Click "Labels"
3. Select "adfeeder_do_not_change"
4. Save

**Result:** AdFeeder will:
- Never pause this ad group
- Never enable this ad group
- Never update this ad group
- Skip it in all operations

### Important: The `require_adfeeder_paused_label_for_enable` Setting

There's a configuration option that affects how AdFeeder handles paused ad groups:

**When `require_adfeeder_paused_label_for_enable = FALSE` (default):**
- AdFeeder will re-enable ANY paused ad group that's in your data
- This includes ad groups you manually paused
- **Risk**: AdFeeder might re-enable ad groups you wanted to keep paused

**When `require_adfeeder_paused_label_for_enable = TRUE`:**
- AdFeeder will ONLY re-enable ad groups with the `adfeeder_paused` label
- This means it only re-enables ad groups **it** paused
- Ad groups you manually paused will stay paused
- **Benefit**: Safer for manual management

**Recommendation**: Ask your technical team to enable `require_adfeeder_paused_label_for_enable = TRUE` if you frequently manually pause ad groups.

---

## Common Scenarios

### Scenario 1: Policy Violation - Permanent Pause

**Situation**: An ad group violates Google Ads policies and must never run again.

**Solution**:
1. Pause the ad group manually in Google Ads
2. Apply `adfeeder_do_not_change` label
3. Remove the ad group from your source data (optional, but recommended)

**Result**: AdFeeder will never touch this ad group again.

---

### Scenario 2: Testing New Copy

**Situation**: You want to test new headlines/descriptions without AdFeeder overwriting them.

**Solution**:
1. Apply `adfeeder_do_not_change` label to the ad group
2. Make your manual changes
3. Test performance
4. When done, remove the label to let AdFeeder take over again

**Result**: AdFeeder respects your testing period.

---

### Scenario 3: Seasonal Campaigns

**Situation**: You have seasonal campaigns (e.g., Christmas) that should only run during specific periods.

**Solution**:
- **Option A**: Keep in source data year-round, use `adfeeder_do_not_change` to pause manually
- **Option B**: Remove from source data when not needed, AdFeeder will pause it automatically

**Recommendation**: Option B is cleaner. Let AdFeeder handle the pause/enable cycle.

---

### Scenario 4: Low Activity Auto-Pause

**Situation**: Google auto-pauses an ad group due to low activity (`AD_GROUP_PAUSED_DUE_TO_LOW_ACTIVITY`).

**AdFeeder behavior**: 
- AdFeeder **never** re-enables ad groups auto-paused by Google
- This is a safety feature to respect Google's optimization

**Your action**: 
- Review why it has low activity
- If you want it running, improve performance or targeting
- AdFeeder won't interfere

---

### Scenario 5: Data Feed Errors

**Situation**: An ad group is accidentally removed from your data feed, causing AdFeeder to pause it.

**Immediate action**:
1. Fix the data feed to include the ad group
2. Run AdFeeder again
3. AdFeeder will see it's back in the data and re-enable it

**Prevention**: 
- Always use dry-run mode first: `uv run main.py --dry-run`
- Review the planned changes before executing

---

## Best Practices

### 1. Always Use Dry-Run First

Before executing real changes:

```bash
# Preview what AdFeeder will do
uv run main.py --dry-run

# Review the output carefully
# Only then execute
uv run main.py --execute
```

### 2. Label Your Manual Changes

Whenever you manually pause or modify an ad group:

- Apply `adfeeder_do_not_change` if you don't want AdFeeder to touch it
- This prevents conflicts and confusion

### 3. Keep Your Data Feed Clean

- Remove ad groups from your data when they're no longer needed
- Don't rely on manual pauses + `adfeeder_do_not_change` for routine management
- Let AdFeeder handle the standard pause/enable cycle

### 4. Monitor Logs

AdFeeder logs all actions. Check the logs regularly:

```bash
# View recent logs
tail -f logs/adfeeder_*.log
```

Look for:
- Ad groups being paused unexpectedly
- Errors or warnings
- Changes to ad groups you thought were protected

### 5. Communication with Technical Team

If you notice issues:

1. Check if the ad group has `adfeeder_do_not_change`
2. Review the logs for that ad group
3. Share the log snippet with your technical team
4. Ask about enabling `require_adfeeder_paused_label_for_enable = TRUE`

### 6. Regular Audits

Periodically review:

- Ad groups with `adfeeder_do_not_change` label (are they still needed?)
- Ad groups paused by AdFeeder (should any be re-enabled?)
- Ad groups enabled by AdFeeder (should any be paused?)

---

## Troubleshooting

### Problem: AdFeeder re-enabled an ad group I manually paused

**Cause**: `require_adfeeder_paused_label_for_enable = FALSE` (default)

**Solutions**:
1. Apply `adfeeder_do_not_change` label to prevent future interference
2. Ask technical team to enable `require_adfeeder_paused_label_for_enable = TRUE`

---

### Problem: AdFeeder paused an ad group I need running

**Cause**: The ad group was removed from your source data

**Solutions**:
1. Add it back to your source data
2. Run AdFeeder again (dry-run first, then execute)
3. AdFeeder will re-enable it

---

### Problem: AdFeeder won't pause an ad group I want paused

**Cause**: The ad group has `adfeeder_do_not_change` label

**Solution**:
1. Remove the `adfeeder_do_not_change` label
2. Remove the ad group from your source data
3. Run AdFeeder again
4. Or: Pause it manually and re-apply `adfeeder_do_not_change`

---

### Problem: AdFeeder is updating ad groups I don't want touched

**Cause**: The ad group doesn't have `adfeeder_do_not_change` label

**Solution**:
1. Apply `adfeeder_do_not_change` label
2. AdFeeder will skip it in future runs

---

### Problem: I can't find the `adfeeder_do_not_change` label

**Cause**: The label hasn't been created yet

**Solution**:
1. Ask your technical team to run AdFeeder once (it will create all labels automatically)
2. Or: Create it manually in Google Ads UI (name it exactly: `adfeeder_do_not_change`)

---

## Quick Reference

### Label Decision Tree

```
Did you manually pause/modify this ad group?
├─ YES
│  └─ Do you want AdFeeder to manage it?
│     ├─ NO → Apply adfeeder_do_not_change
│     └─ YES → Remove adfeeder_do_not_change (if present)
└─ NO
   └─ Let AdFeeder handle it (no action needed)
```

### Action Checklist

Before making manual changes:

- [ ] Does this ad group have `adfeeder_do_not_change`?
- [ ] Should I apply/remove `adfeeder_do_not_change`?
- [ ] Did I update the source data if needed?
- [ ] Should I run AdFeeder in dry-run mode first?

After AdFeeder runs:

- [ ] Review the execution summary
- [ ] Check logs for any warnings
- [ ] Verify unexpected pauses/enables
- [ ] Update documentation if needed

---

## Getting Help

If you have questions or issues:

1. **Check the logs**: `logs/adfeeder_*.log`
2. **Review this guide**: Look for similar scenarios
3. **Contact your technical team**: Share log snippets and describe the issue
4. **Use dry-run mode**: Always preview before executing

---

## Summary

**AdFeeder is your automated assistant, not your replacement.**

- It handles routine pause/enable/update operations based on your data
- You maintain full control via the `adfeeder_do_not_change` label
- Labels are your primary tool for managing the human + automation partnership
- Communication and regular audits ensure smooth collaboration

**Key takeaway**: When in doubt, apply `adfeeder_do_not_change` to protect ad groups you want to manage manually. You can always remove it later when you're ready to hand control back to AdFeeder.
