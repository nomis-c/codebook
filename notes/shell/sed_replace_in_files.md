# sed: Find and Replace Strings in Files

## Simple in-place substitution
```bash
sed -i 's/old/new/g' file
```
- `-i` = edit file directly (in-place)
- `g` = all occurrences per line, not just first

## Across multiple files
```bash
grep -rl "pattern" dir/ | xargs sed -i 's/old/new/g'
```
- `grep -rl` finds all files containing pattern
- `xargs` pipes them to sed
### Example
```bash
# Remove chr prefix from Snakemake wildcards across all rules
grep -rl "chr{i}" workflow/rules/ | xargs sed -i 's/chr{i}/{i}/g'

# Before: expand("results/{sample}/chr{i}.vcf", ...)
# After:  expand("results/{sample}/{i}.vcf", ...)
```

## Verification
```bash
grep -rn "chr{i}" workflow/rules/
```
- `-n` = show line numbers
- If output is empty → all occurrences replaced successfully
- `sed: no input files` from xargs = grep found nothing = success ✓

## Notes
Complex patterns with `"`, `$`, `\` are error-prone due to escaping.
Rule of thumb: simple substitutions → sed / complex strings → edit manually