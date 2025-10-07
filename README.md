# ETS-2025-VSalazar
### obtain the workflow
ml apptainer nextflow
nextflow run epi2me-labs/wf-transcriptomes --help

### download goes to
 ~/.nextflow/assets/epi2me-labs/

### demo dataset
wget https://ont-exd-int-s3-euwst1-epi2me-labs.s3.amazonaws.com/wf-transcriptomes/wf-transcriptomes-demo.tar.gz

### set NXF_SINGULARITY_CACHEDIR to define where to store image
NXF_SINGULARITY_CACHEDIR


###  ~/.nextflow/config:

singularity {
  enabled = true
  autoMounts = true
}

apptainer {
  autoMounts = true
}

process {
  executor = 'slurm' 
  clusterOptions = '--account=def-svassili'
  maxRetries = 1
  errorStrategy = { task.exitStatus in [125,139] ? 'retry' : 'finish' }
  memory = { check_max( 4.GB * task.attempt, 'memory' ) }
  cpu = 1  
  time = '3h' 
}

executor {
  pollInterval = '60 sec'
  submitRateLimit = '60/1min'
  queueSize = 100 
}

profiles {
  fir {
    max_memory='750G'
    max_cpu=192
    max_time='168h'
  }
  nibi {
    max_memory='750G'
    max_cpu=192
    max_time='168h'
  }
}

### run:
nextflow run epi2me-labs/wf-transcriptomes \
    --de_analysis \
    --direct_rna \
    --fastq 'wf-transcriptomes-demo/differential_expression_fastq' \
    --minimap2_index_opts '-k 15' \
    --ref_annotation 'wf-transcriptomes-demo/gencode.v22.annotation.chr20.gtf' \
    --ref_genome 'wf-transcriptomes-demo/hg38_chr20.fa' \
    --sample_sheet 'wf-transcriptomes-demo/sample_sheet.csv' \
    -profile fir,singularity
