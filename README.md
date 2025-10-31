# Lab 06 - Snakemake

Snakemake is similar in concept to Nextflow but is more file centric.
You specify a series of files to be created, and by examining the
inputs and outputs of rules, snakemake will construct a DAG and perform
the required tasks.

Similar to nextflow, snakemake can create and utilize specific software
environments per rule (process) and also can make use of a profile to set
various run options.

```bash
snakemake -s Snakefile --workflow-profile
```

## Create the snakemake environment

Create a minimal conda environment with snakemake installed from the YML file
provided in envs/

## Navigate to the single/ directory

Look at the contents of the Snakefile. It is very similar to the example we 
talked about in lecture. 

### single/ Tasks

1. With your conda environment you just created active, run the following command:

```bash
snakemake -s Snakefile
```

2. Observe what gets created and where - pay attention to the locations of files as
snakemake directly checks if the file exists once the rule has executed. 

## Navigate to the multi/ directory

Look at the files in the samples/ directory. What portions of their name are common?
Which portion is unique to each sample?

Recall from lecture that Snakemake can allow for parallelization and generalization by 
using pattern matching as implemented by  **wildcards**. These wildcards will be automatically
determined by the appropriate regex pattern.

In this example, we can use the entire string before ".bam" as the wildcard.

```
input:
    bam = 'samples/{sample}.bam'

output:
    sorted_bam = 'samples/{sample}.sorted.bam'
```

When Snakemake observes this pattern, it will look in the location specified for
files that are named with any characters (.+) ending in ".bam". For this example,
it will look in the samples/ directory and determine that "sampleA" and "sampleB"
are valid wildcards for this pipeline. Anywhere {sample} is used throughout the
snakemake pipeline will be replaced at runtime with both values the 'sampleA' and
'sampleB'

### multi/ Tasks

With this in mind, please do the following:

1. Specify the following two outputs in a python list in the rule all input - Snakemake
assumes the working directory is where the Snakefile is located.
- results/sampleA.sorted.bam.bai
- results/sampleB.sorted.bam.bai

2. Fill in the rules for samtools_sort and samtools_index
- The input for samtools_sort are our starting files
- The input for samtools_index should be the output of samtools_sort - remember these should be the actual file
- The output of samtools sort should be redirected to a new file using '>'
- Samtools index will create a file with the exact same name as the input with an added ".bai" extension
    i.e. sampleA.sorted.bam.bai

3. When you've confirmed the above works, run your pipeline using:

```bash
snakemake -s Snakefile --workflow-profile profile
```

Remember that expand() is a snakemake function that creates a cross product
of the requested variables:

```bash
expand('{letters}_{numbers}.txt', letters=['A', 'B'], numbers=['1', '2', '3']
```
Keep in mind that {} (curly braces) within an expand statement **do not** refer
to wildcards: the values substituted in are provided by the following variable
in the statement. The above function will create a python **list** that looks 
like below:

```bash
['A_1.txt', 'A_2.txt', 'A_3.txt', 'B_1.txt', 'B_2.txt', 'B_3.txt']
```

This will help us programmatically construct many filenames without having to 
type them all out manually. 

4. Replace your list in the rule all input with an expand() statement.

5. Delete the outputs in results/

6. Re-run your snakemake command

## Navigate to the intermediate/ directory

So far you have seen expand used in the rule all, but you can also use expand()
in the input of other rules. By doing so, you will **ensure** that rule won't
run until all of the files in the list exist. Just as making the output of one
process an input to the next in nextflow creates a dependency, this also ensures
in snakemake that the next rule won't execute until the dependency has finished.


### intermediate/ Tasks

Generate a snakemake pipeline that will run FASTQC on both samples, and then
run MultiQC as a final step.

1. Think carefully about the target file we desire from this pipeline and specify
it as your rule all input

2. Fill in the module for FASTQC
- Specify `-o results/` in your command

3. Fill in the module for MULTIQC
- Use an expand() statement in the input to capture the created files from
the previous step
- You will need to provide multiqc the directory to search
- You will need to specify `-o results/` in your command

4. When you are confident it works, you may use the following command:

```bash
snakemake -s Snakefile --workflow-profile profile
```

## Navigate to the advanced/ directory

If you finish with everything, try to construct a Snakemake pipeline that will
run FASTQC on all of the reads in the samples/ directory. The pipeline should
also run Trimmomatic correctly for *paired end* reads. After both of these
processes have finished, have your pipeline run multiqc.

Snakemake has a similar concept to a stub-run called a dryrun. If you use the
following command:

```bash
snakemake -s Snakefile --workflow-profile profile --dryrun -p
```

Snakemake will not actually run the rules but will show you the plan of the jobs
it would execute.

Hints:
- Several rule all inputs will work, but you can make this pipeline work with only a single
file in your rule all input.
- You will need to specify the output directory for fastqc (-o results/)
- Look closely at the naming patterns and consider using more than a single wildcard
- Trimmomatic requires an adapter fasta, which I've provided for you in the refs/ directory
- Use the default settings for trimmomatic paired end reads
- You will need to use the 2> to redirect the stderr from trimmomatic to a log file for multiqc