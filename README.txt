CSL7610 AI Assignment - Classical Reasoning System for Logical Entailment
Nilesh Mishra (P25CS0006)

Files
-----
reasoning_agent.ipynb   complete implementation, experiments, final accuracy and inference time
report.pdf              representation, algorithms, experiments, reasoning traces
submission.csv          predictions (columns: id, label with values True/False)
README.txt              this file

Requirements
------------
Python 3.9 or newer. The reasoning engines use only the standard library.
The packages "datasets" and "pandas" are needed only to load the public data.
The notebook installs them itself if they are missing; they can also be
installed with:  pip install datasets pandas
No Hugging Face login is needed (FOLIO is read from the authors' public GitHub copy).

How to run
----------
Run all cells from a fresh kernel (Kernel > Restart & Run All), or from a terminal:

    jupyter nbconvert --to notebook --execute reasoning_agent.ipynb --inplace --ExecutePreprocessor.timeout=-1

Settings at the top of the data-loading section:
    LIMIT       None = full splits (about 3-4 minutes in total); a number such as 5000
                gives a quick run on a random sample
    TEST_PATH   path of the test file (see below)

Generating submission.csv for a test file
-----------------------------------------
Set TEST_PATH to the test file and run the notebook, or call directly:

    make_submission("test.csv", "submission.csv")

The test file can be CSV, JSON or JSONL with an "id" column and one column for the
knowledge base and one for the query. Common column names are recognised
(context / premises / theory and statement / question / query / conclusion).
FOL columns (premises-FOL, conclusion-FOL) are used when present.
If TEST_PATH does not exist, the notebook writes submission.csv for the public data instead.

Input formats
-------------
The system accepts both natural language and logic:
  - natural language (RuleTaker style text, CounterBench causal questions)
  - first-order logic, one formula per line, ";" or ". " as separators, in any of:
        Unicode   forall/exists symbols, ¬ ∧ ∨ → ↔ ⊕      e.g.  ∀x (Young(x) ∧ ¬Big(x) → Smart(x))
        ASCII     forall x. (Young(x) & ~Big(x) -> Smart(x)),  |, <->, =>
        Prolog    smart(X) :- young(X), kind(X).
The query is given in the same notation, e.g.  Smart(bob)

Method (details in report.pdf)
------------------------------
A router sends each problem to one of three engines. No generative model is used.
  1. Facts and if-then rules (Horn form) -> stratified forward chaining with
     unification and negation as failure (closed world)
  2. General first-order logic -> resolution refutation (CNF, Skolemisation,
     unification with occurs check, set of support, unit preference, factoring)
  3. Causal statements and counterfactual questions -> structural causal model
     (abduction, intervention, forward propagation)

Results on the public data (Table 1 of report.pdf)
--------------------------------------------------
Dataset                              Accuracy    Time per sample
RuleTaker test split (151,911)       99.97 %     0.075 ms
FOLIO validation (204)               98.04 %     19.3 ms
CounterBench held-out half (600)     94.17 %     0.21 ms

RuleTaker given as logic (151,911)   99.97 %     0.72 ms   (check of the logic-input route)

Overall (average over the three datasets)
    Overall accuracy        (99.97 + 98.04 + 94.17) / 3        = 97.39 %
    Overall inference time  (0.075 + 19.3 + 0.21) ms / 3       = 0.00653 s per sample

Pooled over all 153,315 queries: 99.93 % accuracy, 15.5 s total inference time.
