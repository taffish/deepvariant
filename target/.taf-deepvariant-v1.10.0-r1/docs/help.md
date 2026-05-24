taf-deepvariant 1.10.0-r1

DeepVariant 1.10.0 CPU runtime for germline variant calling from aligned
BAM/CRAM reads to VCF or gVCF outputs.

Usage:
  taf-deepvariant --help
  taf-deepvariant --version
  taf-deepvariant -- --version
  taf-deepvariant -- --model_type=WGS --ref=ref.fa --reads=sample.bam --output_vcf=out.vcf.gz
  taf-deepvariant run_deepvariant --model_type=WGS ...
  taf-deepvariant make_examples --helpfull

Wrapper options:
  taf-deepvariant --help       Show this TAFFISH help.
  taf-deepvariant --version    Show the TAFFISH package version.
  taf-deepvariant --compile    Print the generated runner.
  taf-deepvariant -- --help    Show upstream run_deepvariant help.

Default upstream command:
  run_deepvariant

Command mode:
  Because command_mode is enabled, the first non-option argument is treated as
  an executable inside the container. Use -- before normal run_deepvariant
  option-leading arguments, or name the executable explicitly:

    taf-deepvariant -- --model_type=WGS ...
    taf-deepvariant run_deepvariant --model_type=WGS ...
    taf-deepvariant make_examples --helpfull
    taf-deepvariant samtools --version

Common workflows:
  Show upstream version:
    taf-deepvariant -- --version
    taf-deepvariant run_deepvariant --version

  WGS germline calling:
    taf-deepvariant -- \
      --model_type=WGS \
      --ref=ref.fa \
      --reads=sample.bam \
      --regions="chr20:10000000-10010000" \
      --output_vcf=output.vcf.gz \
      --output_gvcf=output.g.vcf.gz \
      --intermediate_results_dir=deepvariant-intermediate \
      --num_shards=8 \
      --vcf_stats_report=true

  Print planned stage commands without executing the call:
    taf-deepvariant -- \
      --model_type=WGS \
      --ref=ref.fa \
      --reads=sample.bam \
      --output_vcf=output.vcf.gz \
      --num_shards=1 \
      --dry_run=true

Common options:
  --model_type=TYPE        WGS, WES, PACBIO, ONT_R104,
                           HYBRID_PACBIO_ILLUMINA, MASSEQ, or RNASEQ.
  --ref=FASTA             Indexed reference FASTA.
  --reads=BAM|CRAM        Sorted and indexed aligned reads.
  --regions=REGIONS       Optional contig, interval, or BED/BEDPE region list.
  --output_vcf=VCF.GZ     Output VCF path.
  --output_gvcf=GVCF.GZ   Optional gVCF output path.
  --num_shards=N          make_examples parallel shard count.
  --vcf_stats_report      Emit an HTML visual report.
  --logging_dir=DIR       Save per-stage logs.
  --dry_run=true          Print planned commands without running them.

Packaged commands:
  run_deepvariant, make_examples, call_variants, postprocess_variants,
  vcf_stats_report, show_examples, runtime_by_region_vis,
  multisample_make_examples, make_examples_somatic, labeled_examples_to_vcf,
  convert_to_saved_model, run_oracle_inference, train, fast_pipeline,
  samtools, bcftools

Platform:
  Native container platform is linux/amd64.
  Docker and Podman runs request --platform linux/amd64 from src/main.taf.
  On arm64 hosts this is amd64 emulation, not native arm64 support.
  This app packages the CPU image, not google/deepvariant:1.10.0-gpu.

License:
  TAFFISH app packaging: Apache-2.0.
  Upstream DeepVariant: BSD-3-Clause.
  Bundled third-party components, model files, datasets, and external
  resources keep their own license terms.

Boundaries:
  Inputs must already be aligned, sorted, indexed BAM/CRAM files and an indexed
  compatible reference FASTA. This app does not bundle reference genomes,
  read aligners, quick-start test data, GIAB truth sets, DeepTrio,
  DeepSomatic, or clinical interpretation resources.
  Upstream models are trained on human data and germline diploid calling is the
  core supported use case. Non-human, custom-model, somatic, trio, cohort, and
  GPU workflows need additional design or separate apps.
