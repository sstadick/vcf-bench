
# task

the timed task will be to:
iterate over each row in a VCF, extract a value from the INFO field (an
integer), add it to a vector and report the mean at the end.
The exact task is useless, but it gives an idea of relative performance.

# Summary

Individual runs are below. `rust-htslib` was compiled with --release and `hts-nim` was compiled with -d:danger and uses libdeflate. C htslib uses libdeflate and is compiled with -O2.
zig was compiled with -Drelease-fast.

Note that tools using libdeflate are substantially faster 

### BCF

| Tool  | Time   | File |
|-------|--------|------|
| cyvcf2 | 8.15s | BCF  |
| cyvcf2 (libdeflate) | 3.9s | BCF  |
| pysam | 3.8s | BCF  |
| rust-htslib | 5.8s | BCF |
| hts-nim | 3.5s | BCF |
| hts-zig | 3.5s | BCF |
| C htslib | 3.5s | BCF |

### VCF

(note that these times can be improved to nearly match the `BCF` speeds above using [bcf_hdr_set_samples](https://github.com/samtools/htslib/blob/238fe32d8c7aa05d3ac75d2249c61d8e268be58f/htslib/vcf.h#L323-L345) when possible).

| Tool  | Time   | File |
|-------|--------|------|
| pyvcf | 16m49s | VCF  |
| cyvcf2 | 29s   | VCF  |
| cyvcf2 (libdeflate) | 20s   | VCF  |
| pysam | 28s   | VCF  |
| rust-htslib | 22s | VCF |
| hts-nim | 18s | VCF |
| hts-zig | 18s | BCF |
| C htslib | 18s | VCF |


# testing data

```
wget -O - http://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000G_2504_high_coverage/working/20201028_3202_raw_GT_with_annot/20201028_CCDG_14151_B01_GRM_WGS_2020-08-05_chr1.recalibrated_variants.vcf.gz \
    | zcat - \
    | head -30000 \
    | bgzip -c > 1kg.chr1.subset.vcf.gz
```

# Timings

## python pyvcf

```
$ time python python-pyvcf/read.py 1kg.chr1.subset.vcf.gz 
5992.641098163443

real    16m49.068s
user    16m44.487s
sys     0m0.924s
```

## python cyvcf2


### BCF

```
$ time python python-cyvcf2/read.py 1kg.chr1.subset.bcf    
5992.64

real    0m8.154s
user    0m8.168s
sys     0m0.268s
```

### VCF

```
$ time python python-cyvcf2/read.py 1kg.chr1.subset.vcf.gz 
5992.64
real    0m29.703s
user    0m29.715s
sys     0m0.260s
```

### VCF with subset samples

```
$ time python python-cyvcf2/read.py 1kg.chr1.subset.vcf.gz 
5992.64

real    0m9.927s
user    0m9.938s
```

## rust-htslib


### BCF

```
$ time rust-htslib/target/release/rust-rhtslib ./1kg.chr1.subset.bcf    
sum: 158906864, avg:5992

real    0m5.888s
user    0m5.799s
sys     0m0.088s
```

### VCF
```
$ time rust-htslib/target/release/rust-rhtslib ./1kg.chr1.subset.vcf.gz 
sum: 158906864, avg:5992

real    0m22.122s
user    0m22.002s
sys     0m0.116s
```

## hts-nim

### BCF

```
$ time ./nim-hts-nim/read ./1kg.chr1.subset.bcf 
5992.641098163443

real    0m3.579s
user    0m3.482s
sys     0m0.097s
```

### VCF

```
$ time ./nim-hts-nim/read ./1kg.chr1.subset.vcf.gz 
5992.641098163443
real    0m18.346s
user    0m18.276s
sys     0m0.068s
```

## C htslib


### BCF

```
$ time ./c-htslib/read 1kg.chr1.subset.bcf 
5992.64

real	0m3.528s
user	0m3.463s
sys	0m0.064s
```

### VCF

```
$ time ./c-htslib/read 1kg.chr1.subset.vcf.gz 
5992.64

real	0m18.103s
user	0m18.022s
sys	0m0.076s
```

## hts-zig

### BCF

```
$ time ./zig-out/bin/hts-zig ../1kg.chr1.subset.bcf 
mean:5.992641098163443e+03

real	0m3.546s
user	0m3.465s
sys	0m0.080s
```

### VCF

```
$ time ./zig-out/bin/hts-zig ../1kg.chr1.subset.vcf.gz 
mean:5.992641098163443e+03

real	0m18.091s
user	0m18.011s
sys	0m0.077s
```

## python-pysam

### BCF

```
$ time python python-pysam/read.py 1kg.chr1.subset.bcf 
[E::idx_find_and_load] Could not retrieve index file for '1kg.chr1.subset.bcf'
5992.641098163443

real	0m3.866s
user	0m3.726s
sys	0m0.137s
```

### VCF

```
$ time python python-pysam/read.py 1kg.chr1.subset.vcf.gz 
[E::idx_find_and_load] Could not retrieve index file for '1kg.chr1.subset.vcf.gz'
5992.641098163443

real	0m28.743s
user	0m28.570s
sys	0m0.166s
```
