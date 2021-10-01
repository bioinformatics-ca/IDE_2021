# Module 4: Phylogenetics and Phylodynamics

```bash
# Activate environment
conda activate augur

# Prepare working directory
cp -r ~/CourseData/IDE_data/module4/ workspace/
mkdir workspace/module4/analysis
cd workspace/module4/analysis
pwd
# /home/ubuntu/workspace/module4/analysis

# Do analysis
# Time: 2 minutes
augur filter --metadata ../data/metadata.csv --sequences ../data/cog_all.fasta.xz --sequence-index ../data/cog_all.index --output filtered.fasta --output-metadata filtered.tsv

# Time: 7 minutes
augur align --nthreads 4 --sequences filtered.fasta -o alignment.fasta --reference-sequence ../data/reference/MN908947.fasta

# Time: 40 seconds
augur tree --nthreads 4 --substitution-model GTR --alignment alignment.fasta -o tree.subs.nwk

# Time: 5 minutes
augur refine --alignment alignment.fasta --tree tree.subs.nwk --metadata filtered.tsv --timetree --divergence-units mutations --output-tree tree.time.nwk --output-node-data refine.node.json

# Time: 1 second
augur export v2 --tree tree.time.nwk --node-data refine.node.json --title "Example Augur" --output analysis-package.json
```

View in <https://auspice.us> (drag-and-drop `analysis-package.json` and then `filtered.tsv`)

To compare to **Figure 4** search for:
* **Figure 4.a**: `Scotland/CVR207`
* **Figure 4.b**: `Scotland/CVR271`
* **Figure 4.c**: `Scotland/CVR44`

You can select to colour leaves by `travel_hx` (Travel history).

To compare to **Figure 5**, colour leaves by `uk_lineage` and search for the appropriate lineages associated with each subtree in the figure (e.g., `UK40`).
