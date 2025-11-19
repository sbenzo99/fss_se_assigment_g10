### 2.1 Complexity Hotspots (Task 2)

For Task 2 we selected **Lines of Code (LoC)** and **Cyclomatic Complexity (CC)** as our complexity measures.  
To visualize the complexity hotspots, we combined two complementary perspectives:

1. **File-level analysis** — identify the most complex individual `.py` files.
2. **Directory-level analysis** — understand how complexity is distributed across the structure of the repository.

---

## File-level hotspots

We first built a **LoC–CC scatterplot**, where each point represents a Python file.  
In this visualization (`complexitylocvscc.png`) we highlighted:

- **Top-10 CC** in **orange** (`hotspot_type = "top CC"`),
- **Top-10 LoC** in **green** (`hotspot_type = "top LoC"`),
- Files that appear in *both* top-10 lists in **dark blue** (`hotspot_type = "both"`).

This allows us to immediately see:

- which files are *outliers* with extremely high LoC and/or CC,
- which files are simultaneously long and structurally complex,
- how the two types of hotspots overlap (≈40% overlap in our case).

Below is the table with the highlighted files:

| short_name                    | file_path                                                             | loc  | cc   | hotspot_type |
|------------------------------|----------------------------------------------------------------------|-----:|-----:|--------------|
| trainer.py                   | src/transformers/trainer.py                                          | 4005 | 1255 | both         |
| modeling_utils.py            | src/transformers/modeling_utils.py                                   | 3950 | 1238 | both         |
| test_modeling_common.py      | tests/test_modeling_common.py                                        | 3431 |  878 | both         |
| modeling_seamless_m4t_v2.py  | src/transformers/models/seamless_m4t_v2/modeling_seamless_m4t_v2.py | 2808 |  533 | both         |
| tokenization_utils_base.py   | src/transformers/tokenization_utils_base.py                          | 2320 |  613 | top CC       |
| utils.py                     | src/transformers/generation/utils.py                                 | 2481 |  612 | top CC       |
| testing_utils.py             | src/transformers/testing_utils.py                                    | 2118 |  600 | top CC       |
| integration_utils.py         | src/transformers/integrations/integration_utils.py                   | 1797 |  575 | top CC       |
| modeling_tf_utils.py         | src/transformers/modeling_tf_utils.py                                | 2048 |  571 | top CC       |
| import_utils.py              | src/transformers/utils/import_utils.py                               | 1827 |  520 | top CC       |
| test_tokenization_common.py  | tests/test_tokenization_common.py                                    | 3541 |  506 | top LoC      |
| test_utils.py                | tests/generation/test_utils.py                                       | 3746 |  497 | top LoC      |
| test_trainer.py              | tests/trainer/test_trainer.py                                        | 4973 |  470 | top LoC      |
| modeling_qwen2_5_omni.py     | src/transformers/models/qwen2_5_omni/modeling_qwen2_5_omni.py        | 2871 |  424 | top LoC      |
| modeling_qwen3_omni_moe.py   | src/transformers/models/qwen3_omni_moe/modeling_qwen3_omni_moe.py    | 2967 |  421 | top LoC      |
| modular_qwen2_5_omni.py      | src/transformers/models/qwen2_5_omni/modular_qwen2_5_omni.py         | 2753 |  362 | top LoC      |

**Observations:**

- The main cloud of points contains files with low LoC and CC, while hotspot files are clear **outliers**.
- Several *LoC hotspots* are under `tests/`, which is expected: complex test scenarios often require long scripts (mocking, setup, edge cases).
- *CC hotspots* mostly occur inside `src/`, including several `*_utils.py` files — a common pattern, since utility modules tend to accumulate heterogeneous logic and more branching.
- Files that are simultaneously top-10 in both metrics are the strongest indicators of potential refactoring candidates.

The scatterplot (`complexitylocvscc.png`) therefore provides a precise view of complexity at the level of individual files.

---

## Directory-level hotspots

We then moved to a **higher-level structural view**.  
For every file we propagated LoC and CC upward through the directory tree and computed, for each directory node at depth 1 and 2:

- **Average LoC per file (`avg_loc`)**,  
- **Average CC per file (`avg_cc`)**.

The resulting visualization (`complexitytree.png`) uses two visual channels:

- **Node size ∝ average LoC**,  
- **Node color ∝ average CC** (`viridis` colormap).

We limited the depth to 2 to avoid visual clutter and to keep the interpretation clear.

The tree layout allows us to see complexity not only at the file level but *structurally*, across the architecture of the project.

**What the tree reveals:**

- At **depth 1**, CC is fairly homogeneous, while LoC varies more across directories.  
  For example, `templates/` shows a large average LoC.
- At **depth 2**, the complexity becomes *much* more heterogeneous:  
  directories such as `tests/peft_integration` stand out with **high average CC** and **high average LoC**, marking them as clear structural hotspots.
- This aggregation provides insights that the file-level scatterplot cannot:  
  it highlights **which branches of the repository** are the most complex as a whole.
- Potential improvements (e.g., log-scaling LoC or CC averages) could make smaller variations even clearer, but for the assignment this level of detail is sufficient.

Overall, the directory-level visualization is a powerful complement to the file-level scatterplot.  
It provides a top-down view of structural complexity, helping identify entire subsystems or directory branches that may require refactoring, not only individual files.


### 2.2 CC vs LoC – Part III

To understand the correlation between our two selected complexity measures (LoC and CC), we first visualized all `.py` files in a scatterplot with a fitted regression line:

- **Figure:** `complexitycorrelation.png` (CC vs LoC)

We then computed two correlation coefficients:

#### **Pearson correlation (r = 0.92)**  
- Pearson evaluates the **linear relationship** between two variables using their raw values.  
- It ranges in \[-1, 1\]:  
  - **+1** → perfect positive linear correlation  
  - **0** → no linear correlation  
  - **–1** → perfect negative linear correlation  
- A value of **0.92** indicates a **very strong positive** linear relationship: in this repository, files with higher LoC tend to have proportionally higher CC.  
- Pearson is sensitive to outliers, which is relevant given the large-file outliers visible in the scatterplot.

#### **Spearman correlation (ρ = 0.95)**  
- Spearman evaluates **monotonic** relationships by ranking the values of both variables.  
- It shares the same \[-1, 1\] scale as Pearson.  
- A value of **0.95** suggests a **very strong positive monotonic** relationship: as LoC increases, CC almost always increases as well, regardless of linearity.  
- This metric is more robust to outliers and confirms the strength of the relationship.

To further validate this trend, we grouped all files into four LoC **quartiles** (“small”, “medium”, “large”, “xlarge”) and computed the mean CC for each group:

- **Figure:** `complexityquartiles.png` (Mean CC by LoC quartile)

The quartile analysis shows a clear progression:
- **small** LoC → very low CC  
- **medium** LoC → higher CC  
- **large** LoC → significantly higher CC  
- **xlarge** LoC → highest CC by a large margin  

This monotonic increase strongly agrees with the Spearman result.

---

### **Conclusion**

Both statistical measures and visualizations support the same conclusion:

> **Files with more lines of code tend to have higher cyclomatic complexity.**

This is also intuitive: longer files typically contain more branches, conditions, and logical paths, which naturally increases the cyclomatic complexity of the code.

This is also confirmed by the second plot. This was constructed by dividing the files according to the distribution of LoC values into respective quartiles. Each quartile was then categorised as small, medium, large, and very large. For each quartile, we calculated the average CC and plotted it in the bar chart. A similar trend to that mentioned above can be seen (see 'complexityquartiles.png').


## 2.3 Defects vs Complexity — Part IV

To investigate whether more complex files tend to be more defective, we first computed a defect count for each Python file in the repository.  
A commit was labelled as a bug-fix commit if its message contained one of the following keywords (intentionally broad)

fix, bug, patch, hotfix, error, issue

For every bug-fix commit, we extracted the list of modified ⁠ .py ⁠ files using

git show --name-only <sha>

Every time a file appeared in such a commit, its defect counter was incremented.  
The resulting defects-per-file table was then merged with the dataset containing LoC and CC, producing a single aligned dataframe.

We evaluated correlation using two measures  
=> Pearson correlation evaluates linear relationships  
=> Spearman correlation evaluates monotonic relationships using ranked values  

Two scatterplots (defectsvscc.png and defectsvsloc.png) were generated to visually inspect the relationships.

---

### Defects vs Cyclomatic Complexity (CC)

Pearson r => 0.67  
Spearman ρ => 0.56  

Both values show a moderate positive correlation.  
This indicates that files with higher CC tend to appear more often in defect-related commits.  
The scatterplot confirms a clear upward trend despite some noise.

---

### Defects vs Lines of Code (LoC)

Pearson r => 0.57  
Spearman ρ => 0.60  

Again, we observe a moderate positive correlation.  
Since LoC and CC are themselves strongly correlated, it is expected that both metrics display similar correlations with defect counts.

---

### Interpretation

The results support the claim that files with higher complexity tend to be more defective, even if the relationship is not extremely strong.  
The correlation is weaker than the nearly collinear relationship between LoC and CC, which is expected because file size naturally increases the likelihood of being modified and therefore appearing in bug-fix commits.

A more refined analysis could normalise defect counts by file size (for example defects per 1000 LoC) to reduce the size-bias.  
Even without such normalisation, the current evidence clearly indicates a positive association between complexity and defect propensity within this repository.



### 2.4 Use of Generative AI (Task 2)

For Task 2 (Complexity Analysis), we used a Generative AI assistant strictly as a tutor, not as an automated code generator. All final code, analysis decisions, and interpretations were written and validated by us. Below we describe how AI was used and provide examples of the prompts.

---

## Environment and setup

We used AI to clarify how to:
- create and activate a Python 3.12 virtual environment,
- generate a `requirements.txt` file,
- check installed package versions,
- update the local repository to the required `transformers` release tag.

**Example prompts:**
- “How do I create and activate a Python 3.12 virtual environment and export a requirements file?”
- “How can I checkout a specific release tag in a local clone of Hugging Face Transformers?”

---

## Understanding APIs and complexity metrics (LoC, CC)

We asked AI for explanations of:
- how the `lizard` library exposes file-level metrics,
- what `analysis.nloc` represents,
- how to sum cyclomatic complexity across functions.

The implementation of the metric extraction function was written by us.

**Example prompts:**
- “How do I compute per-file LoC and Cyclomatic Complexity using `lizard.analyze_file`?”
- “How do I access the list of functions and their CC values?”

---

## Improving the visualization of complexity hotspots

AI was used to help improve the presentation of the results.  
Initially we only had a raw CC–LoC scatter plot; we then requested suggestions on:

- how to extract the top-10 files by CC and by LoC,
- how to combine the lists and highlight overlaps,
- how to display a summary table (`hotspots_table`) with annotated labels.

The code structure and final visualization were produced by us.

**Example prompts:**
- “How can I annotate the top-10 most complex files in a scatter plot?”
- “How can I print a table with top CC and top LoC files side by side?”

---

## Statistical interpretation (Pearson and Spearman)

We used AI as a tutor to understand the theory behind:

- difference between Pearson and Spearman correlations,
- how Spearman can be computed using ranked variables,
- how to visualize complexity–defect trends clearly.

**Example prompts:**
- “What is the conceptual difference between Pearson and Spearman correlation?”
- “Is it correct to compute Spearman by ranking values and applying Pearson?”
- “How can I visualize correlations between CC, LoC, and defects?”

All statistical computations, regression plots, and interpretations were implemented by us, with AI providing conceptual clarification or visualization suggestions.

---

## Tree-based aggregation and directory-level visualization

The hierarchical directory-tree visualization was conceptually designed entirely by us.  
This was the most technically difficult part of Task 2, since it required:

- aggregating file-level metrics upward into directory nodes (depth 1, 2, 3),
- computing average and total values across levels,
- representing two metrics simultaneously (size = avg LoC, color = avg CC),
- positioning, scaling, and centering nodes across depth levels.

AI was used only for **technical guidance** on how to implement the aggregation and plotting mechanics, not for designing the idea or the visualization structure itself. Concretely, AI helped with:

- clarifying how to propagate file metrics up the directory tree,
- understanding how to compute averages and totals per depth level,
- centering nodes vertically within each depth,
- scaling marker sizes and colors appropriately,
- adjusting the legend placement and size,
- implementing alphabetical sorting within each level for readability.

All conceptual ideas — the tree layout, metric propagation logic, depth-based recursion, and the dual encoding of complexity — were entirely developed by us.

**Example prompts:**
- “How can I aggregate metrics per directory level starting from file paths?”
- “How can I display two variables (average LoC and average CC) in a single hierarchical plot?”
- “How do I center node positions in matplotlib?”
- “How can I enlarge and reposition a legend?”
- “How do I alphabetically sort nodes at each depth for a cleaner visualization?”

All design choices and the final implementation were written and validated independently.

---
