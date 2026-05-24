ARG DEEPVARIANT_IMAGE=google/deepvariant:1.10.0@sha256:962e5a83b1d76aae6990625d47102785f791603f2138aa1fa9aa4fb6a2eecbe6
FROM ${DEEPVARIANT_IMAGE}

WORKDIR /root

ENV PATH=/opt/deepvariant/bin:/opt/conda/envs/bio/bin:$PATH
ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=deepvariant

RUN set -eux; \
    run_deepvariant --version 2>&1 | grep -Fx "DeepVariant version 1.10.0" >/dev/null; \
    run_deepvariant --helpshort 2>&1 | grep -F -- "--model_type" >/dev/null; \
    make_examples --helpfull 2>&1 | grep -F -- "--reads" >/dev/null; \
    call_variants --helpfull 2>&1 | grep -F -- "--outfile" >/dev/null; \
    postprocess_variants --helpfull 2>&1 | grep -F -- "--outfile" >/dev/null; \
    test -s /opt/models/wgs/saved_model.pb; \
    test -s /opt/models/wes/saved_model.pb; \
    test -s /opt/models/pacbio/saved_model.pb; \
    test -s /opt/models/ont_r104/saved_model.pb; \
    test -s /opt/models/hybrid_pacbio_illumina/saved_model.pb; \
    test -s /opt/models/masseq/saved_model.pb; \
    test -s /opt/models/rnaseq/saved_model.pb; \
    samtools --version 2>&1 | grep -F "samtools 1.15" >/dev/null; \
    bcftools --version 2>&1 | grep -F "bcftools 1.15" >/dev/null; \
    tmp=/tmp/taf-deepvariant-build-check; \
    rm -rf "$tmp"; \
    mkdir -p "$tmp"; \
    run_deepvariant \
      --model_type=WGS \
      --ref=/tmp/missing-ref.fa \
      --reads=/tmp/missing-reads.bam \
      --output_vcf="$tmp/out.vcf.gz" \
      --output_gvcf="$tmp/out.g.vcf.gz" \
      --intermediate_results_dir="$tmp/intermediate" \
      --num_shards=1 \
      --dry_run=true \
      2>&1 | grep -F "/opt/deepvariant/bin/make_examples" >/dev/null; \
    rm -rf "$tmp"
