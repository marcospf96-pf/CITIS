# Install

pip install -U python-jobspy

#There are 2 sections, one for the Base code and the other one to filter just entry level positions.

# Modify Values

Modify the values from the step 1 section and add the search tearm , location and number of results.

# Entry Level

from jobspy import scrape_jobs
import pandas as pd
import csv

#Step 1: scrape
```
df = scrape_jobs(
    site_name=["linkedin"],
    search_term="Network Engineer",
    location="Jalisco",
    results_wanted=30,  # you can increase later
    linkedin_fetch_description=True,
)

print("Columns:", df.columns)
```
#Step 2: define description keywords

```
keywords = ["vpn", "tcp", "wan", "lan", "ip", "cloud", "vmware", "cisco"]
pattern = "|".join(keywords)

```
#Step 3: find description column
```
desc_col = next((c for c in df.columns if "desc" in c.lower()), None)
if not desc_col:
    raise ValueError("No description column found!")
```
#Step 4: filter by description content
```
filtered_df = df[df[desc_col].astype(str).str.contains(pattern, case=False, na=False)]
```

#Step 5: filter by entry-level job level (when available)

```
if "job_level" in filtered_df.columns:
    entry_keywords = ["entry", "junior", "intern", "graduate", "trainee"]
    filtered_df = filtered_df[
        filtered_df["job_level"].astype(str).str.contains("|".join(entry_keywords), case=False, na=False)
        | filtered_df["title"].astype(str).str.contains("|".join(entry_keywords), case=False, na=False)
    ]
```

#Step 6: pick link column (LinkedIn has both job_url and job_url_direct)

```
link_col = "job_url_direct" if "job_url_direct" in df.columns else "job_url"
```

```
#Step 7: show only the first few for console readability
pd.set_option("display.max_colwidth", 120)
print(filtered_df[["title", "company", link_col, desc_col]].head())
```

#Step 8: save to CSV with full description text
```
filtered_df.to_csv("jobs_filtered_entry_level.csv", index=False, quoting=csv.QUOTE_ALL, encoding="utf-8")
print(f"\n✅ Saved {len(filtered_df)} filtered entry-level jobs to jobs_filtered_entry_level.csv")
```

# Base Code

```
import csv
from jobspy import scrape_jobs

jobs = scrape_jobs(
    site_name=["indeed", "linkedin", "zip_recruiter", "google"], # "glassdoor", "bayt", "naukri", "bdjobs"
    search_term="software engineer",
    google_search_term="software engineer jobs near San Francisco, CA since yesterday",
    location="San Francisco, CA",
    results_wanted=20,
    hours_old=72,
    country_indeed='USA',
    
    # linkedin_fetch_description=True # gets more info such as description, direct job url (slower)

```
    # proxies=["208.195.175.46:65095", "208.195.175.45:65095", "localhost"],
)
print(f"Found {len(jobs)} jobs")
print(jobs.head())
jobs.to_csv("jobs.csv", quoting=csv.QUOTE_NONNUMERIC, escapechar="\\", index=False) # to_excel


