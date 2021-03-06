# MethGET

A web-based bioinformatics software for correlating DNA methylation and gene expression. The web user-interface can be accessed through  https://paoyang.ipmb.sinica.edu.tw/Software.html .


# Contents
## [System Requirements](#SystemRequirements)

## [MethGET Input](#MethGETInput)

## [Running MethGET](#RunningMethGET)

### [Data Preprocessing](#DataPreprocessing)

### [Single-methylome analyses](#Singlemeth):
  - [Correlation analyses of genome-wide DNA methylation and gene expression](#scatter)
  - [Ordinal association analyses with genes ranked by gene expression level](#rankscatter)
  - [Distribution of DNA methylation by groups of genes with different expression level](#boxplot)
  - [Average methylation level profiles with different expression groups around genes](#metaplot)
 

### [Multiple-methylome analyses](#Multiplemeth):
  - [Gene level association between changes of DNA methylation and changes of gene expression](#deltascatter)
  - [Heatmap representation of DNA methylation and gene expression data together](#heatmap)
 

# <a name="SystemRequirements"></a>System Requirements
* Linux environment

* Python 2.7 
(Type " python -V" to see the installed version. Python2.7 could be downloaded from  http://www.python.org/download/) 
* [ucsc-gtftogenepred](https://bioconda.github.io/recipes/ucsc-gtftogenepred/README.html)

Python modules are needed. To install the packages, use the following commands on linux terminal:
```
	$ pip install numpy
	$ pip install pandas
	$ pip install matplotlib
	$ pip install seaborn
	$ pip install scipy
	$ pip install argparse
	$ pip install sklearn
	$ pip install openpyxl
	$ pip install pyBigWig
```
# <a name="MethGETInput"></a>MethGET Input
Here is the [example data](http://140.109.56.62:8003/Downloadexample) (322MB) to explore the tool’s functions.
### For single methylome
1. DNA Methylation
> CGmap.gz file (need gzip compressed format) is the output of [BS-Seeker2](http://pellegrini.mcdb.ucla.edu/BS_Seeker2/).
>```
>chr1    G       13538   CG      CG      0.6     6       10
>chr1    G       13539   CHG     CC      0.0     0       9
>chr1    G       13541   CHH     CA      0.0     0       9
>chr1    G       13545   CHH     CA      0.0     0       8
>```
>The methylation calling files from other aligners/callers, MethGET provides a python script (methcalls2CGmap.py) to convert them to CGmap.gz, including CX report files generated by Bismark, the methylation calls generated by methratio.py in BSMAP (v2.73), the allc files by methylpy, and the TSV files exported from the methylation calling status with METHimpute. 
>```
>usage: methcalls2CGmap.py [-h] [-n FILENAME]
>                          [-f {bismark,bsmap,methylpy,methimpute}]
>
>optional arguments:
>  -h, --help            show this help message and exit
>
>Input format:
>  -n FILENAME, --filename FILENAME
>                        the file name that the users want to convert to CGMap
>                        format
>  -f {bismark,bsmap,methylpy,methimpute}, --format {bismark,bsmap,methylpy,methimpute}
>                        the type of file to CGmap
>```
>>Example for comverting methylation calls to CGmap.gz:
>>```
>># bismark to CGmap.gz
>>python methcalls2CGmap.py -n CX_report.txt.gz -f bismark
>>```
> 2. Gene Expression File
>> Tab-delimited text ﬁle: contain the gene names and gene expression values.
```
AT1G01310       0.152868
AT1G01320       9.06088
AT1G01340       14.1157
AT1G01350       24.8099
AT1G01355       0.082233
AT1G01360       42.5391
AT1G01370       8.28111
```
> 3. Gene Annotation File
>> gene annotation in GTF file: It may be available from [EnsemblPlant](https://plants.ensembl.org/info/website/ftp/index.html) or [ensemble FTP](https://asia.ensembl.org/info/data/ftp/index.html)

### For multiple methylomes 
>1. Sample Description File (tab-delimited)
>> Sample list
```
trt_1    trt_1_CGmap.gz   trt_1_exp.txt    trt
trt_2    trt_2_CGmap.gz   trt_2_exp.txt    trt
ctrl_1    ctrl_1_CGmap.gz   ctrl_1_exp.txt    ctrl
ctrl_2    ctrl_2_CGmap.gz   ctrl_2_exp.txt    ctrl
```
>2. Gene Annotation File
>> gene annotation in GTF file


# <a name="RunningMethGET"></a>Running MethGET 
## <a name="DataPreprocessing"></a>Data Preprocessing

### preprocess.py
Use the script preprocess.py to preprocess the data for downstream analyses

**Usage:**
```bash
preprocess.py {-s <samplelist> | -n <samplename> -f <cgmap> -e <expressionfile>} -g <gtf> [options]

optional arguments:
  -h, --help            show this help message and exit

For samplelist:
  -s SAMPLELIST, --samplelist SAMPLELIST
                        input sample description file

For individual data:
  -n SAMPLENAME, --samplename SAMPLENAME
                        setting the name of the set of data. This determines
                        the names of output files for downstream analyses
  -f CGMAPFILENAME, --cgmapfilename CGMAPFILENAME
                        input CGmap file
  -e EXPRESSIONFILE, --expressionfile EXPRESSIONFILE
                        input gene expression file

General options:
  -g GTFFILE, --gtffile GTFFILE
                        input gene annotation file
  -c CUTOFF, --cutoff CUTOFF
                        minimum cytosines that are covered by reads
  --TE {True,False}     The input GTF file is TE GTF. The TE methylation each
                        gene will be calculated in GeneMean.txt. The TE
                        analyses can be conducted if the target region is
                        'Gene_Body'
```
**Example for preprocess:**
```
# individual data
python preprocess.py -n demo -f WT.CGmap.gz -e WT_exp.txt -g genes.gtf
# for samplelist
python preprocess.py -s samplelist.txt -g genes.gtf
```
## <a name="Singlemeth"></a>Single-methylome analyses
### <a name="scatter"></a>Correlation analyses of genome-wide DNA methylation and gene expression
### correlation.py
**Usage:**
```
usage: correlation.py [-h] [-n SAMPLENAME] [-p {scatter,kernel}]
                      [-c {CG,CHG,CHH,all}]
                      [-t {Promoter,Gene_Body,Exon,Intron,all}]
                      [-cor {False,pearson,spearman}] [-re0 {True,False}]
                      [-thrs THRESHOLD] [-xlim XLIMIT] [-ylim YLIMIT]
                      [--dotsize DOTSIZE] [--textsize TEXTSIZE]
                      [--ticksize TICKSIZE] [--labelsize LABELSIZE]
                      [--titlesize TITLESIZE]

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  -n SAMPLENAME, --samplename SAMPLENAME
                        the name of the set of data
  -p {scatter,kernel}, --plot {scatter,kernel}
                        create scatterplot or kernel density plot, default is 'scatter'
  -c {CG,CHG,CHH,all}, --context {CG,CHG,CHH,all}
                        choose the context of methylation, default 'all' is to choose them all
  -t {Promoter,Gene_Body,Exon,Intron,all}, --target {Promoter,Gene_Body,Exon,Intron,all}
                        choose the genomic location of methylation, default 'all' is to choose them all

Important general arguments:
  -cor {False,pearson,spearman}, --correlation {False,pearson,spearman}
                        select the type of correlation, default is 'pearson'
  -re0 {True,False}, --skip0 {True,False}
                        Whether genes with 0 expression value would be included. Default 'False' is to include them
  -thrs THRESHOLD, --threshold THRESHOLD
                        Whether skip genes with expression value that is too high, default is to skip genes higher than 2000
                        expression. If want to include them, please set 'None'
  -xlim XLIMIT, --xlimit XLIMIT
                        Nemeric zoom in the gene expression value to clearly understand the distribution
  -ylim YLIMIT, --ylimit YLIMIT
                        Nemeric zoom in the DNA methylation level to clearly understand the distribution

Graphing arguments:
  --dotsize DOTSIZE     dotsize, default is 30
  --textsize TEXTSIZE   textsize, default is 20
  --ticksize TICKSIZE   ticksize, default is 15
  --labelsize LABELSIZE
                        labelsize, default is 20
  --titlesize TITLESIZE
                        titlesize, default is 20

```
**Example for correlation:**
```
# scatter plot
python correlation.py –n demo –p scatter
```

### <a name="rankscatter"></a>Ordinal association analyses with genes ranked by gene expression level
### ordinal.py
**Usage:**
```
usage: ordinal.py [-h] [-n SAMPLENAME] [-c {CG,CHG,CHH,all}]
                  [-t {Gene_Body,Promoter,Exon,Intron,all}]
                  [-re0 {True,False}] [-thrs THRESHOLD] [-shsca {True,False}]
                  [-line {True,False}] [-smoo SMOOTH_N] [-ylim YLIMIT]
                  [--dotsize DOTSIZE] [--textsize TEXTSIZE]
                  [--ticksize TICKSIZE] [--labelsize LABELSIZE]
                  [--titlesize TITLESIZE] [--legendsize LEGENDSIZE]

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  -n SAMPLENAME, --samplename SAMPLENAME
                        the name of the set of data
  -c {CG,CHG,CHH,all}, --context {CG,CHG,CHH,all}
                        choose the context of methylation, default 'all' is to choose them all
  -t {Gene_Body,Promoter,Exon,Intron,all}, --target {Gene_Body,Promoter,Exon,Intron,all}
                        choose the genomic location of methylation, default 'all' is to choose them all

Important general arguments:
  -re0 {True,False}, --skip0 {True,False}
                        whether genes with 0 expression value would be included. Default 'False' is to include them
  -thrs THRESHOLD, --threshold THRESHOLD
                        whether to skip genes with expression value that is too high, default is to skip genes higher than 2000.
                        If want to include them, please set 'None'

chart visulaization arguments:
  -shsca {True,False}, --showscatterplot {True,False}
                        whether to show the scatterplot, default is to show
  -line {True,False}, --smoothline {True,False}
                        whether to show the fitting curves, default is to show
  -smoo SMOOTH_N, --smooth_N SMOOTH_N
                        set the number of ticks to average when drawing the fitting curve, default is 500
  -ylim YLIMIT, --ylimit YLIMIT
                        numeric zoom in the DNA methylation level to clearly understand the distribution

Graphing arguments:
  --dotsize DOTSIZE     dotsize, default is 20
  --textsize TEXTSIZE   textsize, default is 25
  --ticksize TICKSIZE   ticksize, default is 15
  --labelsize LABELSIZE
                        labelsize,, default is 25
  --titlesize TITLESIZE
                        titlesize, default is 25
  --legendsize LEGENDSIZE
                        legendsize, default is 20

```
**Example for ordinal association:**
```
# scatterplot and fitting curves
python ordinal.py -n demo
```

### <a name="boxplot"></a>Distribution of DNA methylation by groups of genes with different expression level
### grouping.py
**Usage:**
```
usage: grouping.py [-h] [-n SAMPLENAME] [-p {boxplot,violinplot}]
                   [-c {CG,CHG,CHH,all}]
                   [-t {Gene_Body,Promoter,Exon,Intron,all}]
                   [-nb NUMBEROFGROUP] [-re0 {True,False}]
                   [-cor {False,pearson,spearman}] [-mean {True,False}]
                   [-sf {True,False}] [-ylim YLIMIT] [--dotsize DOTSIZE]
                   [--textsize TEXTSIZE] [--ticksize TICKSIZE]
                   [--labelsize LABELSIZE] [--titlesize TITLESIZE]
                   [--legendsize LEGENDSIZE]

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  -n SAMPLENAME, --samplename SAMPLENAME
                        the name of the set of data
  -p {boxplot,violinplot}, --plot {boxplot,violinplot}
                        create boxplot or violinplot, default is boxplot
  -c {CG,CHG,CHH,all}, --context {CG,CHG,CHH,all}
                        choose the context of methylation, default 'all' is to choose them all
  -t {Gene_Body,Promoter,Exon,Intron,all}, --target {Gene_Body,Promoter,Exon,Intron,all}
                        choose the genomic location of methylation, default 'all' is to choose them all
  -nb NUMBEROFGROUP, --numberofgroup NUMBEROFGROUP
                        define how many group to seperate gene expression, default is 5

Important general arguments:
  -re0 {True,False}, --skip0 {True,False}
                        whether genes with 0 expression value would be included. Default 'False' is to include them
  -cor {False,pearson,spearman}, --correlation {False,pearson,spearman}
                        select the type of correlation in the table, default is pearson

Chart visulaization arguments:
  -mean {True,False}, --showmeans {True,False}
                        whether to show the position of mean in boxplot or violin plot, default 'True' is to show
  -sf {True,False}, --showfliers {True,False}
                        whether to show outliers in boxplots, default 'True' is to show
  -ylim YLIMIT, --ylimit YLIMIT
                        numeric value. zoom in the DNA methylation level to understand the methylation distribution

Graphing arguments:
  --dotsize DOTSIZE     dotsize, default is 20
  --textsize TEXTSIZE   textsize, default is 20
  --ticksize TICKSIZE   ticksize, default is 15
  --labelsize LABELSIZE
                        labelsize, default is 20
  --titlesize TITLESIZE
                        titlesize, default is 20
  --legendsize LEGENDSIZE
                        legendsize, default is 20

```
**Example for grouping statistics:**
```
# boxplot
python grouping.py -n demo -p boxplot
```

### <a name="metaplot"></a>Average methylation level profiles with different expression groups around genes
### metagene.py
**Usage:**
```
usage: metagene.py [-h] [-n SAMPLENAME] [-p {region,site}] [-ma {True,False}]
                   [-nb NUMBEROFGROUP] [-re0 {True,False}]
                   [-No_bins NUMBEROFBINS] [-psn {TSS,TES}] [-bp BASEPAIR]
                   [-yaxis {auto,set100}] [-xtick XTICKSIZE]
                   [-ytick YTICKSIZE] [-label LABELSIZE] [-title TITLESIZE]
                   [-legend LEGENDSIZE]

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  -n SAMPLENAME, --samplename SAMPLENAME
                        the name of the set of data
  -p {region,site}, --plot {region,site}
                        choose the format of the metaplot, default is region
  -ma {True,False}, --metavaluefile {True,False}
                        put in the metaplot value file if you have created, default is False
  -nb NUMBEROFGROUP, --numberofgroup NUMBEROFGROUP
                        define how many group to seperate gene expression, default is 5
  -re0 {True,False}, --skip0 {True,False}
                        remove genes that their expression value is equal to 0, default is not removing them.

Important arguments:
  -No_bins NUMBEROFBINS, --numberofbins NUMBEROFBINS
                        for 'region' or 'site' plot. 'region' define the total of windows from upstream 
			to downstream, suggest 30. 'site' is the windows of one side, suggest 10.
  -psn {TSS,TES}, --posname {TSS,TES}
                        for 'site' plot. Choose the specific position of the metagene,
			default is TSS (transcription start site)
  -bp BASEPAIR, --basepair BASEPAIR
                        for 'site' plot. The basepairs flanking by the chosen
                        position, default is 2000
  -yaxis {auto,set100}, --yaxissetting {auto,set100}
                        choose if the ticks of yaxis set on 100, default
                        'auto' will automatically adjusted.

Graphing arguments:
  -xtick XTICKSIZE, --xticksize XTICKSIZE
                        the size of the xticks (upstream, gene, downstream),
                        default is 20
  -ytick YTICKSIZE, --yticksize YTICKSIZE
                        the size of the yticks (methylation level), default is 
  -label LABELSIZE, --labelsize LABELSIZE
                        labelsize, default is 20
  -title TITLESIZE, --titlesize TITLESIZE
                        titlesize, default is 20
  -legend LEGENDSIZE, --legendsize LEGENDSIZE
                        legendsize, default is 15

```
**Example for metagene:**
```
# region plot
python metagene.py -n demo -p region -No_bins 30
# site plot
python metagene.py -n demo -p site -No_bins 10 -bp 2000
# if created metagene 'region' value
python metagene.py -n demo -p region -No_bins 30 -ma True
```

## <a name="Multiplemeth"></a>Multiple-methylome analyses
### <a name="deltascatter"></a>Gene level association between changes of DNA methylation and changes of gene expression
### comparison.py
**Usage:**
```
usage: comparison.py [-h] [-s SAMPLELIST] [-c {CG,CHG,CHH}]
                     [-t {Promoter,Gene_Body,Exon,Intron,all}]
                     [-pro PROB_CUTOFF] [-p {scatter,kernel}]
                     [-cor {False,pearson,spearman}]
                     [--shownumber {False,True}] [--cutoff {False,True}]
                     [-mthr METHTHRESHOLD] [-ethr EXPTHRESHOLD]
                     [--methmin METHMIN] [--methmax METHMAX] [--expmin EXPMIN]
                     [--expmax EXPMAX] [--dotsize DOTSIZE]
                     [--textsize TEXTSIZE] [--ticksize TICKSIZE]
                     [--labelsize LABELSIZE] [--titlesize TITLESIZE]
                     [--fontsize FONTSIZE]

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  -s SAMPLELIST, --samplelist SAMPLELIST
                        put in the sample description file
  -c {CG,CHG,CHH}, --context {CG,CHG,CHH}
                        choose the context of methylation, default is CG
  -t {Promoter,Gene_Body,Exon,Intron,all}, --target {Promoter,Gene_Body,Exon,Intron,all}
                        choose the genomic location of methylation, default is
                        Promoter
  -pro PROB_CUTOFF, --prob_cutoff PROB_CUTOFF
                        define the differential genes by their probrability in
                        Gaussian mixture model, default is 0.000001

Important general arguments:
  -p {scatter,kernel}, --plot {scatter,kernel}
                        create scatterplot or kernel density plot, default is
                        scatter
  -cor {False,pearson,spearman}, --correlation {False,pearson,spearman}
                        select the type of correlation, default is pearson
  --shownumber {False,True}
                        whether to show the number of significant genes,
                        default False is not show

Define anomaly genes with methylation changes and gene expression changes:
  --cutoff {False,True}
                        whether to use methylation and gene expression cutoff
                        to define outliers, default is False
  -mthr METHTHRESHOLD, --meththreshold METHTHRESHOLD
                        set cutoff of differential methylated genes. default
                        'auto' uses methylation changes, CG:10, CHG:1, CHH:1
  -ethr EXPTHRESHOLD, --expthreshold EXPTHRESHOLD
                        set cutoff to identify genes that have expression
                        changes, default uses expression changes: log2FC is 1
  --methmin METHMIN     minimum methylation changes for x-axis
  --methmax METHMAX     maximum methylation changes for x-axis
  --expmin EXPMIN       minimum gene expression changes for y-axis
  --expmax EXPMAX       maximum gene expression changes for y-axis

Graphing arguments:
  --dotsize DOTSIZE     dotsize, default is 20
  --textsize TEXTSIZE   textsize, default is 25
  --ticksize TICKSIZE   ticksize, default is 15
  --labelsize LABELSIZE
                        labelsize, default is 25
  --titlesize TITLESIZE
                        titlesize, default is 25
  --fontsize FONTSIZE   fontsize, default is 1.2

```
**Example for comparison:**
```
# show the correlation on scatterplot
python comparison.py -s samplelist.txt -c CG -t Promoter -cor pearson
# show number of differential genes on scatterplot
python comparison.py -s samplelist.txt -c CG -t Promoter -cor False --shownumber True
```

### <a name="heatmap"></a>Heatmap representation of DNA methylation and gene expression data together
### heatmap.py
**Usage:**
```
usage: heatmap.py [-h] [-s SAMPLELIST] [-c {CG,CHG,CHH}]
                  [-t {Promoter,Gene_Body,Exon,Intron}] [-pro PROB_CUTOFF]
                  [-mmax MMAX] [-emax EMAX] [--cutoff {False,True}]
                  [-mthr METHTHRESHOLD] [-ethr EXPTHRESHOLD]
                  [--fontsize FONTSIZE]

optional arguments:
  -h, --help            show this help message and exit

Required arguments:
  -s SAMPLELIST, --samplelist SAMPLELIST
                        put in the sample description file
  -c {CG,CHG,CHH}, --context {CG,CHG,CHH}
                        choose the context of methylation, default is CG
  -t {Promoter,Gene_Body,Exon,Intron}, --target {Promoter,Gene_Body,Exon,Intron}
                        choose the genomic location of methylation, default is
                        Promoter
  -pro PROB_CUTOFF, --prob_cutoff PROB_CUTOFF
                        define the differential genes by their probrability in
                        Gaussian mixture model, default is '0.000001'

Important general arguments:
  -mmax MMAX, --mmax MMAX
                        set the max methylation value for heatmap, default is
                        100
  -emax EMAX, --emax EMAX
                        set the max expression value for heatmap, default is
                        20

Define anomaly genes with methylation changes and gene expression changes:
  --cutoff {False,True}
                        whether to use methylation and gene expression cutoff
                        to define outliers, default is False
  -mthr METHTHRESHOLD, --meththreshold METHTHRESHOLD
                        set cutoff of differential methylated genes. default
                        'auto' uses changes of methylation, CG:10, CHG:1,
                        CHH:1
  -ethr EXPTHRESHOLD, --expthreshold EXPTHRESHOLD
                        set cutoff to identify genes that have expression
                        changes, default uses expression changes: log2FC is 1

Graphing arguments:
  --fontsize FONTSIZE   fontsize, default is 1.2

```
**Example for heatmap:**
```
# heatmap
python heatmap.py -c CG -t Promoter
```




