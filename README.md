# 📊 AI Jobs Market 2025–2026: Salary & Skills Analysis

A Python data analysis project exploring salaries and in-demand skills across **1,500 AI job postings** from 2025–2026, built while learning pandas, matplotlib, and seaborn from the ground up.

---

## 📑 Table of Contents

- [Overview](#-overview)
- [The Questions](#-the-questions-i-set-out-to-answer)
- [Tools I Used](#-tools-i-used)
- [The Data](#-the-data)
- [Data Cleanup](#-data-cleanup)
- [The Analysis](#-the-analysis)
  - [1. Salary by Job Title](#1-average-salary-by-job-title)
  - [2. Most In-Demand Skills](#2-top-10-most-in-demand-skills)
  - [3. Demand vs. Salary](#3-skill-demand-vs-salary-distribution)
  - [4. Highest-Paying Skills](#4-skills-that-pay-the-most)
- [Key Insights](#-key-insights)
- [What I Learned](#-what-i-learned)
- [Challenges I Faced](#-challenges-i-faced)
- [How to Run This Project](#-how-to-run-this-project)
- [Acknowledgments](#-acknowledgments)

---

## 📌 Overview

This project digs into a snapshot of the AI job market to answer a question I was genuinely curious about: **which AI skills are worth learning — the popular ones, or the ones nobody's talking about?**

It started as a practice project for learning pandas and matplotlib (loosely following the structure of Luke Barousse's Python data analysis course), then turned into a full mini-analysis once I found a dataset I actually cared about.

---

## ❓ The Questions I Set Out to Answer

1. What's the average salary by job title, and which roles show up most often?
2. Which skills are most **in-demand** across job postings?
3. Is there a relationship between how in-demand a skill is and how much it pays?
4. Which skills pay the **most**, regardless of how common they are?

---

## 🛠 Tools I Used

- **Python** — core language
- **Pandas** — data cleaning, grouping, aggregation
- **Matplotlib** — plotting and custom formatting
- **Seaborn** — statistical visualizations (bar plots, boxplot, joint plot)
- **VS Code / Jupyter Notebooks** — development environment

---

## 📂 The Data

**Source:** [AI Jobs Market 2025–2026 | Salaries](https://www.kaggle.com/datasets/alitaqishah/ai-jobs-market-2025-2026-salaries) — Kaggle dataset by Syed Ali Taqi

| Detail | Value |
|---|---|
| File | `ai_jobs_market_2025_2026.csv` |
| Rows | 1,500 job postings |
| Columns | 25 |
| Unique job titles | 25 |
| Unique skills | 93 |
| Countries represented | 14 |
| Salary range | $90,000 – $384,000 |
| Median salary | $180,000 |
| Posting window | Jan 2025 – Dec 2026 |

**Columns used most:** `job_title`, `annual_salary_usd`, `required_skills`

Other available columns (not all used yet, but worth exploring further): `experience_level`, `country`, `remote_work`, `company_size`, `industry`, `is_llm_role`, `salary_tier`.

---

## 🧹 Data Cleanup

Two things needed fixing before any analysis could happen:

```python
df = pd.read_csv("ai_jobs_market_2025_2026.csv")

# required_skills comes in as one long pipe-separated string,
# e.g. "Python|SQL|AWS" — split it into an actual list
df['required_skills'] = df['required_skills'].str.split('|')

# explode() turns each skill in that list into its own row,
# so a job with 5 skills becomes 5 rows (one per skill)
df_exploded = df.explode('required_skills')
```

This turned 1,500 job postings into **9,548 job–skill pairs** — the dataset I actually needed for the skills-based questions (Q2–Q4).

---

## 📈 The Analysis

### 1. Average Salary by Job Title

To find the roles that show up most often and see how their pay compares, I grouped by `job_title` and pulled the median salary and posting count for the top 10 by volume.

```python
skill_stats = df_exploded.groupby('job_title').agg(
    median_salary=('annual_salary_usd', 'median'),
    job_count=('job_title', 'count')
).sort_values(by='job_count', ascending=False).head(10)

ax.scatter(skill_stats['job_count'], skill_stats['median_salary'], s=150, alpha=0.6)
for idx, row in skill_stats.iterrows():
    ax.annotate(idx, xy=(row['job_count'], row['median_salary']),
                xytext=(10, 10), textcoords='offset points')
```

**Chart type:** Scatter plot, with each job title labeled directly on its point.

**Findings:**
| Job Title | Postings | Median Salary |
|---|---|---|
| LLM Engineer | 487 | $232,000 |
| Prompt Engineer | 467 | $153,000 |
| Robotics Engineer (AI) | 462 | $154,000 |
| Generative AI Engineer | 459 | $177,000 |
| AI Product Manager | 445 | $175,000 |
| Senior Data Scientist | 434 | $162,000 |
| Senior ML Engineer | 431 | $227,000 |
| Data Scientist | 397 | $159,000 |
| AI Engineer | 391 | $142,000 |
| Multimodal AI Engineer | 391 | $209,000 |

**LLM Engineer** is the standout — it's both the *most-posted* role **and** the *highest-paying* one in the top 10. Meanwhile, **Prompt Engineer** and **Robotics Engineer (AI)** are nearly as common but pay ~$75–80K less, showing that posting volume alone doesn't predict pay.

---

### 2. Top 10 Most In-Demand Skills

```python
skill_counts = df_exploded['required_skills'].value_counts().head(10)
df_plot = pd.DataFrame({
    'job_skills': skill_counts.index,
    'skill_count': skill_counts.values,
})
df_plot['skill_percent'] = (df_plot['skill_count'] / total_jobs * 100)

sns.barplot(data=df_plot, y='job_skills', x='skill_percent',
            hue='skill_count', palette='dark:salmon_r', ax=ax)
```

**Chart type:** Horizontal seaborn bar chart, with each bar labeled with its exact percentage.

**Findings:**
| Skill | % of Job–Skill Pairs |
|---|---|
| Python | 9.9% |
| SQL | 4.7% |
| Cloud | 4.5% |
| Leadership | 4.0% |
| Communication | 4.0% |
| Research | 3.9% |
| Agile | 3.7% |
| Statistics | 3.7% |
| Linux | 3.4% |
| Problem Solving | 3.3% |

**Python dominates** — it shows up in nearly 1 out of every 10 skill listings, roughly double the next closest skill (SQL). After the top two technical skills, the list is a mix of general/soft skills (Leadership, Communication, Problem Solving) rather than niche AI tools.

---

### 3. Skill Demand vs. Salary Distribution

```python
skill_stats = df_exploded.groupby('required_skills').agg(
    skill_count=('required_skills', 'count'),
    mean_salary=('annual_salary_usd', 'mean')
).reset_index()

g = sns.jointplot(data=skill_stats, x='skill_count', y='mean_salary',
                   kind='kde', height=8, cmap='Blues', fill=True)
```

**Chart type:** KDE joint plot — a density map of where skills cluster, with marginal distributions on each axis.

**Findings:** The correlation between how in-demand a skill is and how much it pays on average came out to **r ≈ 0.04** — essentially no relationship. Most skills cluster tightly at low demand (fewer than ~100 postings) across a *wide* range of salaries ($140K–$250K), while a small handful of generalist skills (Python chief among them) have very high demand without commanding a premium.

This is the most important chart in the project — it visually proves that **"in demand" and "high paying" are two separate axes**, not the same thing.

---

### 4. Skills That Pay the Most

```python
top_10_skills = df_exploded.groupby('required_skills')['annual_salary_usd'].mean().nlargest(10).index
df_top_pay = df_exploded[df_exploded['required_skills'].isin(top_10_skills)]
skill_order = df_top_pay.groupby('required_skills')['annual_salary_usd'].median().sort_values(ascending=False).index

sns.boxplot(data=df_top_pay, y='required_skills', x='annual_salary_usd',
            order=skill_order, palette='Set2',
            flierprops=dict(marker='o', markerfacecolor='red', markersize=8))
```

**Chart type:** Boxplot — shows the full salary distribution (median, spread, outliers) per skill, not just the average.

**Findings:**
| Skill | Mean Salary |
|---|---|
| System Design | $248,948 |
| Prompt Engineering | $246,382 |
| LLM Fine-tuning | $242,633 |
| RAG | $241,190 |
| Enterprise Architecture | $240,613 |
| MLOps | $230,352 |
| Tool Use | $228,568 |
| LangChain | $227,987 |
| Fine-tuning | $226,628 |
| Search Systems | $226,225 |

**None of these skills appear in the Top 10 most in-demand list from Q2.** The highest-paying skills are specialized, LLM/architecture-adjacent capabilities — the opposite profile of the generalist skills (Python, SQL, Cloud) that dominate job postings.

---

## 💡 Key Insights

- **Demand ≠ pay.** The correlation between skill frequency and salary is close to zero (r ≈ 0.04). Being everywhere (like Python) doesn't mean being paid a premium for it.
- **Specialization pays.** Every skill in the top-10 highest-paying list (System Design, Prompt Engineering, RAG, LLM Fine-tuning, MLOps...) is a specialized, LLM-era skill — none of them crack the top-10 *most common* list.
- **Volume leader ≠ pay leader, except once.** LLM Engineer is the rare case where the most-posted role is also the highest-paid — most other high-volume roles (Prompt Engineer, Robotics Engineer) pay well below the top tier.
- **LLM-specific roles pay more on average** — jobs flagged `is_llm_role` average **$207,746** vs. **$191,309** for non-LLM roles, a premium of roughly $16K.
- **The job market is broad, not concentrated** — 93 distinct skills appear across postings, and most individual skills show up in fewer than 100 of the 1,500 postings, meaning employers are asking for a long tail of specialized capabilities rather than a handful of universal ones.

---

## 🎓 What I Learned

Going into this project I didn't know pandas or matplotlib beyond the basics — most of these concepts I picked up while building it:

- **`inplace=True`** — modifying a DataFrame directly vs. returning a new copy
- **`enumerate()`** — looping with both an index and a value at once
- **`fig, ax = plt.subplots(rows, cols)`** — the difference between the figure (whole canvas) and axes (individual plots), and indexing into `ax[i]` / `ax[i, j]` for multi-plot layouts
- **`.groupby().agg()`** — aggregating multiple stats (count, mean, median) in a single pass
- **`.str.split('|')` + `.explode()`** — turning a pipe-separated string column into one row per item, which was essential for any skill-level analysis
- **`.iloc[]`** — positional indexing, and why it's needed when pairing a loop counter with column values
- **`plt.text()` vs. `ax.annotate(..., xytext=..., textcoords='offset points')`** — the second one lets you offset labels so they don't sit directly on top of data points
- **`FuncFormatter`** — custom axis tick formatting (turning `275000` into `$275K`)
- **Choosing the right chart per question** — scatter for point-by-point comparison, bar for ranked categories, KDE joint plot for density/distribution, boxplot for spread + outliers — rather than defaulting to the same chart type everywhere
- **Seaborn vs. matplotlib/pandas plotting** — seaborn needs a DataFrame (not a Series) and uses `hue`/`palette` for color encoding, e.g. `sns.barplot(data=df, x=..., y=..., hue=..., palette=...)`

---

## 🚧 Challenges I Faced

- **Kaggle data loading.** My first attempt used `kagglehub.load_dataset()` pointed at a folder instead of the actual CSV filename, which threw a `ValueError: Unsupported file extension`. Switched to a straightforward `pd.read_csv()` once the file was downloaded locally.
- **`plt.subplot()` vs. `plt.subplots()` and `.plot()` vs. `.subplot()`.** Easy to mix up — one letter changes the whole function.
- **`kind='scatter'` on a grouped Series.** `df.groupby(...)['col'].mean().plot(kind='scatter')` fails because scatter needs two variables (x *and* y); a single aggregated Series only has one.
- **Overlapping scatter labels.** With 10 job titles crammed into one chart, several labels landed on top of each other (`Robotics Engineer (AI)` and `Prompt Engineer` especially). Fixed with a bigger figure size and manual per-point `xytext` offsets.
- **The `1e-5` scientific notation** on the joint plot's marginal axis — resolved by turning off that axis (`g.ax_marg_y.axis('off')`) rather than fighting with the formatter.
- **Picking a chart for Q3.** Went scatter → hexbin → KDE joint plot before landing on the one that actually told the story (density of where skills cluster), instead of just plotting more dots.

---

## 🚀 How to Run This Project

```bash
# 1. Install dependencies
pip install pandas matplotlib seaborn

# 2. Download the dataset from Kaggle and place it in your project folder:
#    https://www.kaggle.com/datasets/alitaqishah/ai-jobs-market-2025-2026-salaries

# 3. Run the analysis
python ai_jobs_analysis.py
```

**Requirements:** Python 3.9+, pandas, matplotlib, seaborn

---

## 🙏 Acknowledgments

- Dataset: [Syed Ali Taqi on Kaggle](https://www.kaggle.com/datasets/alitaqishah/ai-jobs-market-2025-2026-salaries)
- Project structure loosely inspired by Luke Barousse's Python data analysis course
