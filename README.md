# QPCR

Two python scripts aim at calculating the **Delta_Ct, Delta_Delta_Ct, Fold Changes, Student's t-test, and P-value** value which
produced by Quantitative real-time polymerase chain reaction (qRT-PCR).

### Why

I have too much qpcr data to process. It costs me a lot of time. I want a simple way to do this.


### Compute QPCR results from QPCR output
Notes:

1. For **ABi 7900** users, data column names must be 'Sample Name','Detector Name','Ct','Ct Mean'. But 'Ct StdEV' is optional.
2. For **ABi ViiA 7 or Q7** users, you can use `qpcrRead.py` to extract data computing results directly.
3. For **Other machines**, you could use `qpcrCalculate.py` to calculate results with your data. the Input file format has to exactly be
the same as **test_interest_data.xls**.

You should specify **internal control** name and **experimental control** name for your data sets.
The following file formats are supported: **xls, xlsx, csv, txt**.

## Dependency

Python2.7 or Python3+

* numpy
* scipy: for t-test
* pandas
* xlrd: excel reader
* xlwt: excel writer
* matplotlib: for plotting (to do)

### Before using this module, see help
```bash
    python qpcrRead.py -h

    python qpcrCalculate.py -h

```

#### Parameters for qpcrCalculate.py

Parameters:
```
    usage: qpcrCalculate.py [-h] -d DATA [-s SHEET] [-i IC] -e EC [-o OUT]
                            [-m {bioRep,techRep,dropOut,stat}] [--header HEAD]
                            [--tail TAIL] [--version]
```

Calculate Delta Ct, DDelta Ct, Fold Changes, P-values for QPCR results.
```
    optional arguments:
      -h, --help            show this help message and exit
      -d DATA, --data DATA  the file(s) you want to analysis. For multi-file
                            input, separate each file by comma.
      -s SHEET, --sheetName SHEET
                            str, int. the sheet name of your excel file you want
                            to analysis.Strings are used for sheet names, Integers
                            are used in zero-indexed sheet positions.
      -i IC, --internalControl IC
                            the internal control gene name of your sample, e.g.
                            GAPDH
      -e EC, --experimentalControl EC
                            the control group name which your want to compare,
                            e.g. hESC
      -o OUT, --outFileNamePrefix OUT
                            the output file name
      -m {bioRep,techRep,dropOut,stat}, --mode {bioRep,techRep,dropOut,stat}
                            calculation mode. Choose from {'bioRep',
                            'techRep','dropOut'.'stat'}.
                            'bioRep': using all data to calculate mean DeltaCT.
                            'techRep': only use first entry of replicates.
                            'dropOut': if sd < 0.5, reject outlier and recalculate mean CT.
                            'stat': statistical testing for each group vs experimental control.
                            Default: 'dropOut'.
      --header HEAD         Row (0-indexed) to use for the column labels of the
                            parsed DataFrame
      --tail TAIL           the tail rows of your excel file you want to skip
                            (0-indexed)
      --version             show program's version number and exit

```
## Usage

### Extract Data from ABi machine output and Calculate Foldchange
```bash
bash qpcr-run.sh -d test/h9_vii7_export.xls -i GAPDH -e H9_NT_LSB_D16 -m dropOut -o test
```

Behind the scenes, extract data first
```bash
    python qpcrRead.py -d test/h9_vii7_export.xls -o test/test_interest_data.xls
```

Then calculate Delta_Ct, Delta_Delta_Ct, Fold_Changes

```bash
    ## from qpcrRead.py output
    python qpcrCalculate.py -d test/test_interest_data.xls \
                            -i GAPDH \
                            -e H9_NT_LSB_D16 \
                            -m dropOut
                            -o test/20150625_NPC_Knockdown
```

### Not ABi machine output data
For other qRT-QPCR output formats, you can reshape your data structure to be the same as **test_interest_data.xls**. Then use `qpcrCalculate.py` directly for your input.

```bash
python qpcrCalculate.py -d test/test_interest_data.xls \
                        -i GAPDH \
                        -e H9_NT_LSB_D16 \
                        -m dropOut
                        -o test/20150625_NPC_Knockdown
```

### Perform Student's t-test from n independent experiments.

#### Step 1: Calculate `Delta Ct`
Input file format:
1. use `qpcrCalulate.py -m {'bioRep','techRep','dropOut'} ` output results as 'stat' input.
2. or your own file contained column: `Delta Ct`. See example file in `test` folder.

#### Step 2: Run statistical testing

**Set `-m stat` explicitly.**

i.e. For 3 independent experiments (biological replicates), try:

* Method 1:  assume all input files contained column `Delta Ct`.  
```bash
python qpcrCalculate.py -d test/input1.xls,test/input2.xls,test/input3.xls \
                        -e H9_NT_LSB_D16 \
                        -m stat \
                        -o test/output
```

* Method 2: single input file contained column `Delta Ct` ( 3 experiments concatenated into one single table).  
then run:
```bash
python qpcrCalculate.py -d test/input_combined.xls \
                        -e H9_NT_LSB_D16 \
                        -m stat \
                        -o test/output
```

**Note:** if you use `-m stat` and **NOT** `Delta Ct` column in your input, the program will try to run `-m bioRep` first, and then do statistical testing.


## TODO List

1. Generate bar or line Plots using Matplotlib automatically
2. Generate Python Plotting Scripts for customized modification
