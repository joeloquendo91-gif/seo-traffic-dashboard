# SEO Traffic Analysis Dashboard

Interactive Streamlit dashboard analyzing GSC traffic data and page metadata for a client's SEO growth.

## Deploy to Streamlit Community Cloud (Free)

### Step 1: Push to GitHub
1. Create a new GitHub repo (e.g., `seo-traffic-dashboard`)
2. Upload all files maintaining this structure:
```
seo-traffic-dashboard/
├── app.py
├── requirements.txt
├── README.md
└── data/
    ├── looker_monthly_trend.csv
    ├── looker_page_summary.csv
    ├── looker_cluster_performance.csv
    ├── looker_weekly_trend.csv
    └── looker_monthly_by_cluster.csv
```

### Step 2: Deploy
1. Go to [share.streamlit.io](https://share.streamlit.io)
2. Sign in with GitHub
3. Click "New app"
4. Select your repo, branch `main`, file `app.py`
5. Click "Deploy"

Your dashboard will be live at `https://your-app-name.streamlit.app` in about 2 minutes.

## Run Locally (Optional)
```bash
pip install -r requirements.txt
streamlit run app.py
```

## Dashboard Pages
- **Overview** — Total traffic trend, key metrics, before/after comparison
- **Subdomains** — www vs other subdomains breakdown
- **WWW Sections** — Section-level analysis, blog→resources migration
- **Content Clusters** — Topic cluster performance and trends
- **Top Pages** — Filterable/sortable page-level data
- **Opportunities** — CTR/position scatter plot, quick wins, optimization targets
- **H2 2023 Deep Dive** — Weekly trend analysis, plateau investigation
- **Action Plan** — Prioritized recommendations with impact estimates
