# <a name="data-lakes"></a>Data Lake

Data Lake 是一种存储库，可以按数据本来的原始格式存储大量的数据。 Data Lake 存储经过优化，数据规模可以达到数 TB 甚至数 PB。 这些数据通常来自多个异类源，可以是结构化的、半结构化的或非结构化的。 Data Lake 的理念是指以原始的非转换状态存储一切内容。 此方法不同于传统的[数据仓库](../relational-data/data-warehousing.md)，后者在引入数据时对数据进行转换和处理。

Data Lake 的优点：

- 绝不会丢弃数据，因为数据是以原始格式存储的。 这在无法事先知道可以从数据中获得何种见解的大数据 环境中特别有用。
- 用户可以浏览数据并创建自己的查询。
- 可能比传统的 ETL 工具更快。
- 比数据仓库更灵活，因为它可以存储非结构化和半结构化数据。 

完整的 Data Lake 解决方案由存储和处理两部分组成。 Data Lake 存储的设计用途包括：容错、确保无限可伸缩性，以及在引入不同形状和大小的数据时实现高吞吐量。 Data Lake 处理涉及生成一个或多个用于实现此类目的的处理引擎，可以对存储在 Data Lake 中的数据进行大规模的处理。

## <a name="when-to-use-a-data-lake"></a>何时使用 Data Lake

Data Lake 的典型用途包括[数据浏览](./interactive-data-exploration.md)、数据分析和机器学习。 

Data Lake 也可充当数据仓库的数据源。 使用此方法时，原始数据先引入到 Data Lake 中，然后转换为结构化的可查询格式。 此转换通常使用 [ELT](../relational-data/etl.md#extract-load-and-transform-elt)（提取-加载-转换）管道，就地对数据进行引入和转换操作。 已经是关系数据的源数据可以通过 ETL 过程直接进入数据仓库，跳过 Data Lake。

Data Lake 存储通常用于事件流式处理或 IoT 方案，因为此类存储可以保存大量的关系数据和非关系数据，不需进行转换，也不需架构定义。 此类存储可以在低延迟的状况下处理大量的小型写入数据，并已针对大规模的吞吐量进行了优化。

## <a name="challenges"></a>挑战

- 缺少架构或描述性元数据可能使得数据难以使用或查询。
- 整个数据缺少语义一致性可能导致数据难以分析，除非用户精于数据分析。
- 可能难以保证进入 Data Lake 的数据的质量。 
- 如果没有正确的监管，可能造成访问控制和隐私方面的问题。 什么信息会进入 Data Lake？谁可以访问该数据？该数据的用途是什么？
- 若要集成已经是关系数据的数据，Data Lake 可能不是最佳方式。
- Data Lake 本身不提供跨组织的集成视图或整体视图。 
- Data Lake 可能会成为那些从来没有进行过实际分析或见解挖掘的数据的转储之地。

## <a name="relevant-azure-services"></a>相关的 Azure 服务

- [Data Lake Store](/azure/data-lake-store/) 是一种兼容 Hadoop 的超大规模存储库。
- [Data Lake Analytics](/azure/data-lake-analytics/) 是一项按需分析作业服务，用于简化大数据分析。

