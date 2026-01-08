# ra_dgidb_query.ipynb — README

## Overview

This Jupyter Notebook (ra_dgidb_query.ipynb) demonstrates how to query drug–gene interaction data from DGIdb (Drug–Gene Interaction Database) for rheumatoid arthritis (RA) research / drug‑repurposing analysis. The notebook shows how to request interaction data (via DGIdb's API/GraphQL), inspect the returned JSON, extract relevant fields (drug name, gene name, interaction types, source databases), and save parsed results for further downstream analysis such as network visualization (e.g., Cytoscape) or integration with bioinformatics pipelines.

The included notebook output (example shown in the notebook) shows interactions returned for the drug "METHOTREXATE" and several associated genes, with interaction types (when provided) and source databases.

## Key Goals

- Query DGIdb for drug–gene interactions.
- Parse the JSON/GraphQL response into a tabular format.
- Save results to a user-friendly format (CSV / Excel) for downstream use.
- Provide a minimal reproducible workflow inside Jupyter for network-pharmacology or drug‑repurposing studies.

## Requirements

- Python 3.8+
- Jupyter Notebook / JupyterLab
- Packages (the notebook installs / uses these):
  - requests
  - pandas
  - openpyxl

(These packages appear in the notebook installation step; if you prefer a virtual environment, create and activate it before running.)

Example pip install:
```
pip install requests pandas openpyxl
```

## How to run

1. Clone the repository or download the file `ra_dgidb_query.ipynb`.
2. Launch Jupyter Notebook or JupyterLab in the folder that contains the notebook:
   ```
   jupyter notebook
   ```
3. Open `ra_dgidb_query.ipynb` and run cells in order. The first cell installs packages (it uses `!pip install ...`), and subsequent cells perform the DGIdb query and show results.
4. Inspect outputs printed by the notebook. The notebook prints the HTTP status and the returned JSON snippet (the notebook shows the "HTTP status: 200" and the JSON with `interactions.edges`).

Note: If you run the pip install cell in an already configured environment, it will usually show "Requirement already satisfied" messages as in the notebook.

## What the notebook does (cell-by-cell explanation)

The following is an inferred, stepwise explanation based on the notebook contents and printed outputs:

1. Installation cell
   - Installs runtime dependencies (requests, pandas, openpyxl).
   - This ensures the Python environment has the required libraries.

2. Query / request cell
   - Sends an HTTP request to DGIdb's interaction endpoint (likely GraphQL or REST).
   - The printed output shows:
     - "HTTP status: 200" (successful request)
     - A JSON structure with `data.interactions.edges`, where each edge has a `node` containing:
       - `drug.name`
       - `gene.name`
       - `interactionTypes` (list; may be empty; when present contains `type` and `definition`)
       - `sources` (list with `sourceDbName`)
   - Example JSON node (shortened):
     ```
     {
       "node": {
         "drug": {"name": "METHOTREXATE"},
         "gene": {"name": "DHFR"},
         "interactionTypes": [{
           "type": "inhibitor",
           "definition": "In inhibitor interactions, the drug binds ... "
         }],
         "sources": [{"sourceDbName": "ChEMBL"}, {"sourceDbName": "NCI"}]
       }
     }
     ```

3. Parsing & transformation
   - The notebook likely extracts for each edge: drug name, gene name, interaction type(s) (if any), interaction definition (if available), and sources (possibly joined into a semicolon-separated string).
   - The extracted data is then converted into a DataFrame (pandas).

4. Saving outputs
   - The notebook likely saves the results to disk in one or more formats (CSV, Excel).
   - openpyxl presence indicates Excel export is used for convenience:
     - pandas.DataFrame.to_excel(...) or .to_csv(...)

5. (Optional) Downstream export for network tools
   - The parsed table (drug — gene — interaction type) is a suitable input for network visualization in Cytoscape (edges file) or for additional network analysis.

## Typical output schema

When converted to a table, each row represents a drug–gene interaction. Typical columns:

- drug_name
- gene_name
- interaction_types — possibly a list or semicolon-separated string
- interaction_definitions — optional, text describing the interaction class
- sources — semicolon-separated source database names (e.g., PharmGKB; ChEMBL)
- raw_json (optional) — original JSON node (useful for provenance)

Example (CSV/Excel row):
```
drug_name, gene_name, interaction_types, interaction_definitions, sources
METHOTREXATE, DHFR, inhibitor, "In inhibitor interactions, the drug binds to a target and decreases its expression or activity...", NCI;ChEMBL;GuideToPharmacology
```

## Example GraphQL query (representative)

The JSON shown in the notebook suggests a GraphQL query similar to the following was used (this is a representative example — the actual notebook may construct the query string programmatically):

```graphql
query {
  interactions(query: "METHOTREXATE") {
    edges {
      node {
        drug {
          name
        }
        gene {
          name
        }
        interactionTypes {
          type
          definition
        }
        sources {
          sourceDbName
        }
      }
    }
  }
}
```

If the notebook uses REST, the fields requested and parsed will be similar but delivered by the REST schema.

## How to customize

- Change target drugs:
  - Replace the query string from "METHOTREXATE" to any drug name or iterate over a list of drug names to build a combined table.
- Batch queries:
  - If querying many drugs, batch the queries responsibly (respect DGIdb API terms and rate limits).
- Filter by source:
  - Keep only results from specific source databases (e.g., PharmGKB or ChEMBL) by filtering the `sources` content.
- Expand fields:
  - If more metadata is available from the API (e.g., PubMed references, confidence scores), include these fields when constructing the request and parsing the response.
- Export formats:
  - Use pandas to export to CSV, Excel, or JSON for downstream tools.

## Limitations & considerations

- API changes: DGIdb's API and GraphQL schema may evolve. If a query fails, check the DGIdb API documentation for updated field names and endpoints.
- Incomplete interaction types: Many returned nodes have empty `interactionTypes` lists; this is normal — not all interactions are annotated with a type.
- Data provenance: Different source databases have different curation standards. Use source annotations (sourceDbName) to assess provenance.
- Licensing & usage: Respect DGIdb's terms of use and citations when using data.
- Rate limits: If performing many queries, throttle requests and respect rate limits or contact DGIdb for bulk access options.

## Troubleshooting

- HTTP status is not 200:
  - Check network connectivity and the DGIdb endpoint URL.
  - Verify any headers or authentication required by the endpoint (DGIdb public endpoints often do not require auth for basic queries, but policies can change).
- Parsing errors:
  - Confirm the JSON keys exist; handle missing keys safely (use .get() or try/except).
  - Use defensive code to manage empty lists for `interactionTypes` and `sources`.
- Excel export errors:
  - If to_excel fails, ensure openpyxl is installed and the DataFrame does not contain problematic datatypes.

## Next steps / suggestions

- Aggregate interactions across multiple drugs for RA and build a network where nodes are drugs and genes and edges are annotated with interaction types and sources.
- Use the exported table as an edge list for Cytoscape or network analysis libraries (NetworkX / igraph).
- Integrate gene expression or GWAS data for RA to prioritize interactions relevant to patient-specific contexts (personalized medicine).
- Add caching or local DB to avoid re-querying the API repeatedly for the same drugs.

## Data source and citation

This notebook queries DGIdb (Drug–Gene Interaction Database). If you publish results generated from DGIdb data, cite DGIdb per their citation guidance: https://www.dgidb.org/ (check their website for the recommended citation).

## Licensing & attribution

- The notebook itself does not specify a license. If you plan to reuse or distribute this notebook, add a license (e.g., MIT, Apache 2.0) to the repository and make sure any redistributed DGIdb data complies with DGIdb's terms.
- Attribution: credit the original notebook author/repository owner when reusing or publishing derivative work.

## Contact / author

- Repository: Bhuvana214/ra_dgidb_query.ipynb  
- If you need clarifications about the notebook's implementation details (e.g., exact filenames used for outputs), open the notebook in Jupyter and examine the cells that perform `requests` and `pandas` operations — the code cells show the precise variable names and filenames used for saving.

---

If you'd like, I can:
- produce a cleaned README.md commit-ready file,
- extract the exact parsing & saving code from the notebook and add usage examples with concrete filenames,
- or generate a small standalone Python script that performs the same DGIdb query and writes results to CSV/Excel.
Tell me which of those you'd like next.
