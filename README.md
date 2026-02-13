import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import numpy as np

# ============================================================
# CONFIG
# ============================================================
st.set_page_config(
    page_title="SEO Traffic Analysis",
    page_icon="📊",
    layout="wide",
    initial_sidebar_state="expanded"
)

COLORS = {
    'primary': '#FF6B35',
    'secondary': '#1B998B',
    'tertiary': '#5B8C5A',
    'red': '#E84855',
    'purple': '#7B68EE',
    'gold': '#F4A261',
    'gray': '#94A3B8',
    'dark': '#1E293B',
}
COLOR_SEQUENCE = [COLORS['primary'], COLORS['secondary'], COLORS['tertiary'],
                  COLORS['red'], COLORS['purple'], COLORS['gold'], COLORS['gray']]

# ============================================================
# CUSTOM CSS
# ============================================================
st.markdown("""
<style>
    @import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&display=swap');

    .stApp {
        font-family: 'DM Sans', sans-serif;
    }
    .metric-card {
        background: white;
        border: 1px solid #E8E5DF;
        border-radius: 12px;
        padding: 20px;
        text-align: center;
    }
    .metric-value {
        font-size: 32px;
        font-weight: 700;
        color: #FF6B35;
        margin: 4px 0;
    }
    .metric-label {
        font-size: 12px;
        color: #94A3B8;
        text-transform: uppercase;
        letter-spacing: 0.5px;
    }
    .metric-sub {
        font-size: 11px;
        color: #94A3B8;
        margin-top: 2px;
    }
    .finding-box {
        background: #FFF9F5;
        border-left: 4px solid #FF6B35;
        padding: 16px 20px;
        border-radius: 0 8px 8px 0;
        margin: 12px 0;
    }
    .finding-box-green {
        background: #F0FAF8;
        border-left: 4px solid #1B998B;
        padding: 16px 20px;
        border-radius: 0 8px 8px 0;
        margin: 12px 0;
    }
    div[data-testid="stSidebar"] {
        background: #0F2B3C;
    }
    div[data-testid="stSidebar"] * {
        color: white !important;
    }
    h1, h2, h3 {
        color: #1E293B !important;
    }
</style>
""", unsafe_allow_html=True)


# ============================================================
# DATA LOADING
# ============================================================
@st.cache_data
def load_data():
    monthly = pd.read_csv('data/looker_monthly_trend.csv')
    pages = pd.read_csv('data/looker_page_summary.csv')
    clusters = pd.read_csv('data/looker_cluster_performance.csv')
    weekly = pd.read_csv('data/looker_weekly_trend.csv')
    monthly_cluster = pd.read_csv('data/looker_monthly_by_cluster.csv')
    return monthly, pages, clusters, weekly, monthly_cluster

monthly, pages, clusters, weekly, monthly_cluster = load_data()


# ============================================================
# HELPER FUNCTIONS
# ============================================================
def fmt_k(n):
    if n >= 1_000_000:
        return f"{n/1_000_000:.1f}M"
    if n >= 1_000:
        return f"{n/1_000:.1f}K"
    return f"{n:.0f}"

def metric_card(label, value, sub=""):
    st.markdown(f"""
    <div class="metric-card">
        <div class="metric-label">{label}</div>
        <div class="metric-value">{value}</div>
        <div class="metric-sub">{sub}</div>
    </div>
    """, unsafe_allow_html=True)

def finding_box(text, color="orange"):
    css_class = "finding-box" if color == "orange" else "finding-box-green"
    st.markdown(f'<div class="{css_class}">{text}</div>', unsafe_allow_html=True)

def clean_plotly(fig, height=400):
    fig.update_layout(
        template="plotly_white",
        height=height,
        font=dict(family="DM Sans", size=12),
        margin=dict(l=40, r=20, t=50, b=40),
        legend=dict(orientation="h", yanchor="bottom", y=1.02, xanchor="right", x=1),
        colorway=COLOR_SEQUENCE,
    )
    return fig


# ============================================================
# SIDEBAR
# ============================================================
with st.sidebar:
    st.markdown("## 📊 SEO Analysis")
    st.markdown("---")
    page_selection = st.radio(
        "Navigate",
        ["📈 Overview", "🌐 Subdomains", "📂 WWW Sections", "🏷️ Content Clusters",
         "📄 Top Pages", "🔍 Opportunities", "📉 H2 2023 Deep Dive", "📋 Action Plan"],
        label_visibility="collapsed"
    )
    st.markdown("---")
    st.markdown("""
    <div style="font-size: 11px; opacity: 0.6;">
    Data: Jan 2022 — Jan 2024<br>
    Source: Google Search Console<br>
    Pages analyzed: 2,156
    </div>
    """, unsafe_allow_html=True)


# ============================================================
# PAGE: OVERVIEW
# ============================================================
if page_selection == "📈 Overview":
    st.title("Client SEO Traffic Analysis")
    st.caption("GSC data · Jan 2022 — Jan 2024 · All subdomains")

    # Metrics row
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        metric_card("Total Clicks", "947K", "Jan 2022 — Jan 2024")
    with col2:
        metric_card("Peak Month", "103K", "October 2023")
    with col3:
        metric_card("Growth", "~12x", "8K → 103K /mo")
    with col4:
        metric_card("Unique Pages", "2,156", "Across all subdomains")

    st.markdown("<br>", unsafe_allow_html=True)

    # Total monthly trend
    total_monthly = monthly.groupby('month').agg(
        clicks=('clicks', 'sum'),
        impressions=('impressions', 'sum')
    ).reset_index().sort_values('month')

    fig = go.Figure()
    fig.add_trace(go.Scatter(
        x=total_monthly['month'], y=total_monthly['clicks'],
        mode='lines', fill='tozeroy',
        line=dict(color=COLORS['primary'], width=2.5),
        fillcolor='rgba(255, 107, 53, 0.1)',
        name='Clicks'
    ))
    fig.add_vline(x='2023-01', line_dash="dash", line_color=COLORS['red'],
                  annotation_text="Content Launch", annotation_position="top left")
    fig.update_layout(title="Total Monthly Clicks — All Subdomains",
                      xaxis_title="", yaxis_title="Clicks")
    st.plotly_chart(clean_plotly(fig, 400), use_container_width=True)

    finding_box("🚀 <strong>Primary growth driver:</strong> 302 new articles published under /resources/articles/ starting Jan 2023. "
                "Traffic grew from ~8K to ~103K clicks/month — a 12x increase in 10 months.")

    # Before/after comparison
    col1, col2 = st.columns(2)
    with col1:
        st.markdown("#### 2022 (Pre-Content)")
        st.markdown("""
        - Avg monthly: **~8.6K clicks**
        - Top sources: Homepage (60%), Careers (18%), Blog (12%)
        - Resources section: **did not exist**
        """)
    with col2:
        st.markdown("#### 2023 H2 (Post-Content)")
        st.markdown("""
        - Avg monthly: **~93K clicks**
        - Top sources: Resources/Articles (91%), Homepage (5%), Careers (3%)
        - Resources section: **THE growth driver**
        """)


# ============================================================
# PAGE: SUBDOMAINS
# ============================================================
elif page_selection == "🌐 Subdomains":
    st.title("Subdomain Analysis")
    st.caption("Traffic breakdown by subdomain")

    # Summary table
    sub_summary = pages.groupby('subdomain').agg(
        total_clicks=('total_clicks', 'sum'),
        total_impressions=('total_impressions', 'sum'),
        pages_count=('page', 'nunique')
    ).reset_index().sort_values('total_clicks', ascending=False)
    sub_summary['pct'] = (sub_summary['total_clicks'] / sub_summary['total_clicks'].sum() * 100).round(2)

    col1, col2 = st.columns([2, 1])
    with col1:
        # Monthly trend by subdomain
        monthly_sub = monthly.groupby(['month', 'subdomain'])['clicks'].sum().reset_index()
        www_data = monthly_sub[monthly_sub['subdomain'] == 'www'].sort_values('month')
        other_data = monthly_sub[monthly_sub['subdomain'] != 'www'].groupby('month')['clicks'].sum().reset_index()

        fig = go.Figure()
        fig.add_trace(go.Scatter(x=www_data['month'], y=www_data['clicks'],
                                 mode='lines', fill='tozeroy', name='www',
                                 line=dict(color=COLORS['primary'], width=2.5),
                                 fillcolor='rgba(255, 107, 53, 0.1)'))
        fig.add_trace(go.Scatter(x=other_data['month'], y=other_data['clicks'],
                                 mode='lines', name='All other subdomains',
                                 line=dict(color=COLORS['gray'], width=2)))
        fig.update_layout(title="Monthly Clicks: www vs Everything Else")
        st.plotly_chart(clean_plotly(fig, 350), use_container_width=True)

    with col2:
        fig = px.pie(sub_summary.head(5), values='total_clicks', names='subdomain',
                     color_discrete_sequence=COLOR_SEQUENCE)
        fig.update_layout(title="Share of Total Clicks")
        st.plotly_chart(clean_plotly(fig, 350), use_container_width=True)

    finding_box("🌐 <strong>www = 99.5% of all traffic.</strong> Other subdomains (go, app, home, es, fr) "
                "account for less than 5K clicks combined over 2 years. Focus all SEO efforts on www.")

    st.dataframe(
        sub_summary.style.format({
            'total_clicks': '{:,.0f}',
            'total_impressions': '{:,.0f}',
            'pct': '{:.2f}%'
        }),
        use_container_width=True,
        hide_index=True
    )


# ============================================================
# PAGE: WWW SECTIONS
# ============================================================
elif page_selection == "📂 WWW Sections":
    st.title("WWW Section Breakdown")
    st.caption("How different sections of the www subdomain contribute to traffic")

    # Stacked area by section
    monthly_sections = monthly[monthly['subdomain'] == 'www'].groupby(
        ['month', 'section']
    )['clicks'].sum().reset_index()

    top_sections = monthly_sections.groupby('section')['clicks'].sum().nlargest(6).index.tolist()
    monthly_sections['section_group'] = np.where(
        monthly_sections['section'].isin(top_sections),
        monthly_sections['section'],
        'other'
    )
    monthly_grouped = monthly_sections.groupby(['month', 'section_group'])['clicks'].sum().reset_index()

    fig = px.area(monthly_grouped, x='month', y='clicks', color='section_group',
                  color_discrete_sequence=COLOR_SEQUENCE)
    fig.update_layout(title="Monthly Clicks by WWW Section (Stacked)",
                      xaxis_title="", yaxis_title="Clicks",
                      legend_title="Section")
    st.plotly_chart(clean_plotly(fig, 450), use_container_width=True)

    finding_box("📂 <strong>Resources/Articles dominates all growth.</strong> It launched ~Jan 2023 and grew to represent "
                "~90% of www traffic by H2 2023. Homepage and Careers remained flat throughout.")

    # Blog → Resources migration
    st.markdown("### Blog → Resources Migration")
    blog_res = monthly[
        (monthly['subdomain'] == 'www') &
        (monthly['section'].isin(['blog', 'resources']))
    ].groupby(['month', 'section'])['clicks'].sum().reset_index()

    fig = px.line(blog_res, x='month', y='clicks', color='section',
                  color_discrete_map={'blog': COLORS['red'], 'resources': COLORS['secondary']},
                  markers=True)
    fig.update_layout(title="Blog vs Resources — Content Migration",
                      xaxis_title="", yaxis_title="Clicks")
    st.plotly_chart(clean_plotly(fig, 350), use_container_width=True)

    finding_box("📝 <strong>Blog went to 0 by March 2023.</strong> Content was migrated to /resources/articles/ "
                "as part of the new content strategy.", "green")

    # Section summary table
    section_summary = monthly[monthly['subdomain'] == 'www'].groupby('section').agg(
        total_clicks=('clicks', 'sum'),
        unique_pages=('unique_pages', 'max')
    ).reset_index().sort_values('total_clicks', ascending=False)
    section_summary['pct'] = (section_summary['total_clicks'] / section_summary['total_clicks'].sum() * 100).round(1)

    st.markdown("### Section Summary")
    st.dataframe(
        section_summary.head(15).style.format({
            'total_clicks': '{:,.0f}',
            'pct': '{:.1f}%'
        }),
        use_container_width=True,
        hide_index=True
    )


# ============================================================
# PAGE: CONTENT CLUSTERS
# ============================================================
elif page_selection == "🏷️ Content Clusters":
    st.title("Content Cluster Performance")
    st.caption("302 articles across 15 topic clusters — which verticals are winning?")

    # Cluster bar charts
    col1, col2 = st.columns(2)
    with col1:
        fig = px.bar(clusters.sort_values('total_clicks'), x='total_clicks', y='Cluster',
                     orientation='h', color_discrete_sequence=[COLORS['primary']],
                     text=clusters.sort_values('total_clicks')['pages'].apply(lambda x: f"{x} pages"))
        fig.update_layout(title="Total Clicks by Cluster", xaxis_title="Total Clicks", yaxis_title="")
        fig.update_traces(textposition='outside')
        st.plotly_chart(clean_plotly(fig, 500), use_container_width=True)

    with col2:
        fig = px.bar(clusters.sort_values('clicks_per_page'), x='clicks_per_page', y='Cluster',
                     orientation='h', color_discrete_sequence=[COLORS['secondary']])
        fig.update_layout(title="Clicks Per Page (Efficiency)", xaxis_title="Clicks/Page", yaxis_title="")
        st.plotly_chart(clean_plotly(fig, 500), use_container_width=True)

    finding_box("🏷️ <strong>Back and Neck clusters have highest total clicks AND best per-page efficiency.</strong> "
                "Exercise Therapy has the most pages (79) but lowest clicks/page. "
                "Glossary Terms (48 pages) have near-zero traffic — major underperformer.")

    # Cluster trends over time
    st.markdown("### Cluster Growth Over Time")
    top_6 = clusters.head(6)['Cluster'].tolist()
    cluster_trend = monthly_cluster[monthly_cluster['Cluster'].isin(top_6)]

    fig = px.line(cluster_trend, x='month', y='clicks', color='Cluster',
                  color_discrete_sequence=COLOR_SEQUENCE, markers=True)
    fig.update_layout(title="Monthly Clicks by Top 6 Clusters",
                      xaxis_title="", yaxis_title="Clicks")
    st.plotly_chart(clean_plotly(fig, 400), use_container_width=True)

    # Full cluster table
    st.markdown("### Full Cluster Data")
    st.dataframe(
        clusters.style.format({
            'total_clicks': '{:,.0f}',
            'total_impressions': '{:,.0f}',
            'avg_position': '{:.1f}',
            'avg_word_count': '{:.0f}',
            'ctr': '{:.2f}%',
            'clicks_per_page': '{:,.0f}'
        }),
        use_container_width=True,
        hide_index=True
    )


# ============================================================
# PAGE: TOP PAGES
# ============================================================
elif page_selection == "📄 Top Pages":
    st.title("Top Performing Pages")

    # Filters
    col1, col2, col3 = st.columns(3)
    with col1:
        section_filter = st.selectbox("Section", ["All"] + sorted(pages['section'].unique().tolist()))
    with col2:
        content_filter = st.selectbox("Content Type", ["All", "New Content", "Existing Content"])
    with col3:
        sort_by = st.selectbox("Sort by", ["total_clicks", "total_impressions", "ctr", "avg_position"])

    filtered = pages.copy()
    if section_filter != "All":
        filtered = filtered[filtered['section'] == section_filter]
    if content_filter != "All":
        filtered = filtered[filtered['content_type'] == content_filter]

    filtered = filtered.sort_values(sort_by, ascending=(sort_by == 'avg_position'))

    # Top 20 bar chart
    top_20 = filtered.head(20)
    fig = px.bar(top_20, x='total_clicks', y='short_name', orientation='h',
                 color_discrete_sequence=[COLORS['primary']],
                 hover_data=['total_impressions', 'avg_position', 'ctr'])
    fig.update_layout(title=f"Top 20 Pages by {sort_by.replace('_', ' ').title()}",
                      xaxis_title="", yaxis_title="", yaxis=dict(autorange="reversed"))
    st.plotly_chart(clean_plotly(fig, 550), use_container_width=True)

    # Data table
    st.markdown(f"### All Pages ({len(filtered):,} results)")
    display_cols = ['short_name', 'section', 'content_type', 'total_clicks',
                    'total_impressions', 'avg_position', 'ctr']
    if 'Cluster' in filtered.columns:
        display_cols.insert(3, 'Cluster')

    st.dataframe(
        filtered[display_cols].head(100).style.format({
            'total_clicks': '{:,.0f}',
            'total_impressions': '{:,.0f}',
            'avg_position': '{:.1f}',
            'ctr': '{:.2f}%'
        }),
        use_container_width=True,
        hide_index=True
    )


# ============================================================
# PAGE: OPPORTUNITIES
# ============================================================
elif page_selection == "🔍 Opportunities":
    st.title("CTR & Ranking Opportunities")
    st.caption("Find pages with untapped potential")

    # Scatter plot: position vs CTR
    articles = pages[pages['section'] == 'resources'].copy()
    articles = articles[articles['total_impressions'] > 5000]

    fig = px.scatter(articles, x='avg_position', y='ctr',
                     size='total_impressions', color='content_type',
                     hover_name='short_name',
                     hover_data=['total_clicks', 'total_impressions'],
                     color_discrete_map={
                         'New Content': COLORS['primary'],
                         'Existing Content': COLORS['gray']
                     },
                     size_max=40, opacity=0.6)
    fig.add_hline(y=1.0, line_dash="dash", line_color=COLORS['gray'], opacity=0.5)
    fig.add_vline(x=20, line_dash="dash", line_color=COLORS['gray'], opacity=0.5)
    fig.update_layout(title="Position vs CTR — Bubble Size = Impressions",
                      xaxis_title="Average Position", yaxis_title="CTR %",
                      xaxis=dict(range=[0, 60]))
    st.plotly_chart(clean_plotly(fig, 500), use_container_width=True)

    finding_box("🔍 <strong>Bottom-left quadrant = biggest opportunities.</strong> High impressions, low CTR, poor position. "
                "Improving titles/meta or building links to push these up could unlock significant traffic.")

    # Quick wins: position 11-15 with high impressions
    st.markdown("### 🎯 Quick Wins: Pages Just Below Page 1")
    st.markdown("These pages rank position 11-15 with significant impressions. A small ranking boost pushes them to page 1.")
    quick_wins = articles[
        (articles['avg_position'] >= 11) &
        (articles['avg_position'] <= 15) &
        (articles['total_impressions'] > 30000)
    ].sort_values('total_impressions', ascending=False)

    st.dataframe(
        quick_wins[['short_name', 'total_clicks', 'total_impressions', 'avg_position', 'ctr']].style.format({
            'total_clicks': '{:,.0f}',
            'total_impressions': '{:,.0f}',
            'avg_position': '{:.1f}',
            'ctr': '{:.2f}%'
        }),
        use_container_width=True,
        hide_index=True
    )

    # Low CTR with high impressions
    st.markdown("### ⚡ Title/Meta Optimization Targets")
    st.markdown("High impressions but low CTR — the page is ranking but people aren't clicking. Fix the title and meta description.")
    low_ctr = articles[
        (articles['total_impressions'] > 50000) &
        (articles['ctr'] < 1.0)
    ].sort_values('total_impressions', ascending=False)

    st.dataframe(
        low_ctr[['short_name', 'total_clicks', 'total_impressions', 'avg_position', 'ctr']].style.format({
            'total_clicks': '{:,.0f}',
            'total_impressions': '{:,.0f}',
            'avg_position': '{:.1f}',
            'ctr': '{:.2f}%'
        }),
        use_container_width=True,
        hide_index=True
    )

    # Position distribution
    st.markdown("### 📊 Position Distribution")
    bins = [0, 3, 10, 20, 50, 100]
    labels = ['1-3 (Top)', '4-10', '11-20', '21-50', '50+']
    articles['pos_bucket'] = pd.cut(articles['avg_position'], bins=bins, labels=labels)
    pos_dist = articles.groupby('pos_bucket').agg(
        pages=('short_name', 'count'),
        clicks=('total_clicks', 'sum')
    ).reset_index()

    col1, col2 = st.columns(2)
    with col1:
        fig = px.bar(pos_dist, x='pos_bucket', y='pages',
                     color_discrete_sequence=[COLORS['secondary']], text='pages')
        fig.update_layout(title="# Articles by Position", xaxis_title="", yaxis_title="Articles")
        fig.update_traces(textposition='outside')
        st.plotly_chart(clean_plotly(fig, 350), use_container_width=True)
    with col2:
        fig = px.bar(pos_dist, x='pos_bucket', y='clicks',
                     color_discrete_sequence=[COLORS['primary']], text='clicks')
        fig.update_layout(title="Total Clicks by Position", xaxis_title="", yaxis_title="Clicks")
        fig.update_traces(textposition='outside')
        st.plotly_chart(clean_plotly(fig, 350), use_container_width=True)


# ============================================================
# PAGE: H2 2023 DEEP DIVE
# ============================================================
elif page_selection == "📉 H2 2023 Deep Dive":
    st.title("Why Did Traffic Stop Growing?")
    st.caption("Investigating the H2 2023 plateau and decline")

    # Weekly trend
    weekly_total = weekly.groupby('week').agg(
        clicks=('clicks', 'sum'),
        impressions=('impressions', 'sum')
    ).reset_index().sort_values('week')

    # Filter to focus period
    focus = weekly_total[
        (weekly_total['week'] >= '2023-06') &
        (weekly_total['week'] <= '2024-01')
    ]

    fig = make_subplots(rows=2, cols=1, shared_xaxes=True,
                        subplot_titles=("Weekly Clicks", "Weekly Impressions"))
    fig.add_trace(go.Scatter(x=focus['week'], y=focus['clicks'],
                             mode='lines', fill='tozeroy', name='Clicks',
                             line=dict(color=COLORS['primary'], width=2),
                             fillcolor='rgba(255, 107, 53, 0.1)'), row=1, col=1)
    fig.add_trace(go.Scatter(x=focus['week'], y=focus['impressions'],
                             mode='lines', fill='tozeroy', name='Impressions',
                             line=dict(color=COLORS['secondary'], width=2),
                             fillcolor='rgba(27, 153, 139, 0.1)'), row=2, col=1)

    # Google update annotations
    fig.add_vline(x='2023-10-02/2023-10-08', line_dash="dash", line_color=COLORS['red'],
                  annotation_text="Oct Core Update", row=1, col=1)
    fig.add_vline(x='2023-11-06/2023-11-12', line_dash="dash", line_color=COLORS['red'],
                  annotation_text="Nov Core Update", row=1, col=1)

    fig.update_layout(title="Weekly Traffic — Jun 2023 to Jan 2024", height=500,
                      font=dict(family="DM Sans"), template="plotly_white",
                      margin=dict(l=40, r=20, t=60, b=40), showlegend=False)
    st.plotly_chart(fig, use_container_width=True)

    # New vs existing during the plateau
    st.markdown("### New vs Existing Content During Plateau")
    weekly_content = weekly[
        (weekly['week'] >= '2023-06') &
        (weekly['week'] <= '2024-01')
    ].sort_values('week')

    fig = px.area(weekly_content, x='week', y='clicks', color='content_type',
                  color_discrete_map={
                      'New Content': COLORS['primary'],
                      'Existing Content': COLORS['gray']
                  })
    fig.update_layout(title="Weekly Clicks: New vs Existing Content",
                      xaxis_title="", yaxis_title="Clicks")
    st.plotly_chart(clean_plotly(fig, 350), use_container_width=True)

    # Hypothesis
    st.markdown("### 🔬 Hypothesis: Why Traffic Plateaued")

    st.markdown("""
    <div class="finding-box">
    <strong>1. Google Algorithm Updates (Oct & Nov 2023 Core Updates)</strong><br>
    Google rolled out two core updates in Q4 2023. Health/medical content is particularly sensitive to
    E-E-A-T signals. The timing aligns with the initial flattening.
    </div>

    <div class="finding-box">
    <strong>2. Content Saturation / Diminishing Returns</strong><br>
    302 articles were published in ~12 months. Later articles targeted increasingly competitive or lower-volume keywords,
    yielding less traffic per page. Exercise Therapy cluster (79 pages) has the lowest clicks/page.
    </div>

    <div class="finding-box">
    <strong>3. Potential Internal Cannibalization</strong><br>
    With 302 articles in overlapping health topics, pages may compete against each other in SERPs,
    diluting ranking signals.
    </div>

    <div class="finding-box">
    <strong>4. Seasonality</strong><br>
    Health/exercise content typically dips Nov-Dec (holidays). This is consistent with the seasonal pattern
    but doesn't fully explain the plateau starting in October.
    </div>
    """, unsafe_allow_html=True)

    st.markdown("### Recommended Investigation")
    st.markdown("""
    - Pull Google update timeline and overlay with weekly traffic to confirm causation
    - Run a keyword cannibalization audit (multiple pages ranking for same queries)
    - Compare position changes for top 50 pages pre/post October 2023
    - Check Search Console for manual actions or security issues
    - Analyze competitor content growth in the same period
    """)


# ============================================================
# PAGE: ACTION PLAN
# ============================================================
elif page_selection == "📋 Action Plan":
    st.title("Recommended Action Plan")

    st.markdown("""
    <div class="finding-box-green">
    <strong>Priority 1 — Quick Wins (Weeks 1-4)</strong><br><br>
    • Optimize title tags and meta descriptions for high-impression, low-CTR pages<br>
    • Push position 11-15 articles to page 1 via internal linking and content refreshes<br>
    • Audit Glossary Term cluster (48 pages, near-zero traffic) — consider noindexing or consolidating
    </div>

    <div class="finding-box-green">
    <strong>Priority 2 — Content Optimization (Months 1-3)</strong><br><br>
    • Double down on top-performing clusters: Back, Neck, Knee content has proven demand<br>
    • Improve internal linking between related articles within clusters<br>
    • Content refresh for early 2023 articles losing freshness signals<br>
    • Add structured data (FAQ schema, HowTo schema) to exercise/treatment articles
    </div>

    <div class="finding-box">
    <strong>Priority 3 — Strategic Expansion (Months 3-6)</strong><br><br>
    • Expand into adjacent topic clusters (shoulder → rotator cuff, posture, ergonomics)<br>
    • Build topical authority with hub pages linking cluster articles together<br>
    • Audit Exercise Therapy cluster for content quality (79 pages, lowest clicks/page)<br>
    • Run cannibalization audit to identify and merge competing pages
    </div>

    <div class="finding-box">
    <strong>Priority 4 — Technical & Structural</strong><br><br>
    • Consolidate subdomain traffic: redirect relevant go/home/app pages to www<br>
    • Ensure clean canonical tags across the site<br>
    • Monitor Core Web Vitals for the articles section<br>
    • Set up automated GSC reporting for ongoing monitoring
    </div>
    """, unsafe_allow_html=True)

    # Impact estimates
    st.markdown("### Estimated Impact")
    impact_data = pd.DataFrame({
        'Initiative': ['Title/Meta Optimization', 'Page 1 Push (pos 11-15)', 'Glossary Fix/Remove',
                       'Content Refresh', 'Internal Linking', 'New Cluster Content'],
        'Est. Monthly Click Lift': ['5-10K', '10-20K', '< 1K (cleanup)', '5-15K', '5-10K', '10-30K'],
        'Effort': ['Low', 'Medium', 'Low', 'Medium', 'Medium', 'High'],
        'Timeline': ['1-2 weeks', '2-4 weeks', '1 week', '1-3 months', '2-4 weeks', '3-6 months']
    })
    st.dataframe(impact_data, use_container_width=True, hide_index=True)
