## basic linux practices  

### introduction
**UNIX** is an operating system which was first developed in the 1960s, and has been under constant development ever since.  
**Linux** is an entire family of open-source Unix operating systems, that are based on the Linux Kernel. This includes all of the most popular Linux based systems like Ubuntu, Fedora, Mint, Debian, and others. More accurately, they’re called distributions or distros.  
The **shell** acts as an interface between the user and the kernel. Most Linux distributions use a graphic user interface (GUI) as their shell, mainly to provide ease of use for their users，like virtual machine with linux system.  But here we highly recommend using a command-line interface (CLI) because it’s more powerful and effective. Tasks that require a multi-step process through GUI can be done in a matter of seconds by typing commands into the CLI.  The CLI can be easily get access to by Terminal in Mac and Putty/Xshell (install required) in Windows.

### practices tutorial
  - [UNIX Tutorial for Beginners](http://www.ee.surrey.ac.uk/Teaching/Unix/)
  
*************************************************************
*Q1: Define relative path and absolute path*  
*Q2：What ERRORs did you meet and how with deal with those errors?*  
*Q3: if you are using remote server, how to transfer files between server and your local PC? (hint: scp for Mac, xftp for Windows)*   
*************************************************************
  
### linux_practice 
(provide hints and answer from the example file only)  

1.Download a gtf file for plactise; or you can use any other gtf file you have.
```
wget http://hgdownload.soe.ucsc.edu/goldenPath/felCat9/bigZips/genes/felCat9.refGene.gtf.gz
gunzip -d felCat9.refGene.gtf.gz
```

2.How many rows in that gtf file?    
(8487 felCat9.refGene.gtf)  
  
3.How to extract the first 10 rows of the file?  
(try "head")  
  
4.Save all "CDS" transcript information to a new file called "CDS.txt".  
(try "grep" and ">" )  
  
5.print out the first column (ChromosomeID) of file CDS.txt.  
(try "cut")

6.search keyword "gene" in gtf file , why the result from "grep -c" and "grep -o \*\*|wc -l" are different?  
($grep -c "gene" felCat9.refGene.gtf, 8487; $grep -o "gene" felCat9.refGene.gtf|wc -l, 16974)  

7.How many kind of transcript(column 3) listed in the gtf file? and what's the number size of each kind of transcript respectively?  
(combine "grep" "sort" and "uniq" together)  

8.use vim editor to add a header at the first line  of gtf file.  
("vim" and "I" to get access to the editing mode."Esc" to the control mode,":wq" to save and quit)  

9.Jump to line 1000 of gtf file in vim editor  
  
10.replace all "refGene" with "CatRef".  
(try "vim" or "sed")

  
    
    
  
## Analysis on a patient cohort.  

### understand the data  (biomed1 server ~/data/fastq/Genie_9.0/)   

************************************************************* 
What information are involved in each column/row in every file?  
How to define "patient" and "sample"?  
How to merge sample and patient information?  
How to search/locate a target SNP?  
What is the difference between germline mutation and somatic mutation?  
*************************************************************

**Documents**  
*introduction:data_guide.pdf;*  
*Sample information: data_clinical_patient.txt; data_clinical_sample.txt;*    [clinical format](https://docs.cbioportal.org/5.1-data-loading/data-loading/file-formats#clinical-data)  
*Mutation information: data_mutations_extended.txt;*  [MAF format](https://docs.cbioportal.org/5.1-data-loading/data-loading/file-formats#mutation-data)   
  
  

**Task 1: complete the following clinical form**  

summary the Primary tumor samples  
|  Cancer Type  | total |  female   | male  |
|  ----  | ----  |  ----  | ----  |
| LUAD  | 7582 | 4682  | 2897 |
| IDC  | 5669 | 5623  | 44 |
| COAD  | 4089 |    |    |
| PRAD  | 2233 |    |    |
| PAAD  | 2105 |    |    |

*************************  
points for discussion：  
1. How to get all answers(from multi-cancer types) in one command line.  
2. why the total number of female and male not equal to the total size? how to deal with those extra samples?    

**challenge 1: Will the elder become riskier in getting specific cancer or getting metastasis?**  

*************************

  
  
  
**Task 2: basic statistical analysis on mutation data**   

- Which gene with the most mutations?  
- How many shared SNP(found in more than 1 sample) among all samples?  （take Tumor_Seq_Allele1 as allele)  

***************************  

**challenge 2: How is the Variant_Tpye distrubution among the top 10 most mutated genes?**  
**********************








### Answer

**Task 1**  

```
##calculate the number of each cancer typ,remember to keep Primary tumor samples only.  (Variant_Tpye)
grep -v "#" data_clinical_sample.txt |grep "Primary tumor" >Primary_tumor_tmp.txt
cut -f 4 Primary_tumor_tmp.txt|sort|uniq -c|sort -rnk1|head  

##merge sample data with patient data  
awk 'NR==FNR{a[$1]=$2}NR>FNR{print $1,$2,$3,$4,a[$1]}'  data_clinical_patient.txt Primary_tumor_tmp.txt > Gender_tmp.txt  
grep "LUAD" Gender_tmp.txt |cut -d " "  -f 5 |sort |uniq -c  

##calculate the gender in one command line
#cat CancernameList_tmp.txt 
#LUAD  
#IDC  
#COAD  
#PRAD  
#PAAD   
#head -1 data_clinical_patient.txt
#Patient Identifier	Sex	Primary Race	Ethnicity Category	Center  
#head -1 Primary_tumor_tmp.txt 
#PATIENT_ID	SAMPLE_ID	AGE_AT_SEQ_REPORT	ONCOTREE_CODE	SAMPLE_TYPE	SEQ_ASSAY_ID	CANCER_TYPE	CANCER_TYPE_DETAILESAMPLE_TYPE_DETAILED 
for i in `cat CancernameList_tmp.txt`;do echo "$i"; awk 'NR==FNR{a[$1]=$2}NR>FNR{print $1,$2,$3,$4,a[$1]}'  data_clinical_patient.txt Primary_tumor_tmp.txt |grep "$i" |cut -d " " -f 5|sort|uniq -c;done  
```  

**Task 2**  

```
##keep "SNP" data only  
grep "SNP" data_mutations_extended.txt >SNP_tmp.txt  

##calculate mutation in each gene. In fact, the gene length should be considered as well （but not in here)  
cut -f 1 SNP_tmp.txt |sort|uniq -c |sort -rnk1|head   

#target SNP by position and Sample information only  
cut -f 5,6,7,12,13,17 SNP_tmp.txt |wc -l  
(727146)  
#exclude the SNP from the same sample
cut -f 5,6,7,12,13,17 SNP_tmp.txt |sort|uniq |wc -l  
(727030)    
#remove sample information, get all shared SNP.


```

**challenge 1**  
```
#in R  
library(ggplot2)  

#merge sample and patient information  
sample <- as.data.frame(read.table("data_clinical_sample.txt",header = TRUE,sep = "\t", dec = ".",stringsAsFactors=FALSE,check.names = FALSE))   
patients <- as.data.frame(read.table("data_clinical_patient.txt",header = TRUE,sep = "\t", dec = ".",stringsAsFactors=FALSE,check.names = FALSE))  
dat<-merge(sample,patients[,c("PATIENT_ID","SEX")],by="PATIENT_ID",all.x=TRUE)  
dim(dat)  

#select target cancer type  
#dat=subset(dat,ONCOTREE_CODE=="LUAD"|ONCOTREE_CODE=="IDC"|ONCOTREE_CODE=="COAD"|ONCOTREE_CODE=="PRAD"|ONCOTREE_CODE=="PAAD")  
dat=subset(dat,ONCOTREE_CODE=="LUAD")  #take LUAD cancer as an example  
dat$AGE_AT_SEQ_REPORT=gsub("<18", "17",dat$AGE_AT_SEQ_REPORT)  
dat$AGE_AT_SEQ_REPORT=gsub(">89", "90",dat$AGE_AT_SEQ_REPORT)  
dat=subset(dat,AGE_AT_SEQ_REPORT!="Unknown")  
dat$AGE_AT_SEQ_REPORT=as.numeric(as.character(dat$AGE_AT_SEQ_REPORT))  
head(dat)  
dim(dat)  

#plot  
table(dat$ONCOTREE_CODE)  
ggplot(data=dat, aes(x=AGE_AT_SEQ_REPORT,fill = ONCOTREE_CODE)) + geom_histogram(bins = 50)  
ggplot(data=dat, aes(x=AGE_AT_SEQ_REPORT,fill = SAMPLE_TYPE)) + geom_histogram(bins = 20)  

```  









