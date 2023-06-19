

# ATAT-Example-NaFeO_2
## 1 ATAT计算-满钠
### 1.1 结构文件


1. 准备文件：

(1) NaFeO2_O3_unitcell.pwmat

>**NaFeO_2的config结构文件**

(2) config2rndstr.py

>**格式转换**：将config结构文件转换为ATAT计算所用的rndstr格式的结构文件,运行后会输出文件*rndstr.in*

(3) generate_rndstrin.py

>**引入掺杂位点，生成新的结构文件**：可根据用户需求自定义掺杂元素和浓度，输入文件格式为*rndstr_tem.in*，运行后输出掺杂后的结构文件*rndstr.in*
```
# 掺杂位点
element_origin = "Fe"
# 掺杂后的浓度
element2concentration = {"Fe": 0.333333, "Ni": 0.333333, "Mn": 0.333333}
```

2. 操作流程：

(1) 结构文件转换，生成*rndstr.in*文件
```
python config2rndstr.py NaFeO2_O3_unitcell.pwmat
```
(2) 修改*rndstr.in*名称为*rndstr_tem.in*
```
mv rndstr.in rndstr_tem.in
```
(3) 引入掺杂位点，生成新的*rndstr.in*文件
```
python generate_rndstrin.py
```

### 1.2 ATAT计算

1. 准备文件：

(1) rndstr.in
>**ATAT结构文件**:上一步生成的结构文件

(2) sqscell.out
>**扩胞文件**：需要根据掺杂浓度调整

(3) run_sqss.py
>**可控制时间的多进程sqs**:生成500个sqs结构，每次计算运行时间为10s，运行后输出文件夹*Generating*，包含500个结构的sqs结果

```
# 生成的结构数目
num_sqs_folders = 500
# mcsqs 命令的参数，（收敛精度）
tol = 0.01       
# sqs 运行多久后终止
runtime = 10    
```

(4) run_sqss.sh
>**提交服务器脚本**:提交*run_sqss.py*任务

2. 操作流程：

(1) 根据掺杂浓度调整*sqscell.out*文件（这里使用默认值即可）

(2) 提交sqss计算任务，生成包含500个sqs结果的*Generating*文件夹，其中*bestsqs.out*文件为每次计算的最优结构
```
chmod 777 runsqs.sh
sh runsqs.sh
```
### 1.3 静电势计算及排序

1. 准备文件：

(1) Generating

>**包含500次sqs计算结果的文件夹**

(2) rename.sh

>**结构格式转换**：批量转化子文件夹下的ATAT格式结构文件*bestsqs.out*为*POSCAR*格式，以进行静电势计算

(3) get_ewald.python

>**静电势计算**: 计算单个结构的静电势，其中需根据掺杂元素输入元素价态，其输入文件为*POSCAR*
```
# 添加元素价态
data = {"Na":1, "Mn":4,"Fe":3,"Ni":2, "O":-2}
```
(4) ewald_sort.sh

>**静电势批量计算和排序**: 批量计算500个结构的静电势，并进行排序，选出能量最低的50个结构。运行后在每个子文件夹输出ewald能量值文件，并在*Generating*文件下输出静电势排序文件*sort_ewald.txt*

(5) str_sort.sh

>**提取筛选模型的结构数据**：提取筛选的50个结构数据到文件夹*sort_str*中

2. 操作流程

(1) 将脚本*rename.sh*、*ewald_sort.sh*、*str_sort.sh*等复制到*Generating*文件夹中
```
cp rename.sh Generating/
cp ewald_sort.sh Generating/
cp str_sort.sh Generating/ 
```
(2) 批量转换结构格式，运行*rename.sh*，在每个子文件夹下生成*POSCAR*
```
chmod 777 rename.sh
sh rename.sh
```
(3) 在*get_ewald.py*文件中调整元素价态（这里使用默认值即可）

(4) 批量计算静电势，并进行排序，输出最低的50个静电势数据*sort_ewald.txt*，该文件中第一列为静电势数据，第二列为其结构序号
```
chmod 777 ewald_sort.sh
sh ewald_sort.sh
```
(5) 提取筛选的50个结果的结构数据到新文件夹*sort_str*中,子文件夹中包括*ewald*、*POSCAR*、*atom.config*三个文件
```
chmod 777 str_sort.sh
sh str_sort.sh
```
### 1.4 DFT计算

## 2 ATAT计算-空钠

### 静电势计算及排序
1. 准备文件：

(1) POSCAR

>**通过满钠ATAT计算以及DFT计算后得出的最稳定的结构**

(2) NMFNO.py

>**空钠静电势计算以及排序**: 批量生成结构，计算静电势并筛选其中50个能量最低的结构，输出ordering文件夹，包括50个vasp格式的结构文件。注意：其中需输入初始元素价态，以及脱钠后的价态变化

2. 操作流程

(1) 在*NMFNO.py*文件中输入初始元素价态，以及脱钠后的价态变化（这里使用默认值即可）
```
# 添加元素价态
data = {"Na":1, "Ni":2, "Mn":4, "Fe":3, "O":-2}
# 调整脱钠后的价态变化，注意其价态为平均值。
species_map = {"Na1+":{"Na1+":5/9},"Ni2+":{"Ni3.3333+":1.0}}
```
(2) ATAT计算批量生成结构，计算静电势并进行排序，输出文件夹*ordering*，包含能量最低的50个vasp格式的结构文件
```
python NMFNO.py
```



