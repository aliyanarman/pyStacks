# pyStacks

A visual interface for building pandas data cleaning pipelines. Configure transforms on your dataset through a GUI, and pyStacks generates the corresponding pandas code as a Jupyter notebook.

---

## What it is

Data cleaning in pandas is repetitive. The same `str.strip()`, `fillna()`, `pd.to_datetime()` calls appear in every project. The logic is not complex — but writing it, ordering it correctly, and making it readable takes time.

pyStacks sits in front of that process. You load a dataset, click through columns, configure what needs to happen to each one, and get back a `.ipynb` file with the pandas code already written. The notebook has one cell per transform, one comment per cell, sequential execution counts, and automatic verification cells after operations like null drops and type casts.

It runs entirely in the browser. One HTML file, no server, no install.

---

## How the pipeline works

**Load** — drop in a CSV, TSV, XLSX, XLS, ODS, or JSON file. pyStacks reads it and infers a data type for every column. You can override the type from the sidebar.

**Configure** — click a column. The transform list filters automatically to operations relevant to that column's type. Add transforms one at a time, or apply a preset that populates a full stack for common column types like Email, Name, Phone, Date, Salary, Age, and others. Each step in the stack is ordered, parameterisable, and movable.

**Preview** — the table at the bottom updates as you build the stack. Toggle between first-N rows and a random sample at any point.

**Export** — generate a `.ipynb` notebook or export the cleaned data directly as CSV, TSV, XLSX, JSON, XML, SQL, HTML, or Markdown.

---

## Notebook output

The `.ipynb` pyStacks generates follows a strict format:

- One cell per transform step
- One lowercase descriptive comment: `# filled nulls with column median — salary`
- No `print()`, no `pd.set_option`, no boilerplate
- `execution_count` set sequentially — cells open as already-run in Jupyter
- Automatic **verification cells** after: `drop_nulls`, `drop_duplicates`, date parse, numeric cast, boolean normalise, conditional column, and others

The load cell uses the correct reader for the file type:
```python
df = pd.read_excel('data.xlsx')         # .xlsx / .xls / .ods
df = pd.read_json('data.json')          # .json
df = pd.read_csv('data.tsv', sep='\t')  # .tsv
df = pd.read_csv('data.csv')            # .csv
```

Example of a generated cleaning sequence:
```python
# trimmed spaces — name
df['name'] = df['name'].astype(str).str.strip()

# applied title case — name
df['name'] = df['name'].astype(str).str.lower().str.title()

# removed extra @ symbols — email
df['email'] = df['email'].astype(str).apply(
    lambda x: (lambda p: p[0].replace('@','') + '@' + p[-1])(x.split('@')) if x.count('@') > 1 else x
)

# filled nulls with column median — salary
df['salary'] = pd.to_numeric(df['salary'], errors='coerce').fillna(
    pd.to_numeric(df['salary'], errors='coerce').median()
)

# verify: filled nulls — salary
df['salary'].isnull().sum()
```

---

## Transforms

| Category | Operations |
|---|---|
| **String** | `str.strip()`, `str.lower/upper/title()`, `str.replace()`, `str.extract()`, `str.split()`, pad, prefix, suffix, regex extract/replace |
| **Email** | Fix extra `@`, fix consecutive dots in domain, lowercase, flag invalid, extract username / domain / TLD / provider |
| **Name** | Strip invisible unicode, collapse spaces, title case, remove digits, international name character standards, split first/last |
| **Phone** | Strip non-numeric, digits only, US `(xxx) xxx-xxxx` format, international `+` prefix |
| **Date** | `pd.to_datetime()`, Excel serial number conversion, ISO format, extract year / month / day / quarter / weekday, days since |
| **Numeric** | `pd.to_numeric()`, `fillna()` (median / mean / zero), `abs()`, `clip()`, `round()`, normalise 0–1, z-score, `log1p()`, `pd.cut()`, percentile rank |
| **Boolean** | Normalise `yes/no/1/0/y/n` → `True/False`, cast to int, fill nulls, invert |
| **Conditional** | `np.where()` with 12 operators: `>` `>=` `<` `<=` `==` `!=` `contains` `startswith` `endswith` `isnull` `notna` `between` |
| **URL** | Lowercase, strip trailing slash, `urlparse()` for domain / path / scheme, strip query params |
| **General** | `dropna()`, `drop_duplicates()`, `rename()`, `drop()`, `replace()`, `fillna()`, regex |

### Presets

Single-click stacks for: `Email` `Name` `Phone` `Date` `DateTime` `Salary` `Currency` `Age` `Percentage` `ID / Code` `Address` `Country` `URL` `Gender` `Boolean / Flag` `Score / Rating` `Category / Label` `Description / Text`

---

## Excel serial dates

Excel stores dates as integers — `44501` means 2021-11-01. If a column contains a mix of text dates and Excel serials, pyStacks parses the whole column correctly and generates the Python equivalent:
```python
def _parse_date(s):
    s = str(s).strip()
    if s.lower() in ('', 'null', 'na', 'n/a', 'none'): return pd.NaT
    if s.isdigit() and 1 <= int(s) <= 73050:
        return pd.Timestamp('1899-12-30') + pd.Timedelta(days=int(s))
    return pd.to_datetime(s, dayfirst=False, errors='coerce')

df['date'] = df['date'].apply(_parse_date).dt.strftime('%Y-%m-%d')
```

---

## Getting started
```bash
git clone https://github.com/your-username/pystacks
open pystacks/pystacks.html
```

Or download `pystacks.html` from Releases and open it directly. No Python, no server, no dependencies.

---

## Tech stack

- [SheetJS](https://sheetjs.com/) 0.18.5 — Excel / ODS parsing
- [PapaParse](https://www.papaparse.com/) 5.4.1 — CSV / TSV parsing
- Vanilla JavaScript, single HTML file

---

## License

MIT

---

*pyStacks — by Aliyan A.*
