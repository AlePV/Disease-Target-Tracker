# Disease-Target-Tracker
# A tool to find disease-related targets based on protein-protein interaction networks

<p align="center">
    <img src="./Media/Disease_Target_Tracker_WF.png?raw=true" width="1000">
</p>

This Knime workflow uses multiple databases to search for disease related proteins by experimental reports or manual annotations, then proteins are classified by the development phase of their related drugs and assigned a score as follows:

- T1: Score 1.0, the target has approved compounds or phase 4 of development for indicated disease.
- T2: Score 0.7, the target has compounds under clinical trials or phases 1 to 3 of development for indicated disease.
- T3: Score 0.4, the target has compounds under preclinical investigations or phase 0 of development for indicated disease.
- T4: Score 0.1, the target has interactions with T1, T2 or T3 targets related to the disease. T4 targets also have an aditional conectivty score associated to how strong is their connection to the disease related targets.  
 


**Note:** Some targets could have multiple target classifications at the same time.

The workflow provides as results two protein-protein networks and lists of targets with their classifications and scores for the indicated disease.  

## Requirements ##
**This workflow has been programed to be used on <ins>Linux OS</ins>, no other OS where tested**
- MySQL database service. [MySQL website and manual](https://dev.mysql.com/doc/refman/8.0/en/installing.html).
- Last ChEMBL MySQL database. [ChEBML databases](https://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBLdb/releases/chembl_31/).
- "Target to disease mapping with ICD identifiers" and "Drug to disease mapping with ICD identifiers" files from TTD. [TTD full data downloads](http://db.idrblab.net/ttd/full-data-download). 
- Associated targets for a disease of choice on TSV format from Open Targets Platform. [Open Target Platform website](https://platform.opentargets.org/).
- Knime Analytics platform version 4.6.3 or higher. [Download Knime](https://www.knime.com/knime-analytics-platform).
- Our *in-house* Knime workflow [Disease Target Tracker](./Disease_Target_Tracker.knwf).


**To follow these instructions you must have already installed [Knime](https://www.knime.com/), [MySQL](https://dev.mysql.com/doc/refman/8.0/en/installing.html) and configured the lastest [ChEBML database](https://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBLdb/latest/) on your machine.**

## Setting ChEMBL database with MySQL on Linux ##

Quick step by step MySQL installation. Form more information visit [MySQL website](https://dev.mysql.com/doc/refman/8.0/en/installing.html).

### MySQL installation: ### 
```
sudo apt update
sudo apt install mysql-server
```
### MySQL root password.  ###
log into MySQL database as sudo:
```
sudo mysql
```
set a password for root user:
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
### Download ChEMBL database and load data to MySQL ###
**NOTE:** This workflow was design to work with the 31 release of the mysql, later releases may no be compatible due changes on the tables structure of the lastest releases of the ChEMBL database.

Download the 31 release of  ChEMBL database of mysql (chembl_31_mysql.tar.gz). 
<p align="center">
<img src="./Media/ChEMBLdb_download.png?raw=true" width="500">
</p>

Extract files:
```
tar -xvf chembl_31_mysql.tar.gz
```
Log into MySQL and enter yout password:
```
mysql -u root -p
```
Create an empty database on MySQL:
```
mysql> create database chembl_31 DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

```
Logout of MySQL and run the following command to load data to the database (**this could take several hours**):
```
mysql -u root -p chembl_31 < chembl_31_mysql.dmp
```

## 1. Connect ChEMBL local MySQL database with the workflow ##

First download and import our workflow [Disease Target Tracker](./Disease_Target_Tracker.knwf) to Knime software. Then configure **MySQL Connector** node by right clicking at the node and click configure option. Complete the fields with your Hostname, Database name, username and Password based on your personal MySQL information.

<p align="center">
<img src="./Media/MySQL_Connector.png?raw=true" width="500">
</p>

## 2. Select input files from Terapeutic Targets Database (TTD) ##
Download 2021 release of "Target to disease mapping with ICD identifiers" and "Drug to disease mapping with ICD identifiers", the actual files are called ["1_P1-01-TTD_target_download.txt"](./Inputs/1_P1-01-TTD_target_download.txt) and ["2_Drugs_info-TTD.txt"](./Inputs/2_Drugs_info-TTD.txt) respectively on our Inputs folder.

**NOTE:** The 2024 releases have a different format that is not compatible with the current version of the workflow, nevertheless, the last version files can be Downloaded from TTD "Target to disease mapping with ICD identifiers" and "Drug to disease mapping with ICD identifiers" files from [TTD download section](http://db.idrblab.net/ttd/full-data-download).

<p align="center">
<img src="./Media/TTD_website.png?raw=true" width="500">
</p>

Configure the "Therapeutic Target Database" node by browsing the files. Taget to disease file first and Drugs to disease as second file.

<p align="center">
<img src="./Media/Therapeutic_Target_Database.png?raw=true" width="500">
</p>

## 3. Select input files from Human Atlas ##

On [Human Atlas](https://www.proteinatlas.org/search/protein_class%3ADisease+related+genes) download the "The disease related genes" file by clicking on download "TSV" using "Gene synonym" and "Uniprot accession" columns as filters. Or download our april 2024 query from this repository Intputs folder [3_protein_class_Disease](./Inputs/3_protein_class_Disease.tsv).

<p align="center">
<img src="./Media/HumanAtlas_Disease-related-genes.png?raw=true" width="500">
</p>

Configure the "Therapeutic Target Database" node by browsing the files. Taget file first and Drugs file second.

<p align="center">
<img src="./Media/HumanAtlas_node.png?raw=true" width="500">
</p>

## 4. Select input files from Open Targets Platform ##

On Open [Targets Platform](https://platform.opentargets.org/) search for any Disease and download the Associated Targets file on TSV format. Or download our file where we used Alzheimer's disease as query: [MONDO_0004975-associated-diseases.tsv](https://github.com/AlePV/Disease-Target-Tracker/blob/main/Inputs/4_MONDO_0004975-associated-diseases.tsv).

<p align="center">
<img src="./Media/Open_Targets_Platform_website.png?raw=true" width="500">
</p>

Configure "Open Targets Platform" node by browsing amd selecting the file.

<p align="center">
<img src="./Media/Open_Targets_Platform.png?raw=true" width="500">
</p>

## 5. Disease selection ##

Frist execute "Disease list" node to read all available diseases on ChEMBL database.

<p align="center">
<img src="./Media/Disease_list_node.png?raw=true" width="100">
</p>

Then configure and select one disease from the list on "Disease selector" node. If no list is displayed on "Disease selector" configuration reset and execute "Disease list" again, and try to configure "Disease selector" again.

<p align="center">
<img src="./Media/Disease_selector.png?raw=true" width="500">
</p>

## 6. Choose a folder to write the results and execute the workflow ##

Configure "Folder to write results" node by browsing to a folder to write result files. Make sure to select a folder and not a file.

<p align="center">
<img src="./Media/Results_folder.png?raw=true" width="500">
</p>

Finaly execute the rest of the workflow by clicking on "Execute all executable nodes" buttom.

## 7. Results ##

**NOTE:** The following tables are sample of the result files, not the complete results.

### [1_PPI_network_Alzheimer_no-opentarget-filter.csv](./Outputs/1_PPI-network_Alzheimer_Disease_no-opentarget-filter.csv) ###
 Has protein-protein interactions, as Prot_A and Prot_B columns with the protein genes, uniprot ID for each protein, interaction score and Disease. If the target is a protein complex, the gene name is replaced by ChEMBL ID of the complex, and the uniprot ID will be a list of uniprot IDs.

| Prot_A | Prot_B | score | Disease             |
|--------|--------|-------|---------------------|
| GNB3   | HTR7   | 0.932 | Alzheimer's Disease |
| GNB3   | GNAS   | 0.982 | Alzheimer's Disease |
| GNB3   | GNG3   | 0.998 | Alzheimer's Disease |
| GNB3   | GNG5   | 0.998 | Alzheimer's Disease |
| GNB3   | GNG11  | 0.998 | Alzheimer's Disease |
| GNB3   | GNG12  | 0.998 | Alzheimer's Disease |
| GNB3   | GNG10  | 0.998 | Alzheimer's Disease |
| GNB3   | GNGT1  | 0.999 | Alzheimer's Disease |
| GNB3   | GNG13  | 0.999 | Alzheimer's Disease |
|...     |...     |...    |...                  |


### [2_PPI_network_Alzheimer_opentarget-filter.csv](./Outputs/2_PPI-network_Alzheimer_Disease_opentarget-filter.csv) ###
Same as the previous file, but including only targets found on Open Targets Platform.

### [3_Targets_score_Alzheimer_no-opentarget-filter.xlsx](./Outputs/3_Targets-score_Alzheimer_Disease_no-opentarget-filter.xlsx) ###
List of single proteins related to the disease, sorted by target score.

| Target_name                                    | Complex_participants | Node_id  | Uniprot_id             | Target_type     | Target_group   | Source_db           | Target_group_score | Target_group_score_normalized | Conectivity_Score |
|------------------------------------------------|----------------------|----------|------------------------|-----------------|----------------|---------------------|--------------------|-------------------------------|-------------------|
| glutamate nmda receptor; grin1/grin2a          | GRIN2D,GRIN1         | CPX-289  | O15399, Q05586         | PROTEIN COMPLEX | T1, T2, T3, T4 | ChEMBL, STRING, TTD | 16.1               | 1                             |                   |
| ubiquitin carboxyl-terminal hydrolase 1        |                      | USP1     | O94782                 | SINGLE PROTEIN  | T1, T2, T3     | ChEMBL              | 16                 | 0.994375                      |                   |
| butyrylcholinesterase                          |                      | BCHE     | P06276                 | SINGLE PROTEIN  | T1, T2         | ChEMBL, TTD         | 15                 | 0.938125                      |                   |
| mothers against decapentaplegic homolog 2      | SMAD4,SMAD2,SMAD3    | CPX-1    | Q13485, Q15796, P84022 | PROTEIN COMPLEX | T2, T3, T4     | ChEMBL, STRING      | 6.1                | 0.4375                        |                   |
| bile salt export pump                          |                      | ABCB11   | O95342                 | SINGLE PROTEIN  | T2, T3         | ChEMBL              | 6                  | 0.431875                      |                   |
| glutamate receptor 2 …                         | GRIA1,GRIA2          | CPX-8767 | P42261, P42262         | PROTEIN COMPLEX | T3, T4         | STRING, TTD         | 1.1                | 0.15625                       |                   |
| gamma-amino-n-butyrate transaminase            |                      | ABAT     | P80404                 | SINGLE PROTEIN  | T3             | ChEMBL              | 1                  | 0.150625                      |                   |
| alpha-2-macroglobulin-like protein 1 …         |                      | A2ML1    | A8K2U0                 | SINGLE PROTEIN  | T4             | STRING              | 0.1                | 0.1                           | 0.494391081453543 |
| sodium channel protein type 10 subunit alpha … | SCN1B,SCN4B,SCN10A   | CPX-8682 | Q07699, Q8IWT1, Q9Y5Y9 | PROTEIN COMPLEX | T4             | STRING              | 0.1                | 0.1                           | 0.91594389683585  |
|...|...|...|...|...|...|...|...|...|...|

### [4_Targets_score_Alzheimer_opentarget-filter.xlsx](./Outputs/4_Targets-score_Alzheimer_Disease_opentarget-filter.xlsx) ###
Same as the previous file, but including only targets found on Open Targets Platform.

### 8. Network visualization ###
The network can be visualized using Cytoscape and the attributes can be added by loading [3_Targets_score_Alzheimer_no-opentarget-filter](./Outputs/3_Targets-score_Alzheimer_Disease_no-opentarget-filter.xlsx) file to the network nodes.

<p align="center">
<img src="./Media/1_PPI-network_Alzheimer_Disease_no-opentarget-filter.png?raw=true" width="800">
</p>
