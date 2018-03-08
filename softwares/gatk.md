# GATK(Genome Analysis Toolkit)

## 简介

GATK这个工具已经流行了很多年。从我毕业开始（2015年）第一次接触到这个工具，似乎就再也没有离开过这个工具。刚开始只是作为变异检测的软件在使用，也仅仅只会当做变异检测软件使用。不过最近因为工作的需要，从GATK3中找了一些工具，发现原来GATK能完成的功能这么得多，很多之前想要的功能，自己通过一个个脚本去实现，不仅效率低，而且耽误时间。所以决定，依据GATK3.8这个版本（GATK4仍旧在开发中，很多方法变化很大）好好的探究一下他可以完成的功能，实现它在我这“toolkit”的称号名副其实。

## GATK3.8安装

（待补充）

## 方法列表

(待补充)


| 方法名 |描述| 所属类别|
|:------:|:--------:|:------:|
|      |        |      |
|      |        |      |
|      |        |      |

## 测试数据说明

**所有的参数测试均会使用两套数据**

1. 全外bam数据+小区域bed文件（**A**方案）
2. 随机抽取全外bam文件数据+全外bed文件（**B**方案）



## 整体参数

- `-T,--analysis_type`
- `-I,--input_file`
- `-L,--intervals`
- `-R,--reference_sequence`


## annotator

### VariantAnnotator

## bqsr

### AnalyzeCovariates

### BaseRecalibrator

## cancer                          

### AssignSomaticStatus


## contamination

### AnnotatePopulationAFWalker

### ContEst

## coverage

### CallableLoci

### CompareCallableLoci  

### DepthOfCoverage

**个人理解**：
- 这个方法在做芯片评估的时候非常有用，按照意义设计出来的芯片也有可能捕获效果不好，通过这个方法能够很快的得到其中不需要的区域，有问题的区域。以便在下次升级时可以进行优化。

评估序列的覆盖度相关数据，结果可以分成不同样本，不同类别，或者不同文库。可以同时作用多个bam。这项功能是最近刚好用到，而自己写的程序可以统计但是对于内存消耗太大。


除了整体的参数之外，对于`DepthOfCoverage`还有特有的参数

#### `-o,--out`

输出文件前缀，如果不给出则直接输出到标准输出（打印在屏幕上）

```
java -jar GenomeAnalysisTK.jar \
				-T DepthOfCoverage \
				-I $TEST_PATH/test.bam \
				-L $TEST_PATH/test.bed \
				-R $TEST_PATH/hg19.fasta \
				-o $TEST_PATH/DOC_test
```
获得如下7个文件：

![2018-03-08 14_46_55](/softwares/gatk_images/2018-03-08%2014_46_55.png)


**A**方案结果：

DOC_test：第一列为位置，第二列为总深度，第三列为平均深度，第四列则为样本深度（如果输入多个bam，就会出现后续会有后续列）
>Locus	Total_Depth	Average_Depth_sample	Depth_for_1
chr17:48745062	60	60.00	60
chr17:48745063	59	59.00	59
chr17:48745064	62	62.00	62
chr17:48745065	61	61.00	61
chr17:48745066	62	62.00	62
chr17:48745067	64	64.00	64
chr17:48745068	65	65.00	65
chr17:48745069	65	65.00	65
chr17:48745070	65	65.00	65


DOC_test.sample_summary：第一列为样本Id，第二列为总的碱基数目，第三列为target区域平均深度**我们的区域不是13，而是22，但是我们可以发现974和74.32之间的关系却是13，这是因为我输入的bed中有9bp（chr22:100-110），他们在参考基因组上其实都是N，所以默认情况下，这个区域是会被去掉的，后续会有参数能够改变这个默认情况（--includeRefNSites）**，第四列~第六列是分位数，第七列为默认计算超过15X的区域所占百分比
>sample_id	total	mean	granular_third_quartile	granular_median	granular_first_quartile	%_bases_above_15
1	974	74.92	100	66	63	100.0
Total	974	74.92	N/A	N/A	N/A


DOC_test.sample_cumulative_coverage_proportions：覆盖度的累积分布，下列是其中一部分，表示的是深度不小于0X的区域在test_0001这个样本中占100%的区域，`gte`表示`>=`
>	gte_0	gte_1	gte_2	gte_3	gte_4	gte_5	gte_6	gte_7...
>1	1.00	1.00	1.00	1.00	1.00	1.00	1.00	1.00...

DOC_test.sample_cumulative_coverage_counts：结果同上，只是输出不是比例，而是确切位点数，从这也可以看出我们的区域大小是`13`，但其实还有一个区域是因为ref是N所以被去掉了。
>	gte_0	gte_1	gte_2	gte_3	gte_4...
>NSamples_1	13	13	13	13	13...

DOC_test.sample_statistics：上述为深度覆盖的累积分布，这一项结果为每一个深度确切位点数。位点深度为0的位点数为`0`正好是`gte_0-gte_1(13-13)`
>Source_of_reads	from_0_to_1)	from_1_to_2)	from_2_to_3)...
>sample_test_0001	0	0	0...

DOC_test.sample_interval_statistics：**平均深度**超过x的区域数目，因为是超过的统计，所以大于1的肯定也会统计到大于0的里面。而且从这也可以看出我们的bed区域一共有多少区域（`3`）**很奇怪，这个时候他会计算ref为N的那个区域，因为真实情况下我是输入了3个区域，按照位点的时候，他都没有算第三个区域，但是算区域的时候，他会计算第三个区域**
>	depth>=0	depth>=1	depth>=2...
>At_least_1_samples	3	2	2...

DOC_test.sample_interval_summary：每个区域的深度详情，第一列为区域，第二列为总深度，第三列为平均深度，第四列~第九列是一个样本的（总深度、平均深度、1分位、中位数、三分位及默认的超过15X的区域在该区域中所占比例）
>Target	total_coverage	average_coverage	test_0001_total_cvg	test_0001_mean_cvg	test_0001_granular_Q1	test_0001_granular_median	test_0001_granular_Q3	test_0001_%_above_15
chr17:48745062-48745070	563	62.56	563	62.56	62	63	66	100.0
chr19:12874507-12874510	411	102.75	411	102.75	100	102	106	100.0
chr22:101-110	0	NaN	0	NaN	1	1	1	NaN


**B**方案结果：
DOC_test：第一列为位置，第二列为总深度，第三列为平均深度，第四列则为样本深度（如果输入多个bam，就会出现后续会有后续列）
>Locus	Total_Depth	Average_Depth_sample	Depth_for_test_0001
>chr1:17346	0	0.00	0
>chr1:17347	0	0.00	0
>chr1:17348	0	0.00	0
>chr1:17349	0	0.00	0
>chr1:17350	0	0.00	0
>chr1:17351	0	0.00	0
>chr1:17352	0	0.00	0
>chr1:17353	0	0.00	0
>chr1:17354	0	0.00	0

DOC_test.sample_summary：第一列为样本Id，第二列为总的碱基数目，第三列为target区域平均深度（我们的bed区域为46584178，所以我们算一下46584178*0.44差不多就是总数目，但是有出入，后续会说明出入在哪），第四列~第六列是分位数，第七列为默认计算超过15X的区域所占百分比
>sample_id	total	mean	granular_third_quartile	granular_median	granular_first_quartile	%_bases_above_15
test_0001	20440631	0.44	1	1	1	0.0
Total	20440631	0.44	N/A	N/A	N/A

DOC_test.sample_cumulative_coverage_proportions：覆盖度的累积分布，下列是其中一部分，表示的是深度不小于0X的区域在test_0001这个样本中占100%的区域，`gte`表示`>=`
>	gte_0	gte_1	gte_2	gte_3	gte_4	gte_5	gte_6	gte_7...
>test_0001	1.00	0.23	0.15	0.03	0.02	0.01	0.00	0.00...

DOC_test.sample_cumulative_coverage_counts：结果同上，只是输出不是比例，而是确切位点数，从这也可以看出我们的区域大小是`46584178`
>	gte_0	gte_1	gte_2	gte_3	gte_4...
>NSamples_1	46584178	10647046	6806354	1585048...

DOC_test.sample_statistics：上述为深度覆盖的累积分布，这一项结果为每一个深度确切位点数。位点深度为0的位点数为`35937132`正好是`gte_0-gte_1(46584178-10647046)	`
>Source_of_reads	from_0_to_1)	from_1_to_2)	from_2_to_3)...
>sample_test_0001	35937132	3840692	5221306...

DOC_test.sample_interval_statistics：**平均深度**超过x的区域数目，因为是超过的统计，所以大于1的肯定也会统计到大于0的里面。而且从这也可以看出我们的bed区域一共有多少区域（`223775`）
>	depth>=0	depth>=1	depth>=2...
>At_least_1_samples	223775	51442	29119...

DOC_test.sample_interval_summary：每个区域的深度详情，第一列为区域，第二列为总深度，第三列为平均深度，第四列~第九列是一个样本的（总深度、平均深度、1分位、中位数、三分位及默认的超过15X的区域在该区域中所占比例）
>Target	total_coverage	average_coverage	test_0001_total_cvg	test_0001_mean_cvg	test_0001_granular_Q1	test_0001_granular_median	test_0001_granular_Q3	test_0001_%_above_15
chr1:17346-17452	0	0.00	0	0.00	1	1	1	0.0
chr1:30371-30507	0	0.00	0	0.00	1	1	1	0.0
chr1:69056-69689	0	0.00	0	0.00	1	1	1	0.0





#### `-mmq,--minMappingQuality`
#### `--maxMappingQuality`
#### `-mbq,--minBaseQuality`
#### `--maxBaseQuality`
#### `--countType`
#### `-baseCounts,--printBaseCounts`
#### `-omitLocusTable,--omitLocusTable`
#### `-omitIntervals,--omitIntervalStatistics`
#### `-omitBaseOutput,--omitDepthOutputAtEachBase`
#### `-geneList,--calculateCoverageOverGenes`
#### `--outputFormat`
#### `--includeRefNSites`
#### `--printBinEndpointsAndExit`
#### `--start`
#### `--stop`
#### `--nBins`
#### `-omitSampleSummary,--omitPerSampleStats`
#### `-pt,--partitionType`
#### `-dels,--includeDeletions`
#### `--ignoreDeletionSites`
#### `-ct,--summaryCoverageThreshold`



### GCContentByInterval          
## diagnosetargets                 
### DiagnoseTargets                      
## diagnostics                     
### ErrorRatePerCycle            
### FindCoveredIntervals          
### ReadGroupProperties          
### ReadLengthDistribution        
### DiffObjects                  
## examples                        
### GATKPaperGenotyper            
## fasta                           
### FastaAlternateReferenceMaker  
### FastaReferenceMaker           
### FastaStats                   
### VariantFiltration            
## genotyper                       
### UnifiedGenotyper                       
## haplotypecaller                 
### HaplotypeCaller               
### HaplotypeResolver             
## indels                          
### IndelRealigner               
### LeftAlignIndels               
### RealignerTargetCreator        
## m2                              
### MuTect2                        
## missing                         
### QualifyMissingIntervals       
## phasing                         
### PhaseByTransmission           
### ReadBackedPhasing            
## qc                              
### CheckPileup                  
### CountBases                   
### CountLoci                    
### CountMales                   
### CountReadEvents              
### CountReads                    
### CountRODs                    
### CountRODsByRef              
### CountTerminusEvent           
### ErrorThrowing                 
### FlagStat                      
### Pileup                       
### PrintRODs                    
### QCRef                         
### ReadClippingStats            
## readutils                       
### ClipReads                    
### PrintReads                   
### SplitSamFile                 
## rnaseq                          
### ASEReadCounter                
### SplitNCigarReads             
## simulatereads                   
### SimulateReadsForVariants      
## validationsiteselector          
### ValidationSiteSelector       
## varianteval                     
### VariantEval                  
## variantrecalibration            
### ApplyRecalibration           
### VariantRecalibrator           
## variantutils                    
### CalculateGenotypePosteriors   
### CombineGVCFs                  
### CombineVariants              
### GenotypeConcordance           
### GenotypeGVCFs                
### LeftAlignAndTrimVariants      
### RandomlySplitVariants        
### RegenotypeVariants            
### SelectHeaders                
### SelectVariants               
### ValidateVariants             
### VariantsToAllelicPrimitives   
### VariantsToBinaryPed           
### VariantsToTable               
### VariantsToVCF                
