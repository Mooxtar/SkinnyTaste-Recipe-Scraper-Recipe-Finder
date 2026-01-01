# SkinnyTaste Recipe Scraper + Recipe Finder (Colab)

This project collects recipe data from **SkinnyTaste** archive pages, builds a clean CSV dataset, performs a small exploratory analysis (EDA), and provides an interactive **Gradio** UI to filter recipes by **Calories** and **Personal Points**.

It was written to be runnable in **Google Colab**, with reliability features like retries, caching, and checkpointing so the scraping process can survive interruptions.

---

## What this project does

### 1) Web scraping (dataset creation)
- Collects recipe URLs from SkinnyTaste archive pages (default: **50 pages**).
- Scrapes each recipe page and extracts:
  - **Recipe Name**
  - **Calories**
  - **Personal Points**
  - **Image URL**
  - **Recipe Keys** (tags/icons on the page)
  - **Summary**
  - **URL**
- Saves progress regularly to a checkpoint CSV so you don’t lose work if Colab disconnects.

### 2) Data cleaning + EDA
- Checks missing values
- Drops incomplete rows
- Converts calories to numeric
- Splits “Recipe Keys” into lists, explodes them, and visualizes their distribution
- Plots:
  - Calories distribution
  - Personal points distribution
  - Calories vs points scatterplot

### 3) Interactive Recipe Finder (Gradio)
A simple UI where you choose:
- Min/Max Calories
- Min/Max Points

Then it displays the **top 10 recipes** (sorted by calories) as clean “cards” with:
- title, calories, points
- summary
- clickable recipe URL
- image (if available)

---

## Results (from the notebook run)

- Scraped and saved: **1001 recipes** into `recipes.csv`
- Missing values before cleaning:
  - Calories: **176**
  - Personal Points: **176**
  - Recipe Keys: **166**
  - Summary: **1**
- After dropping missing rows: **815 complete recipes** remained (7 columns)

---

## Files produced

During scraping (Colab paths):
- `recipes_links.txt` — cached list of collected recipe URLs  
- `recipes_checkpoint.csv` — checkpoint dataset saved every N recipes  
- `recipes.csv` — final dataset



## How to run (Google Colab)

1. Upload the notebook to Colab
2. Run cells top-to-bottom

The notebook installs the needed packages (requests, bs4, lxml, pandas, tqdm, seaborn/matplotlib, gradio).

### Output location
In Colab the CSVs are saved into:
- `/content/recipes.csv`
- `/content/recipes_checkpoint.csv`

---

## How scraping works (implementation details)

### Reliable requests session
- Uses `requests.Session()`
- Adds browser-like headers
- Enables retries with `urllib3.Retry`:
  - total retries: 6
  - backoff factor: 0.5
  - retries on status codes: 429, 500, 502, 503, 504

### URL collection
- Iterates archive pages 1..MAX_PAGES
- Extracts recipe article links (`article[id^='post-']`)
- Deduplicates links
- Saves them to `recipes_links.txt`
- Adds a small delay (`sleep(0.1)`) to be polite

### Parallel scraping
- Uses `ThreadPoolExecutor` (default workers: 8)
- Scrapes in batches
- Saves checkpoint every `SAVE_EVERY` recipes

---

## Configuration (easy knobs)

These constants are in the notebook:

- `MAX_PAGES = 50`  
  Number of archive pages to scan.

- `WORKERS = 8`  
  Number of threads used for parallel scraping.

- `SAVE_EVERY = 25`  
  Checkpoint frequency (in scraped recipes).

- `FORCE_RECOLLECT_LINKS = False`  
  If `True`, ignores cached `recipes_links.txt` and recollects links again.

---

## Notes on responsible scraping

This is an educational mini-project. When scraping any website:
- respect the site’s rules and rate limits
- avoid aggressive request rates
- do not republish scraped content as your own

The dataset here stores only short metadata fields (name, numbers, tags, summary, and link).

---

## Troubleshooting

**`recipes.csv not found`**
- Run the scraping cell first, or upload your CSV as `recipes.csv`
- The Gradio cell searches these paths:
  - `/content/recipes.csv`
  - `/content/recipes_checkpoint.csv`
  - `recipes.csv`


---

## Acknowledgements
Data source: SkinnyTaste recipe pages (public website).
