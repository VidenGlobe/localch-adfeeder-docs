Google Ads Ad Feeder - Design Document
======================================

**Company Name**: local.ch (Swisscom Directories AG)

**Business Model**: local.ch operates a vast multilingual platform connecting Swiss consumers with local businesses. To drive high-intent traffic to these listings, we manage hyper-local Google Ads campaigns across 3,000+ categories and multiple Swiss cities. We only advertise for our own properties and do not manage ads for third parties.

### Tool Access/Use:
This tool is for internal use only by the local.ch marketing and advertising team. Campaign management is predefined in code and executed programmatically - no external UI is provided.

#### Access model:
- The tool runs as a Python script/service executed by authorized internal users
- Configuration is managed via environment variables and CSV data files stored in cloud storage
- Google Ads API access is controlled via OAuth2 credentials stored securely
- The tool and its outputs are not publicly accessible

#### Execution modes:
- Dry-run mode: Preview planned changes without executing them in Google Ads
- Production mode: Apply changes directly to the Google Ads account

The tool is designed to run on a scheduled basis to update Google Ads assets based on the feed provided, but a CLI interface is also available for manual execution.

### Tool Design: The Ad Feeder implements an idempotent diff-based synchronization approach:

1.  Source Data Loading: Category-city data (with result counts and URLs) is loaded from CSV files stored in cloud storage
2.  Current State Fetching: The tool queries Google Ads API to retrieve existing campaigns, ad groups, keywords, and their statuses
3.  Diff Computation: Compares desired state (source file) with current state (Google Ads) to compute minimal changes
4.  Batch Execution: Applies changes via the Google Ads Batch API for efficiency

Based on predefined rules, the configuration, and the content of the feed, the following are automatically updated in Google Ads on each run:
- Campaigns (organized by L1 category and language)
- Ad Groups (one per category-city combination)
- Keywords (generated from templates + category + city)
- Responsive Search Ads (headlines and descriptions with placeholders)

Over 3,000 categories are assigned to different campaigns across multiple languages (en, de, fr).

### API Services Called:

* Create and manage campaign budgets via the [CampaignBudgetService](https://developers.google.com/google-ads/api/reference/rpc/v22/CampaignBudgetService)
* Create and manage campaigns via the [CampaignService](https://developers.google.com/google-ads/api/reference/rpc/v22/CampaignService)
* Create campaign language and geo targeting via the [CampaignCriterionService](https://developers.google.com/google-ads/api/reference/rpc/v22/CampaignCriterionService)
* Manage campaign labels via the [CampaignLabelService](https://developers.google.com/google-ads/api/reference/rpc/v22/CampaignLabelService)
* Create and manage ad groups via the [AdGroupService](https://developers.google.com/google-ads/api/reference/rpc/v22/AdGroupService)
* Create and manage ad group criteria via the [AdGroupCriterionService](https://developers.google.com/google-ads/api/reference/rpc/v22/AdGroupCriterionService)
* Manage ad group labels via the [AdGroupLabelService](https://developers.google.com/google-ads/api/reference/rpc/v22/AdGroupLabelService)
* Create Responsive Search Ads via the [AdGroupAdService](https://developers.google.com/google-ads/api/reference/rpc/v22/AdGroupAdService)
* Execute bulk operations via the [BatchJobService](https://developers.google.com/google-ads/api/reference/rpc/v22/BatchJobService)
* Fetch current state of campaigns, ad groups, and other resources using the [GoogleAdsService](https://developers.google.com/google-ads/api/reference/rpc/v22/GoogleAdsService)
* Pull account and campaign performance data using the [Customer](https://developers.google.com/google-ads/api/fields/v22/customer) and [Campaign](https://developers.google.com/google-ads/api/fields/v22/campaign) resources for internal reporting purposes (when needed)
* Manage Google Ads assets via the [AssetService](https://developers.google.com/google-ads/api/reference/rpc/v22/AssetService) (manual use when needed)
* Manage account-level content exclusions via the [AccountExclusionService](https://developers.google.com/google-ads/api/reference/rpc/v22/AccountExclusionService) (manual use when needed)
* Manage shared sets and criteria via the [SharedSetService](https://developers.google.com/google-ads/api/reference/rpc/v22/SharedSetService) (manual use when needed)

