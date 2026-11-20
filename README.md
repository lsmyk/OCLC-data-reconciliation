# Alma OCLC Record Matcher

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/lsmyk/OCLC-data-reconciliation/blob/main/Matching%20Alma%20to%20OCLC%20records%20LAST%20update.ipynb)

A Python utility designed to evaluate and improve the precision of matching local library catalog records with OCLC data using multi-field validation.

## Purpose & Background
During a Greenglass inventory evaluation, our library identified tens of thousands of bibliographic records that lack publication dates. While we want to update these records by pulling the missing dates from OCLC using OCLC number matching, direct 1:1 matches are sometimes incorrect or mismatched.

The goal of this tool is to determine if we can match OCLC numbers and local records with high precision by combining the **OCLC number with secondary bibliographic data** (Title, Author, Edition, and Publication Date).

## How It Works
To make the matching process robust, the script reads raw MARC exchange/text files and performs a multi-field comparison rather than relying solely on the OCLC number:
1. **Primary Key Match:** It links the local record to the OCLC record via the OCLC control number.
2. **Bibliographic Validation:** It compares additional metadata fields between the two sources to confirm they represent the same entity:
   * **Title** matching (with text normalization)
   * **Author** alignment
   * **Edition** verification
   * **Publication Date** proximity/validation
## Matching Logic & Thresholds
To ensure precision, the script normalizes text and uses a **word-overlap percentage** rather than requiring exact matches. 

### 1. Data Normalization
* **Titles & Authors:** Converted to lowercase; diacritics, punctuation, special characters, and excess whitespace are completely removed.
* **Publication Dates:** Extracts the first valid 4-digit year found in the statement. 
* **Editions:** Heavily standardized to catch manifestation-level variations (e.g., lowercase conversion, standardizing ordinals like `second` or `2nd` $\rightarrow$ `2`, and truncating terms like `revised` $\rightarrow$ `rev`).

### 2. Match Rules & Decision Thresholds
The script calculates similarity based on the proportion of shared, normalized keywords between the two records:
* **Title Match:** Requires at least **60% keyword overlap** to pass.
* **Author Match:** Requires at least **50% keyword overlap** to pass.
* **Edition (Hard Rule):** Must match exactly. If the normalized editions conflict (e.g., `2nd ed` vs `3rd ed`), the script **instantly stops** and rejects the match.
* **Publication Date:** Treated as informational for cross-validation rather than a strict gatekeeper.

## Prerequisites
This script relies entirely on standard Python libraries. No external package installations (like `pip`) are necessary!
* **`re`** (for regex-based MARC tag parsing)
* **`csv`** (for writing the final report)
* **`unicodedata`** (for text normalization and character flattening)
* **`collections`** (specifically `defaultdict` for optimized record mapping)

## Required Input Format
The script processes raw **MARC text files (.mrk format)**. You must upload two separate files into your environment:
1. **`ALMA.mrk`** – The exported MARC text records from your local Alma catalog lacking publication dates.
2. **`OCLC.mrk`** – The corresponding MARC text records retrieved from OCLC for cross-validation.

## How to Run in Google Colab
1. Open a new notebook in [Google Colab](https://google.com).
2. Copy and paste the script code into a code cell.
3. Click the **Files** icon (folder shape) on the left sidebar of Colab.
4. Upload your **`ALMA.mrk`** and **`OCLC.mrk`** files directly into that file area.
5. Click the **Play** button on the code cell to execute.

## Output Files
After running, the script will generate a single, comprehensive evaluation report in your Colab files sidebar for you to download:
* **`comparison_results.csv`** – A structured matrix showing the local identifiers, matched OCLC numbers, and validation scores across text fields to help you safely isolate high-confidence matches.

## Contributing
Contributions are welcome! Please open an issue or submit a pull request if you want to add features (such as fuzzy matching string thresholds, support for binary `.mrc` files, or direct API workflows).
