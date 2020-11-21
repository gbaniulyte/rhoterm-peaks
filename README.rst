Rho termination peak-caller
===========================

This tool was designed to call peaks at Rho termination regions.
The tool identifies genomic regions where read coverage is significantly
higher in the 'Treated' sample vs 'Untreated'.

Usage
-----
Prerequisite: Python 3.x or higher, SciPy.

Required input is 2 tab-delimited files ('Untreated' and 'Treated') with genome coordinates,
read coverage, strand information and no headers. Strand information can be represented as '+' or '-'
in a separate column, or as a positive/negative read coverage value.

Input parameters must be first entered in the parameters file:
::

	Rho-termination_peak-caller_Input-parameters.json

- **'optional'** parameters can be left blank and will default to current working directory; if no input directory is provided,
  input, parameters and python executable files need to be in the same folder;
- Parameters and paths need to be in double quotation marks;
- **'window'** : Number of positions to include upstream and dowstream of each coordinate when adding read coverage;
- **'single_peak_window'** : Min distance between regions with significant difference in transcriptional readthrough when calling peaks;
- **'p_value_threshhold'** : Max p value from fisher's exact test to use for selecting transcriptional readthrough regions;
- **'ratio_threshhold'** : Min 1/(untreated/treated) ratio direction to use when calling peaks;
  e.g. ratio > 1 means treated sample has a higher read coverage;
- **'genome_size'** : Required to calculate transcriptional readthrough for each position and select the best peak in the region;
- **'coordinate_column'** : Column number in the tab-delimited input file that holds genome coordinate;
- **'read_count_column'** : Column number in the tab-delimited input file that holds read count number for a specific coordinate;
- **'optional_strand_column'** : Can be blank if read count column has negative values to represent minus strand;
- **'optional_input_file_directory'** : Can be left blank if input read count files are in the same directory as the program;
- **'untreated_file_name'** : Full file name with extension; must be tab-delimited;
- **'treated_file_name'** : Full file name with extension; must be tab-delimited;
- **'optional_output_file_directory'** : Can be left blank and will save output files to to current working directory;
- **'optional_out_peak_file_name'** : Full file name with extension; if left blank, output files will have an auto-generated
  file name with parameters; Reports just selected peaks with associated information;
- **'optional_out_sum-score_file_name'** : Summary file; Full file name with extension or if left blank, output files will have
  an auto-generate a file name with parameters; Reports all positions that meet significance/pvalue threshold and were used for
  peak calling and associated information.

Run python executable file.

The program takes ~4h to run with a 4.6 Mbp genome. It can be tested by setting the **'genome_size'** to a lower value (e.g. 100,000)
and using test .gff files that include read coverage up to the genome position 100,000. Test output files are also provided.


Algorithm
---------

1. The program first calculates read coverage upstream and downstream of each genomic position from the 'Untreated' and 'Treated'
samples in a given window size;

2. 'Rho score' (1/((untreated_ds/untreated_us)/(treated_ds/treated_us))) and 'Significance score' (p value from fisher's exact test)
is then calculated using the four read read coverage values; if those scores/positions meet set thresholds, they will be used for peak
calling and saved in a 'Summary' file;

3. 'Rho termination peak' is called as a single position with the highest 'Rho score' in a region. Minimal required distance between peaks
should be set as **'single_peak_window'**.

Output
------

1. Summary file for all positions that met 'Rho score' and 'Significance score' thresholds represents clusters/regions
   of significant Rho termination (increase in read coverage in the 'Treated' file conpared to 'Untreated' sample).
2. Peak file of positions with highest 'Rho score' in a given region.

Both files include

- Coordinate;
- Strand;
- Read coverage in a given window upstream and downstream of the coordinate;
- Rho Readthrough score (R);
- Significance score (p value from fisher's exact test).

Caveats
-------

- When 'Significance score' (p value) is too low to accurately report and is shown as '0.0';
- If one of the regions used in the calculation has no read coverage, it is difficult to calculate 'Rho Readthrough score'
  (1/((untreated_ds/untreated_us)/(treated_ds/treated_us))) due to division by 0 error. In those cases, 'Readthrough score'
  is estimated and if it passes the set threshold, score is reported as 'N/A'.
