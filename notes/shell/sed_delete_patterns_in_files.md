# sed: Delete Lines or Patterns from Files

## Delete lines containing a pattern
```bash
sed -i '/pattern/d' file
```

### Example
```bash
# Remove comment lines from a VCF file
sed -i '/^#/d' file.vcf

# Before: ##fileformat=VCFv4.1
#         #CHROM  POS  ID  REF  ALT
#         1       752566  .  A  G
# After:  1       752566  .  A  G
```

## Delete a specific line
```bash
sed -i '3d' file
```

### Example
```bash
# Remove the header line from a TSV
sed -i '1d' results.tsv

# Before: sample  score  pvalue
#         HG001   2.3    0.01
# After:  HG001   2.3    0.01
```

## Delete a range of lines
```bash
sed -i '2,5d' file
```

### Example
```bash
# Remove VCF meta-information block (lines 2 to 10)
sed -i '2,10d' file.vcf
```

## Delete blank lines
```bash
sed -i '/^$/d' file
```

### Example
```bash
# Clean up empty lines in a Snakemake rule file
sed -i '/^$/d' workflow/rules/01_preprocessing.smk
```

## Verification
```bash
grep -rn "pattern" dir/
```
- Empty output = all matched lines successfully deleted ✓

## Notes
`d` deletes the entire line — not just the matched pattern within it.
To remove only a pattern within a line (not the whole line) → use substitution with empty string:
```bash
sed -i 's/pattern//g' file
```