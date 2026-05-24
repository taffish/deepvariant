# taf-deepvariant

`taf-deepvariant` packages DeepVariant `1.10.0-r1`, the Google DeepVariant
CPU Docker runtime for germline variant calling from aligned BAM/CRAM reads to
VCF or gVCF outputs.

Package identity:

- name: `deepvariant`
- command: `taf-deepvariant`
- kind: `tool`
- TAFFISH version: `1.10.0-r1`
- container image: `ghcr.io/taffish/deepvariant:1.10.0-r1`
- upstream release/tag: `v1.10.0`
- upstream Docker base: `google/deepvariant:1.10.0`
- upstream Docker digest: `sha256:962e5a83b1d76aae6990625d47102785f791603f2138aa1fa9aa4fb6a2eecbe6`
- runtime version string: `DeepVariant version 1.10.0`
- native container platform: `linux/amd64`
- upstream license: BSD-3-Clause

## What Is Included

This app keeps the official upstream DeepVariant CPU Docker runtime intact. The
image includes:

- `run_deepvariant`, the recommended one-command wrapper for the main
  DeepVariant germline calling workflow
- core DeepVariant commands: `make_examples`, `call_variants`, and
  `postprocess_variants`
- report and inspection helpers such as `vcf_stats_report`,
  `show_examples`, and `runtime_by_region_vis`
- advanced or training helpers shipped by the official image, including
  `multisample_make_examples`, `make_examples_somatic`,
  `labeled_examples_to_vcf`, `convert_to_saved_model`,
  `run_oracle_inference`, `train`, and `fast_pipeline`
- model directories bundled by upstream under `/opt/models` for WGS, WES,
  PacBio, ONT R10.4, hybrid PacBio+Illumina, MAS-Seq, and RNA-seq model types
- upstream-bundled `samtools` and `bcftools` 1.15

The app does not bundle reference genomes, read alignments, GIAB truth sets,
PAR BED files, quick-start test data, or user-trained models. Those should be
provided explicitly by the user or by a workflow/data layer.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install deepvariant
```

Install the exact release:

```sh
taf install deepvariant 1.10.0-r1
```

For local testing before this app is published to the public index:

```sh
taf install --from .
```

## Command Mode

`taf-deepvariant --help` prints the TAFFISH app help. `taf-deepvariant -- --help`
passes `--help` to the default upstream `run_deepvariant` command.

Because `command_mode = true`, the first non-option argument is treated as an
in-container executable. For normal DeepVariant runs, either use `--` before
option-leading arguments or name the upstream command explicitly:

```sh
taf-deepvariant -- --model_type=WGS ...
taf-deepvariant run_deepvariant --model_type=WGS ...
taf-deepvariant make_examples --helpfull
taf-deepvariant samtools --version
```

Avoid `taf-deepvariant run ...` unless `run` is really an executable inside the
container. DeepVariant workflow flags belong to `run_deepvariant`.

## Common Workflows

Show upstream version and help:

```sh
taf-deepvariant -- --version
taf-deepvariant -- --helpshort
taf-deepvariant run_deepvariant --helpfull
```

Run a typical WGS call:

```sh
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
```

Print the commands DeepVariant would run without executing them:

```sh
taf-deepvariant -- \
  --model_type=WGS \
  --ref=ref.fa \
  --reads=sample.bam \
  --output_vcf=output.vcf.gz \
  --num_shards=1 \
  --dry_run=true
```

Use another bundled model type:

```sh
taf-deepvariant -- --model_type=WES ...
taf-deepvariant -- --model_type=PACBIO ...
taf-deepvariant -- --model_type=ONT_R104 ...
taf-deepvariant -- --model_type=HYBRID_PACBIO_ILLUMINA ...
taf-deepvariant -- --model_type=MASSEQ ...
taf-deepvariant -- --model_type=RNASEQ ...
```

Call the three core stages manually when you need the full low-level option
surface:

```sh
taf-deepvariant make_examples --helpfull
taf-deepvariant call_variants --helpfull
taf-deepvariant postprocess_variants --helpfull
```

Use upstream-bundled htslib helpers:

```sh
taf-deepvariant samtools faidx ref.fa
taf-deepvariant bcftools --version
```

For independent alignment, sorting, indexing, VCF normalization, or workflow
steps, prefer the dedicated TAFFISH apps for those tools when available. The
bundled `samtools` and `bcftools` are retained because they are part of the
official DeepVariant runtime.

## Inputs And Outputs

The main `run_deepvariant` path expects:

- a reference FASTA and its `.fai` index
- a sorted, indexed BAM or CRAM file aligned to the same reference
- a model type or custom model
- writable output paths for VCF and optionally gVCF

Common outputs include:

- `output.vcf.gz`
- `output.vcf.gz.tbi`
- `output.g.vcf.gz`
- `output.g.vcf.gz.tbi`
- `output.visual_report.html` when `--vcf_stats_report=true`
- intermediate TFRecord files when `--intermediate_results_dir` is set

DeepVariant cannot read BAM data from stdin; input files must exist on disk.
CRAM input follows normal htslib reference behavior and should be paired with a
compatible reference FASTA.

## Platform And Resources

The official upstream Docker tag used here is a single-architecture image:

```text
linux/amd64
```

`src/main.taf` declares `--platform linux/amd64` for Docker and Podman. On
Apple Silicon or other arm64 hosts this means amd64 emulation, not native arm64
support:

```sh
TAFFISH_CONTAINER_BACKEND=docker taf-deepvariant -- --version
TAFFISH_CONTAINER_BACKEND=podman taf-deepvariant -- --version
```

Apptainer behavior depends on the host's ability to run amd64 images.

DeepVariant is a heavy TensorFlow/model runtime. Real analyses usually need
multiple CPU cores, substantial memory, and enough disk space for intermediate
TFRecord data. `--num_shards` controls make_examples parallelism. The upstream
details documentation reports that `call_variants` uses several GB of memory
and that large WGS runs require production-scale resources.

This app packages the CPU image. The official GPU image
`google/deepvariant:1.10.0-gpu`, GPU runtime flags, and experimental GPU
pipeline behavior are not enabled by this app. A future `deepvariant-gpu` app
could package that separately if it becomes useful.

When the CPU image starts, TensorFlow may print informational messages about
missing CUDA drivers, missing TensorRT, or CPU instruction optimization. Those
messages are expected for the CPU runtime and do not mean the version check or
CPU workflow failed.

## Boundaries

This package targets the core DeepVariant CPU germline caller. It does not
package DeepTrio or DeepSomatic as separate runnable apps. DeepSomatic is a
related upstream project in its own repository, and DeepTrio has its own
upstream workflow and model surface.

The upstream README states that DeepVariant supports germline variant calling
in diploid organisms and that the included models are trained on human data.
Non-human organisms, unusual ploidy or copy-number states, custom models,
somatic calling, trio calling, cohort genotyping, benchmarking with `hap.py`,
and external database/reference management need additional workflow design.

Smoke tests validate the runtime version, core help paths, bundled model
directories, upstream-bundled `samtools`/`bcftools`, and a `run_deepvariant`
`--dry_run=true` workflow-construction path. They do not run a full neural
network variant call on real BAM/FASTA data because that would require large
fixtures and production-scale resources.

Each smoke command is self-contained because the public index runs every
`[smoke].test` entry in a fresh temporary container. No smoke entry depends on
files created by an earlier entry.

## License Boundary

The TAFFISH app packaging files are licensed under Apache-2.0. The packaged
upstream DeepVariant software is covered by the BSD-3-Clause license. Bundled
third-party components, TensorFlow/Nucleus dependencies, model files, datasets,
and external resources keep their own license terms.

The upstream project also states that it is not an official Google product and
is not intended for clinical use of any kind.

## Upstream

- Upstream repository: <https://github.com/google/deepvariant>
- Upstream release: <https://github.com/google/deepvariant/releases/tag/v1.10.0>
- Upstream quick start: <https://github.com/google/deepvariant/blob/v1.10.0/docs/deepvariant-quick-start.md>
- Upstream usage guide: <https://github.com/google/deepvariant/blob/v1.10.0/docs/deepvariant-details.md>
- Upstream Docker image: <https://hub.docker.com/r/google/deepvariant>
- Upstream license: BSD-3-Clause

Citation:

- Poplin R, Chang PC, Alexander D, et al. 2018. A universal SNP and small-indel variant caller using deep neural networks. Nature Biotechnology. DOI: `10.1038/nbt.4235`, PMID: `30247488`.
