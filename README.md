# -VCF-Preparation-and-Filtering-for-Popscle-pileup--Freemuxlet-Demultiplexing

# 🧬 VCF Preparation and Filtering for Freemuxlet Demultiplexing

## 📋 Overview
Prepare a VCF for Freemuxlet by filtering common SNPs, intersecting with exons, and aligning order with BAM.

---

## ✅ 1. Filter Common SNPs (MAF ≥ 0.05)
```bash
singularity exec -B /mnt/nfs:/mnt/nfs /localdata/apps/demuxafy/Demuxafy.sif \
  bcftools filter --include 'MAF>=0.05' -Oz \
  -o common_maf0.05.vcf.gz \
  ALL.wgs.phase3_shapeit2_mvncall_integrated_v5c.20130502.sites.vcf
```

---

## 📁 2. Create a Cleaned Exon BED
**a.** Download UCSC exon BED (hg38)  
**b.** Clean chromosomes:
```bash
awk '{ if($1~/^[0-9XYM]/) $1="chr"$1; print }' hg38exonsUCSC.bed > hg38exonsUCSC_chr_cleaned.bed
```
Or:
```bash
sed 's/^/chr/' hg38exonsUCSC.bed > hg38exonsUCSC_chr_cleaned.bed
```

---

## 🧬 3. Intersect SNPs with Exons
```bash
singularity exec -B /mnt/nfs:/mnt/nfs /localdata/apps/demuxafy/Demuxafy.sif \
  bedtools intersect -header \
  -a common_maf0.05.vcf.gz \
  -b hg38exonsUCSC_chr_cleaned.bed \
  > common_maf0.05_ready_for_freemux.vcf
```

---

## 🧊 4. Compress & Index
```bash
singularity exec -B /mnt/nfs:/mnt/nfs /localdata/apps/demuxafy/Demuxafy.sif \
  bgzip common_maf0.05_ready_for_freemux.vcf

singularity exec -B /mnt/nfs:/mnt/nfs /localdata/apps/demuxafy/Demuxafy.sif \
  tabix -p vcf common_maf0.05_ready_for_freemux.vcf.gz
```

---

## 🔄 5. Sort VCF to Match BAM
```bash
singularity exec -B /mnt/nfs:/mnt/nfs /localdata/apps/demuxafy/Demuxafy.sif \
  bash sort_vcf_same_as_bam.sh \
  /mnt/nfs/.../gex_possorted_bam.bam \
  common_maf0.05_ready_for_freemux.vcf.gz \
  z > common_maf0.05_ready_for_freemux_sorted.vcf.gz
```

✅ Now VCF contigs align exactly with BAM.

---

## 📊 Final Summary

- **SNPs after MAF filter**: 4,034,682  
- **SNPs overlapping exons**: 509,551  
- **Final VCF**: `common_maf0.05_ready_for_freemux_sorted.vcf.gz`  
- **BED file**: `hg38exonsUCSC_chr_cleaned.bed`  
- **Rationale**: MAF filtering, exon focus, and contig matching ensure smooth Freemuxlet demultiplexing  

---

## 📚 Sources

- **Demuxafy VCF Preparation Tutorial**:  
  https://demultiplexing-doublet-detecting-docs.readthedocs.io/en/latest/DataPrep.html

- **sort_vcf_same_as_bam.sh script** (Aerts Lab):  
  https://github.com/aertslab/popscle_helper_tools/blob/master/sort_vcf_same_as_bam.sh
