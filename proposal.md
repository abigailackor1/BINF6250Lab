BINF6400 Project Proposal
Author: Abigail · Fall 2026

1. Project Title
The Phylogeny of Chess Openings: Sequence Alignment and Tree-Building on Chess Games

2. Research Question
Question
Can pairwise sequence alignment of chess opening moves, combined with distance-based tree-building, recover the established families of chess openings as defined by the Encyclopaedia of Chess Openings (ECO)?

Why It Matters
Sequence alignment and phylogenetic tree-building are usually evaluated on biological data, where the true relationships are often uncertain. Chess openings offer a sequence dataset with a well-documented classification system, 
so the methods can be tested against a known structure. The project is innovative in two ways. 
First, it adapts a biological scoring idea, the substitution matrix, into a chess-aware scoring scheme that gives partial credit for similar moves. Second, 
it uses transpositions (the same position reached through a different move order) as a natural test of a known weakness of alignment, which assumes order is preserved, much like genomic rearrangements challenge alignment in biology.

3. Algorithm and Algorithm Class
Algorithm Class
The primary class is dynamic programming, specifically pairwise alignment (week 6). The secondary class is tree-building algorithms, specifically distance-based phylogeny (week 7).

Algorithms to Implement
The project will implement Needleman-Wunsch global alignment to score the similarity of every pair of games, and UPGMA and neighbor-joining to build trees from the resulting distance matrix.

Justification
A chess game is an ordered sequence of discrete symbols (moves), which is the exact input that pairwise alignment is designed for. Dynamic programming finds the optimal alignment efficiently while allowing 
for mismatches and gaps, which captures how two openings can share a main line and then diverge. Chess openings are also organized hierarchically, with broad families branching into named variations, so a tree is a natural 
way to represent their relationships. Comparing UPGMA, which assumes a constant rate of change, with neighbor-joining, which does not, adds a meaningful methodological comparison.

4. Data Plan
Description of Data
Source. The Lichess open database (database.lichess.org), which publishes every rated game played on Lichess, organized by month.

Data type. Chess games in PGN (Portable Game Notation), a plain-text format. Each game includes the move sequence in standard algebraic notation, along with metadata such as player ratings, time control, result, ECO code, and opening name. 
The project will convert each game's opening into a sequence of move tokens, for example [e4, c5, Nf3, d6], and treat each move as one character.

Licensing and access. The Lichess database is freely downloadable without an account and is released under a Creative Commons CC0 (public domain) license, so there are no restrictions on use. Monthly files are compressed and can be very large for recent years, 
so the project will either use smaller early-year files or stream and sample from a recent file rather than loading it fully. PGN parsing will use the python-chess library.

Prototype Data Plan
Minimal dataset. The first prototype will be a hand-built set of about 15 to 20 games covering a few well-known openings, for example several Sicilian Defense variations, several Queen's Gambit lines, and a few Italian Games, plus at 
least one pair of games that reach the same position through a transposition. Because these openings are known in advance, the correct grouping is obvious, which makes it easy to spot bugs in the alignment or tree code. Alignments for a few pairs will also be checked by hand.

Relation to the final dataset. The prototype uses the same format (opening move sequences with ECO labels) as the full dataset, so the same code runs on both. Once the pipeline produces correct trees on the prototype, it will be scaled to a sample of roughly 300 to 500 Lichess games 
balanced across the five ECO volumes (A through E), filtered to standard rated games and truncated to the first 12 to 20 moves per side.

5. Success Criteria
Expected Outputs
The project is expected to produce a working Python implementation of Needleman-Wunsch alignment with two scoring schemes (simple and chess-aware), pairwise distance matrices for the sampled games, UPGMA and neighbor-joining trees visualized with branches colored by ECO volume,
and a table of clade purity scores comparing methods, scoring schemes, and baselines.

Definition of Success
The project is successful if the alignment-based trees group games by ECO volume clearly better than random trees and better than a simple prefix-matching baseline (which only counts how many opening moves two games share before they differ). A secondary sign of success is identifying specific, 
explainable error patterns, such as transpositions, rather than unexplained noise.

Reasonableness Checks
Known small example. The hand-built prototype has an obvious correct tree. If the code doesn't group the Sicilian games together on that set, something is wrong.

Comparison to standard tools. Alignment scores will be checked against Biopython's PairwiseAligner by encoding each distinct move as a single character, so both implementations align the same strings with the same scoring. Trees will be compared against SciPy's average-linkage clustering (which is equivalent to UPGMA) 
and Biopython's DistanceTreeConstructor, which supports both UPGMA and neighbor-joining.

Hand verification. A few alignments from the prototype will be worked out by hand with the dynamic programming table and compared to the code's output.

6. Pitfall Scan
Data Pitfall: Imbalance and Noise Across Openings
Why it's realistic. Some openings are far more popular than others. Games starting with 1. e4 (ECO volumes B and C) are much more common than flank openings (volume A), so a random sample would be dominated by a few families. Games between low-rated players also often leave established opening lines early, which adds noise.

Detection and mitigation. The ECO distribution of the sample will be checked before analysis. Sampling will be stratified so each ECO volume is equally represented, and a minimum rating filter will reduce noisy, off-book openings. Results can also be compared across rating levels to measure how much noise matters.

Algorithmic Pitfall: Runtime and Parameter Sensitivity
Why it's realistic. Aligning every pair of games grows quickly. With 500 games, there are about 125,000 pairs, and each alignment fills a table proportional to the product of the two sequence lengths. In pure Python this could take hours. In addition, the results may depend heavily on the chosen match, mismatch, and gap scores, 
and converting alignment scores into distances requires care, since raw scores depend on sequence length and can be negative.

Detection and mitigation. Runtime will be measured on the prototype and on samples of increasing size before running the full dataset. Mitigations include truncating openings to a fixed length, using NumPy for the alignment table, caching results, and keeping the sample size modest. For parameter sensitivity, the analysis will be 
repeated over a small range of gap and mismatch penalties to check whether the conclusions hold. Scores will be normalized by sequence length before conversion to distances, and the distance matrix will be checked for invalid values such as negative entries.

Evaluation Pitfall: Partially Circular Ground Truth and Misleading Metrics
Why it's realistic. Lichess assigns ECO codes by matching each game's opening moves to a reference list. Because the labels are themselves based on move sequences, some agreement between the trees and ECO is built in, which could make results look better than they are. Clade purity can also be misleading when groups differ in size, 
and tuning the chess-aware scoring scheme to maximize agreement with ECO risks overfitting.

Detection and mitigation. The prefix-matching baseline addresses circularity directly: it measures how much agreement comes simply from shared opening moves, so alignment only counts as successful if it improves on that. Purity will be reported per ECO group rather than only as an overall average, so large groups don't hide poor performance on small ones. 
Scoring parameters will be tuned on one subset of games and evaluated on a separate held-out subset.

7. Planned Repository Structure (Initial Sketch)
chess-opening-phylogeny/
├── README.md                  # Project overview, setup, and how to run
├── requirements.txt           # Python dependencies
├── data/
│   ├── raw/                   # Downloaded Lichess PGN files (not committed)
│   ├── prototype/             # Hand-built test games with known openings
│   └── processed/             # Cleaned, sampled move sequences with ECO labels
├── src/
│   ├── parse_games.py         # PGN parsing, filtering, and sampling
│   ├── scoring.py             # Simple and chess-aware scoring schemes
│   ├── alignment.py           # Needleman-Wunsch implementation
│   ├── distance.py            # Score-to-distance conversion, distance matrix
│   ├── trees.py               # UPGMA and neighbor-joining implementations
│   ├── baselines.py           # Prefix-matching and random-tree baselines
│   └── evaluate.py            # Clade purity and comparison metrics
├── tests/
│   ├── test_alignment.py      # Hand-checked alignment examples
│   ├── test_trees.py          # Trees on the prototype set
│   └── test_against_tools.py  # Comparison to Biopython and SciPy
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_prototype_results.ipynb
│   └── 03_full_analysis.ipynb
└── results/
    ├── figures/               # Tree visualizations
    └── tables/                # Purity scores and comparisons
8. Generative AI Disclosure
Claude (Anthropic) was used during the planning stage of this project to brainstorm candidate project ideas that matched the course algorithms,
to discuss how chess openings could be framed as a sequence alignment and phylogeny problem, and to help draft and organize the text of this proposal. All project decisions, and the final wording of this proposal, were reviewed by the author.
Generative AI use during implementation will be documented in the repository README.
