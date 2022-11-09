---
title: 电子病历数据处理系统说明文档

summary: 本项目使用Java语言、基于Spring Boot框架实现，负责对EHR数据进行预处理并转换为二进制向量格式，供Tensorflow使用。

authors:
     - Ruoyu Chen
     - Zhen Li
     - admin

date: '2021-05-29T00:00:00Z'

---

## 0. GitHub项目说明

- 本项目[(ehr4hf)](https://github.com/ruoyu-chen/ehr4hf)使用Java语言、基于Spring Boot框架实现，
负责对EHR数据进行预处理并转换为二进制向量格式，供Tensorflow使用。
- 项目[(ehr4hf_tf)](https://github.com/ruoyu-chen/ehr4hf_tf)使用Python语言实现，
基于Tensorflow框架实现深度学习模型的构建、训练和性能评估。  
本文档是对上述两个项目的描述。  

## 1. 关于原始数据的说明

- 本课题的原始数据来自于UPMC的电子病例系统，为数据库导出文件，包括结构化和非结构化数据两大类。  
结构化数据共分为10个表，从数据库导出后为CSV格式。结构化数据的基本信息如下表所示：

|序号| 文件名（R3_138_LU_*_2019_02_08.csv）|   文件大小    | 文件行数 |           说明           |
|----| -----------------------------------| ------------ | ------- |-------------------------|
|1| DIAGNOSES     |      1.7G    | 18891464|约一千八百万条记录。内容为医生给出的诊断，形式为ICD编码|
|2| ENCOUNTERS    |      2.0G    | 20008930|约两千万条记录。内容为患者的就诊记录，包括门诊、电话、住院等等|
|3| LABS          |      9.8G    | 96096430|约九千万条记录。内容为各类生化检查的结果，如GLUCOSE（葡萄糖），POTASSIUM(K)（钾），CREATININE（肌酸酐），CHLORIDE(CL)（氯），SODIUM(NA)（钠）等|
|4| MED_DISPENSES |      1.4G    | 13584529|约一千三百万条记录。内容为患者的用药记录，包含药品的NDC编码|
|5| MED_FILLS     |      2.9G    | 23375187|约两千三百万条记录。内容为患者的取药记录，包含药品的NDC编码|
|6| MED_ORDERS    |      956M    | 4558267 |约四百五十万条记录。内容为医生开出的处方，包含药品的NDC编码|
|7| ORDER_RESULTS |      367M    | 4324460 |约四百三十万条记录。内容为各类检查结果，如 PULMONARY ARTERY SYSTOLIC PRESSURE（肺动脉收缩压）, RIGHT VENTRICLE DILATED（右心室扩张）, RIGHT VENTRICLE FUNCTION（右心室功能）, TRICUSPID REGURGITATION（三尖瓣反流）, MITRAL REGURGITATION（二尖瓣反流）等等|
|8| PATIENT_DEMOGRAPHICS| 3.6M   | 60863   |60863条，每位患者一条记录，内容为患者的人口学数据，包括性别、出生日期、种族等|
|9| PATIENT_VITALS|      198M    | 3970630 |约四百万条，内容为患者的部分生命体征检查结果|
|10| PROBLEM_LIST |      94M     | 1087067 |约一百万条，内容为患者的自述问题，形式为ICD编码|

可以看到，数据的总规模大约为一亿八千万条记录，但其中各类数据分布并不均匀。

## 2. 系统架构及处理流程  
### 2.1 系统架构  
- ehr4hf项目基于Spring Boot框架构建，使用Maven管理项目依赖，重要的依赖包括：  
（1）univocity-parsers, 用于读写CSV文件  
（2）Apache Lucene, 用于检索药品和ICD知识库（由其他工具构建）  
（3）Protobuf Java, 用于将数据导出为tfrecords格式，因为tfrecords格式本质上是一种基于Protobuf的序列化方案  

### 2.2 配置选项
- 项目基于Spring Boot构建，因此所有的配置数据都使用properties配置文件注入，存储于src/main/resources/目录下。
包括：  
（1）combstat.properties，存储组合规则，用于统计数据的分布。  
【注意】：主要用在前期开发和数据分析阶段，目前用处不大。

（2）vitals.properties，存储处理PATIENT_VITALS表时需要的配置信息，包括每个字段的最大值、最小值、平均值和标准差，用于数字归一化。  
【注意】：这些取值范围来自于对PATIENT_VITALS表的统计，暂时不需要修改。但未来更高效、更可靠的方法应该是在运行时计算出这些范围并加以应用

（3）vocab.properties，存储统计受控词表时需要的配置信息。
在本课题中，受控词表指的是：
  - **ICD编码**（来自于 PROBLEM_LIST 和 DIAGNOSES 表的DX_CODE字段，表示疾病编码）、
  - **ATC编码**（由 MED_ORDERS，MED_FILLS，MED_DISPENSES 表的NDC编码转换得到，表示药品分类）、
  - **COMPONENT_NAME**（来自于 LABS 和 ORDER_RESULTS 表，表示患者接受的检查项）。  
  - 提取受控词表是为了将数据向量化。由于本课题的数据规模还是相对比较大的（患者数超过了6万人，数据条数更是超过了一亿条），
很容易理解，只有出现频次超过一定次数的ICD编码、ATC编码和COMPONENT_NAME才是有价值的。
  - 为了提取受控词表，同时控制数据维度，在提取词表的过程中需要对上述表中的特定字段的可能的取值进行统计，
  - 出现频次超过一定阈值、并且按照出现频次排序位于前K个的词汇会被保留下来，为了定制受控词表的提取方法，
  - 设定了如下的配置项：表名（table）、字段名（column）、词汇最少出现频次阈值（minFreq）、保留词汇数量（topK）、
特征类型（featureType，可以是离散型/CATEGORICAL或连续型/CONTINUOUS）以及特征名前缀（prefix，用于避免词汇重名）  
  - 【注意】：这个文件通常不需要修改，最有可能修改的部分是minFreq，目前设置为10000  

（4）csv_files.properties，配置对原始的CSV文件的部分处理逻辑。不考虑NOTES的情况下，目前一共有10类CSV文件，见第一部分的表格。
  - 每个CSV文件中都有日期字段，但名称不同，需要加以解析，从而形成时间序列；
  - 同时并不是所有的字段都需要保留（有部分字段的取值目前全部为空，也有部分字段的取值目前用不到），
  - 为此设计了这个配置文件，用于存储每个CSV文件的相关信息，包括文件名、日期字段的字段名、需要保留的字段列表等。  
  - 【注意】：通常不需要修改这个文件，但如果需要增加对某个CSV文件中某个字段的提取，需要在这里修改配置  

（5）counters.properties，这个配置文件配合cn.edu.bistu.cs.ehr4hf.util.Counters类使用，主要用于数据统计，暂时用处不大。

（6）application.properties，这个配置文件是最重要，也是最常修改的配置文件，包括系统运行所需要的数据文件路径信息，各流水段的配置信息等。

  - 【注意】：最常修改的是三个配置项：  
  - ehr4hf.dirs.raw指定原始数据所在的目录；
  - ehr4hf.dirs.home指定系统的工作目录，很多其他目录都是以这个目录为基准的；
  - ehr4hf.sys.start指定系统启动运行后，从那个流水段开始执行，具体含义见2.2节的数据处理流程。  
  - 此外， ehr4hf.sys.pipeConfigs.<PIPE_NAME>.* 用于配置每个流水段的行为
  - (这里以 <PIPE_NAME>指代流水段的名称，可能的取值见 cn.edu.bistu.cs.ehr4hf.pipelines.PipeType 类)，
  - 其中的ehr4hf.sys.pipeConfigs.<PIPE_NAME>.enabled参数用来开启/禁用每个流水段，
  - 因此部署程序之前需要检查一下 enabled 状态的取值。  

上述配置文件，都已经打包在程序中(ehr4hf.jar文件中)，如果需要覆盖某个或某几个配置文件中的配置信息，
可以将对应的文件拷贝到打包后的程序所在的目录下，按照Spring Boot读取配置文件的约定，
会优先读取这些配置文件，从而覆盖掉打包文件中对应的配置项。  
### 2.3 数据处理流程
从第一节对数据的说明中可以看到，原始数据中所有患者的数据是混杂在一起的，为了开展后续的工作，
首先需要把同一个患者的各类数据从原始数据中提取出来并集中在一起，同时还要对数据格式进行转换，并进行向量化，
然后再针对具体的训练任务提取对应的预测目标标注值。  
为此，本项目的数据处理流程如下图所示：

![图1 数据处理流程图](../imgs/img1.png)

具体来说，目前的数据处理流程被划分为四个流水线段，分别为：  

* "**原始数据预处理**"（RawProcComponent）：负责对原始数据进行过滤，，清除部分垃圾数据，对数据格式进行归一化等。  
* "**患者数据聚集**"（AggregateComponent）：负责将每个患者的数据聚集在一起，写入以患者ID命名的目录中。  
* "**数据向量化及预测目标提取**"（VectorizeComponent）：负责对患者数据进行过滤，根据预先定义的规则，提取、封装特征数据和预测任务目标数据，并将数据向量化。  
* "**tfrecords格式数据导出**"（TFRecConvComponent）：负责对向量化后的特征和预测目标数据进行二进制格式转换与压缩，
变为能够被tensorflow读取的tfrecords二进制格式。  

利用Spring Boot的依赖注入特性，所有的流水线段程序模块，都放在cn.edu.bistu.cs.ehr4hf.pipelines包下，
实现了cn.edu.bistu.cs.ehr4hf.pipelines.PipeComponent接口（包括执行 初始化、业务逻辑和清理 三个功能的函数），
并使用Spring Boot提供的 @Component 标注设置为Spring组件，
从而可以在cn.edu.bistu.cs.ehr4hf.Ehr4hfApplication 主函数中，根据名称来获取对应的实例，从而启动执行
（具体的实现方法，参见Ehr4hfApplication类中的run方法代码）。
各流水线之间相互独立，通过文件来交换数据（即前一个流水段的输出目录为后一个流水段的输入目录），可以独立运行，
可以指定从哪个流水段开始执行（通过application.properties文件中的ehr4hf.sys.start参数设置）。  

## 3. 数据预处理  
本节介绍"原始数据预处理"流水段的处理逻辑，这一流水段是在 cn.edu.bistu.cs.ehr4hf.pipelines.RawProcComponent 类中实现的。  

RawProcComponent类的数据处理流程可以简要地用如下地流程图来表示。

![图2 数据预处理流程图](../imgs/img2.png)

描述如下：  
（1）生成待处理的文件列表。
     如果存在配置项 'ehr4hf.sys.pipeConfigs.RAW_DATA_CONVERSION.extra.files2Proc'，
     则将其对应的值取出并以逗号分割，形成文件列表。
     如果不存在该配置项，则将csv_files.properties中所有的文件作为文件列表。  
（2）遍历文件列表，针对列表中的每个文件，逐个调用 cn.edu.bistu.cs.ehr4hf.parsers.CSVParsers 进行解析。
     CSVParsers类的parseCsv方法用于解析处理某个特定的CSV文件，会调用univocity-parsers解析csv文件并将数据记录封装为Record对象。
     然后再逐个对封装的Record对象进行处理。  
（3）记录级数据处理。  
(3.1) 首先是处理STUDY_ID和日期类型的字段。STUDY_ID 是患者的匿名唯一ID，
     被用于关联同一个患者的所有数据，在所有数据文件的每一行中都存在。
     在每个表中都至少存在一个日期类型的字段，字段名称和数据格式都不一致，为了统一处理，
     一方面通过数据统计，为每个CSV文件挑选了数据100%不为空的一个日期类型的字段，
     通过 csv_files.properties 中对应的配置项传递给程序。
     另一方面，设计了组件 cn.edu.bistu.cs.ehr4hf.util.DateConvert ，
     对每一个日期类型的字段值，在配置文件中提供的多个日期格式（参见application.properties中的 ehr4hf.util.date.format_input ）
     中进行遍历，如果能够匹配某一个格式，则转换为统一的输出格式（参见application.properties中的 ehr4hf.util.date.format_output，目前为yyyyMMdd ）。  
(3.2) 然后是逐个应用记录级过滤器，对数据进行格式转换、清洗、数据正则化等处理。  
这里定义"记录级过滤器"的目的是为了将数据处理逻辑模块化，所有的记录级过滤器都必须实现 cn.edu.bistu.cs.ehr4hf.recFilters.RecordFilter 接口，
并且在 application.properties 中的 'ehr4hf.sys.pipeConfigs.RAW_DATA_CONVERSION.extra.recFilters' 配置项中进行注册，
从而传递给CSVParser对象。在处理数据的过程中，CSVParser类会按照注册的顺序，逐个调用过滤器处理数据。
考虑到过滤器有可能删除记录（比如非法数据）、有可能将1条记录映射为多条记录（ICD或ATC代码转换的情况下），
过滤器的doFilter函数输入参数包含一条记录（Map<String, String> rec），返回参数为记录列表List<Map<String, String>>。  
目前使用到的记录级过滤器包括：  

* **IllegalRecFilter**，非法数据的过滤器，整合了对Encounters/MedOrders等多个数据类型的非法数据过滤功能。具体的过滤规则，见相关类的注释。  

* **PatientVitalRecordFilter**，对PATIENT_VITALS数据进行解析并统一数据格式的过滤器。
 会使用vitals.properties中设定的数据范围对数据进行正则化，正则化方法可以选择 MAX_MIN 或 Z_SCORE，
 目前默认选择的是 MAX_MIN 的正则化方法，正则化范围为 [-1, 1]. 可以通过ehr4hf.util.number.* 参数来定制正则化行为。
 数字类型的正则化逻辑，在 cn.edu.bistu.cs.ehr4hf.util.NumberConvert 类中实现。  

* **CodeConvertRecordFilter**，对ICD/NDC编码进行转换的过滤器类，主要完成下面几个方面的工作：
 1）ICD-9转换为ICD-10，2）NDC/药品名 转换为ATC。  
【**注意**】这个过滤器是本课题中实现难度较大的一个过滤器，主要难点在于ICD/NDC/ATC编码知识库的构建，以及转换算法的设计。
 目前的实现还存在一些问题，有待未来完善。  

* **HFRelatedFilter**，根据"_**2013 ACCF/AHA Guideline for the Management of Heart Failure**_[link](https://www.ahajournals.org/doi/abs/10.1161/CIR.0b013e31829e8776)"，
有很多与HF相关的药物、疾病、化验、影像等，HFRelatedFilter就是加载相关的规则，并应用这些规则来对数据进行过滤和标注的。
此外，除上述与HF相关的药物、疾病、化验、影像外，HF本身也是需要标注的，即哪些病人被诊断具有HF也是需要标注的。
ICD-9体系下的HF编码为: 428.*，如 428.0 Congestive heart failure， 428.1 Left heart failure等。
由于在CodeConvertRecordFilter中已经对ICD-9进行了转码，转换为ICD-10，对HF的识别可以只针对ICD-10下的HF编码：I50.*  
【**注意**】这个过滤器需要一些领域知识的帮助，所有这些领域知识都存储在json文件中，包括HF_Diseases.json，HF_Drugs.json，
HF_Orders.json以及HF_LabTests.json四个文件。这些文件存储在项目根目录下的etc目录下
（由 application.properties 中的 ehr4hf.dirs.etc 配置项决定）。这四个文件会在系统启动时，
被读取并缓存在 cn.edu.bistu.cs.ehr4hf.service.**HFRelatedCodesService** 组件中，供运行时查找。  

* **StatisticsRecordFilter**，目前这个类主要完成三方面的功能：  
 （1）统计每个字段的数据分布。这个功能最简单，直接将每个字段的数据添加到计数器（counters）即可。  
 （2）。统计每类数据的受控词表中词汇出现的频次，并按照出现频次由高到低的顺序，提取出TopK的高频词，作为最终的词表。  
 （3）统计LABS/ORDER_RESULTS元数据。供最终生成测试用例使用。  
【**注意**】在统计 LABS/ORDER_RESULTS 表的元数据时，还存在一些问题。
LABS/ORDER_RESULTS 表的格式类似，都包含 COMPONENT_NAME 字段，存储各类检查项的名称，也就是受控词表中的词汇。
此外，在这两个表中，都包含 ORD_VALUE，ORD_NUM_VALUE，RESULT_FLAG，REFERENCE_LOW，REFERENCE_HIGH，REFERENCE_UNIT 等字段。
从字面意义上来看，REFERENCE_UNIT是数据的单位，REFERENCE_LOW/REFERENCE_HIGH分别为数据的上下限，RESULT_FLAG是数据偏高、偏低、正常等的描述性文字，
ORD_VALUE和ORD_NUM_VALUE分别是非数值类型和数值类型的检查项的实际检查结果。
数据处理的过程中，首先需要根据出现频次对这些检查项进行筛选，这一步比较简单。
困难在于第二步，针对具体的检查项（也就是词汇），需要提取关于其数据类型、合理取值范围的元数据，从而用于后续的数据正则化和向量化操作。
这两个表的数据怀疑有部分是人工录入的，存在数据质量问题，给提取元数据带来了很多困难，目前仍没有得到很好的解决。  


完成数据预处理后，所有患者的同一类数据仍然混杂在一个csv文件中。
"患者数据聚集"（由AggregateComponent类实现）流水段会负责将同一个患者的数据聚集起来存入一个以患者ID命名的目录，
这一过程的业务逻辑比较简单，在本文档中不做详细解释。
     
## 4. 数据向量化、预测目标提取及tfrecords格式导出
本节介绍"数据向量化及预测目标提取"、"tfrecords格式数据导出"流水段的处理逻辑。

### 4.1 数据向量化、预测目标提取  

在 "患者数据聚集" 流水段中，已经把同一个患者的所有数据存储到患者个人的目录下，
因此数据向量化、预测目标提取、tfrecords格式数据导出等任务都是在患者级别对数据进行的处理。
"数据向量化和预测目标提取"由VectorizeComponent类实现，执行流程类似于"数据预处理"，执行流程如下图所示：
![图3 数据向量化和预测目标提取流程图](../imgs/img3.png)

从上图可以看到，VectorizeComponent以患者为单位进行数据处理。  
（1）首先，对患者的人口学数据和Encounters数据进行单独的解析。  
  * **Demographics** 数据来自于PATIENT_DEMOGRAPHICS表，每个患者一条记录，可以计算得到患者的年龄（BIRTH_DATE）、
判断患者是否存活（DEATH_DATE是否为空）、获取患者性别等信息。  

  * **Encounters** 数据来自于ENCOUNTERS表，包括了非常丰富的信息，但目前只提取了 ENC_TYPE 字段的值，
用于判断和提取患者的住院记录。方法是判断 ENC_TYPE 字段的值是否为 'INPATIENT'。
如果是，则认定为是住院，并提取START_DATE，END_DATE字段的值作为住院的起止时间。  

（2）接下来，将患者的其他类型的数据从文件中读出，并封装为 cn.edu.bistu.cs.ehr4hf.model.Patient 类型的对象。  
（3）最后，应用患者级过滤器对患者数据进行分析。
与记录级过滤器类似，患者级过滤器需要实现cn.edu.bistu.cs.ehr4hf.patFilters.PatientFilter接口，标注为Spring Boot的Component，
并在application.properties文件的ehr4hf.sys.pipeConfigs.DATA_VECTORIZATION.extra.patientFilters配置项中注册。

目前用到的患者级过滤器包括：
* **InPatientFilter**，患者层面上的住院统计过滤器，目前实现的功能还比较简单，只是合并了相邻的或者有时间重叠的住院记录，没有尝试识别转院。  
【**TODO**】这里还应该对住院记录做更深入的分析和处理。  
  
* **TargetExtractPatientFilter**，用于抽取预测目标的过滤器，目前抽取的预测目标包括:  
1）RE_ADMISSION, 30天内重入院，以30天内重新住院为正例，超过阈值（由ehr4hf.targets.reAdminThresh设置，默认180天）重入院/或未再入院为反例。  
2）INPATIENT_DEATH, 住院期间死亡，检查患者的死亡日期是否在某一次住院的时间范围内。  
3）HF_STAGE, 患者的Heart Failure分级，目前这个数据量很小，基本不可用。  
4）LENGTH_OF_STAY, 住院时长。  
5）EJECTION_FRACTION, 患者的射血分数值。  
在 TargetExtractPatientFilter 中，为每一个预测目标的提取，设计了一个函数来实现业务逻辑，
因此如果要添加新的预测目标，可以增加新的函数，同时修改 TargetExtractPatientFilter 类的 shouldDiscard 方法添加对新函数的调用就可以。

* **TrainingCaseExtractFilter**，提取训练样本并写入文件的过滤器，样本写入文件的功能逻辑由Vectorizer模块实现。  
与 TargetExtractPatientFilter 类似，这个过滤器也是为每一个预测目标单独设置了处理函数，因为在输出训练样本时，
是将同一类预测目标的训练样本放在一起的，而一个患者可能有多个提取得到的训练目标标注值
（比如同时具有30天内重入院，射血分数，住院时长等预测目标的标注值）。   
同时，在输出训练样本到文件的时候，由于存在不同的输出方案，
使用了 cn.edu.bistu.cs.ehr4hf.vectorizers.Vectorizer 接口对输出方案进行抽象，
目前只提供了一种实现：cn.edu.bistu.cs.ehr4hf.vectorizers.SimplePatVectorizer。
当前的输出方案可以用下图来加以简单解释：  

![图4 训练样本的输出方案图解](../imgs/img4.png)  

具体来说，目前使用的 SimplePatVectorizer，会逐个遍历患者数据中的预测目标，
为每一个预测目标（包含了预测目标的类型枚举值、预测目标发生的日期、预测目标的标注值）调用对应的输出函数进行输出。
具体到每一个预测目标上，SimplePatVectorizer采用的方案是从预测目标发生的日期开始，回溯给定的天数（默认值是45天，
由参数 ehr4hf.sys.pipeConfigs.DATA_VECTORIZATION.extra.max_seq 决定）。
然后将这个时间段内的所有时间点数据对象（PatientTPoint类，封装某个患者某一天内的所有数据），
分别转换为一个向量，向量维度由之前提取的受控词表的大小决定。
之后将所有的向量按照时间先后排列为时间序列并输出到一个文本文件（CSV格式，文件名中包含样本ID）。
同时，所有的标注结果（每一个 <日期, 患者ID, 样本ID, 标注值> 四元组为一行）被写入一个 .labels文件，
所有的元数据 (样本个数，所有特征的名称列表，时间序列长度等)被写入一个meta文件。  

【**注意**】这里采用的向量化输出方案是比较简单的，将患者在同一天内的数据压缩为一个向量，
在这个向量中，为受控词表的每一个词预留了维度，所以数据较为稀疏。
未来可以考虑选择一种概念向量化方案，将受控词表中的所有词映射到同一个向量空间中，
然后将同一天内的数据转换为多个较短的向量，从而保留更多的语义。  

### 4.2 tfrecords格式文件导出  
在上一小节中介绍的向量化输出方案，输出结果是文本格式的CSV文件，并且是每个样本一个文件，因此在磁盘上生成了大量的小文件，占用了很多磁盘空间，
同时在后续的tensorflow训练过程中效率也很低。为了优化IO效率，增加了 TFRecConvComponent 将训练样本转存为gzip压缩的tfrecords文件格式。 

TFRecord文件格式的官方文档详见[链接](https://www.tensorflow.org/tutorials/load_data/tfrecord). 
本质上来说，TFRecord格式是一个定制的[Protocol Buffers](https://developers.google.com/protocol-buffers/)序列化格式. 
TFRecord使用的protobuf序列化格式包括两个：
[feature](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/example/feature.proto)
和
[example](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/example/example.proto).
具体的格式可以用下图来解释：

![图5 TFRecords格式图解](../imgs/img5.png)  

即，TFRecords文件的二进制数据是由Example序列组成的，而Example是由Feature列表组成的，Feature则是由键值对组成的，
TFRecords支持bytes，float和int64列表在内的特征值。  
TFRecConvComponent 在实现以tfrecords格式导出数据的时候，借助了protobuf工具编译的Java类库
（包含在cn.edu.bistu.cs.ehr4hf.tensorflow包中）。
训练数据被按照上图所示的格式封装，并与NOTES的向量化数据合并在一起，
按照gzip格式压缩后以 10000 条记录为单位，输出到文件中。

## 5. 深度学习模型训练及评估  
本节介绍与[(ehr4hf_tf)](https://github.com/ruoyu-chen/ehr4hf_tf)项目相关的内容。  

目前，已经完成的深度学习模型是针对 30天内重入院 任务进行预测的。
训练数据中包括 "45天的时序数据"、"Discharge Note的向量化表示"、"预测目标标注值"三部分数据。
其中，45天的时序数据是从结构化EHR数据经过前几节介绍的数据处理过程转换得到的。
Notes的向量化目前有两种方案: TF-IDF提取主题词并经PCA降维到指定维度以及LDA主题模型提取指定个数的主题，两种方案的向量维度均为100。  

在上述数据的基础上，为了完成 30天重入院 的预测任务，设计了如下图所示的基线模型：  

![图6 基线神经网络模型架构图解](../imgs/img6.png)  

基线模型在不同的超参数组合下的性能表现：

| 序号 | Embedding层方案 | RNN层方案 | 输入时序数据维度 | 输入时间序列长度 | 性能表现 |
| --- | --------------- | -------- | -------------- | ------------- |---------|
|  0  | No Embedding    | LSTM     |   5529         | 45 | TBD       |
|  1  | Dense * 1       | LSTM     |   5529         | 14 | TBD       |
|  2  | Dense * 3       | LSTM     |   5529         | 14 | TBD       |
|  3  | Dense * 1       | LSTM     |   3294         | 14 | TBD       |
|  4  | Dense * 3       | LSTM     |   3294         | 14 | TBD       |
|  5  | Auto-Encoder    | LSTM     |   3294         | 14 | TBD       |
|  6  | Stacked Auto-Encoder| LSTM     |   3294         | 14 | TBD       |
|  7  | Denoising Auto-Encoder| LSTM     |   3294         | 14 | TBD       |
|  8  | Variational Auto-Encoder| LSTM     |   3294         | 14 | TBD       |
|  9  | * | Attention     |   3294         | 14 | TBD       |


## 6. 待完成的任务清单

（1）调整特征生成方式，完善LABS/ORDER_RESULTS中非数值类特征的生成方式

（2）尝试使用Attention模型对时序数据进行处理

（3）尝试更多的Notes向量化方案，比如ClinicalBert。尝试基于Notes的时序数据

（4）尝试使用Keras Tuner/Manifold等神经架构搜索框架来自动化超参数搜索。

（5）应对存在类别不均衡的训练数据的影响，对不同的类别设定不同的权重

（6）将 EF回归 作为训练目标，预训练神经网络模型，并用于其他任务的迁移学习


