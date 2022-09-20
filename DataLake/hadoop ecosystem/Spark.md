## parqute hive 테이블 데이터 insert
1. Spark 세션 생성
```python
from pyspark.sql import SparkSession

spark = (
SparkSession
.builder
.appName('pos make fact table')
.config('spark.yarn.queue', 'lotte')
.config('hive.metastore.uris', 'thrift://csdlnn01:9083')
.config('spark.driver.maxResultSize','10g')
.config('spark.driver.memory', '10g')
.config('spark.executor.memory', '10g')
.config('spark.executor.memoryOverhead', '4g')
.config('spark.executor.instanced', '20')
.config('spark.executor.cores', '3')
.config('spark.sql.execution.arrow.pyspark.enabled','true')
.enableHiveSupport()
.getOrCreate())
```
3. Spark DataFrame 생성
```python
insert_f_sales = """

SELECT distinct

/*dimension id*/

    fs.date_id

    , fs.product_id

    , fs.store_id

/*집계 결과 항목*/

    , fs.qty as qty /*판매수량*/

    , fs.qty_delta as qty_delta /*판매수량증감*/

    , fs.agg_qty_delta as agg_qty_delta /*누계판매수량증감*/

    , fs.price as price /*판매가*/

    , fs.avg_price as avg_price /*평균단가*/

    , fs.amount as amount /*판매금액*/

    , fs.agg_amount as agg_amount /*누계판매금액*/

    , fs.total_amount as total_amount /*판매총액*/

    , fs.amount_delta as amount_delta /*판매금액증감*/

    , fs.agg_amount_delta as agg_amount_delta /*누계판매금액증감*/

    , fs.market_share as market_share /*m/s*/

    , fs.market_share_delta as market_share_delta /*m/s증감*/

    , fs.agg_market_share  as agg_market_share /* 누계 m/s*/

    , fs.agg_market_share_delta as agg_market_share_delta /*누계m/s증감*/

    , fs.market_size as market_size /*시장규모*/

    , fs.hhi as hhi /*hhi*/

    , fs.deployment_rate as deployment_rate /*도입률*/

    , fs.agg_deployment_rate as agg_deployment_rate /*누적도입률*/

    , fs.turnover_amount as turnover_amount /*회전량*/

    , fs.price_index as price_index /*가격지수*/

    , fs.growth_rate as growth_rate /*성장률*/

    , fs.growth_cnt as growth_cnt /*성장횟수*/

    , fs.decline_cnt as decline_cnt /*하락횟수*/

    , fs.max_growth_rate as max_growth_rate /*최대성장률*/

    , fs.min_growth_rate as min_growth_rate /*최소성장률*/

    , fs.growth_index as growth_index /*성장지수*/

    , fs.cagr as cagr /*CAGR(3Y)*/

    , fs.store_cnt as store_cnt /*점포수*/

    , fs.new_product as new_product /*신제품여부*/

/*rdbms 파티션 값*/

    , fs.pt_yyyy

FROM ods.pos_fact_aggr_aggregated_all_slim fs

"""

inserts = spark.sql(insert_f_sales) # return type : pyspark.sql.dataframe.DataFrame

```
5. Hive에 parquet 테이블 생성
```python
create external table ods.pos_fact_aggr_f_sales_slim

(

    date_id                varchar(32) comment '날짜 surrogate key',

    product_id             varchar(32) comment '상품 surrogate key',

    store_id               varchar(32) comment '상점 surrogate key',

    qty                    bigint comment '판매수량',

    qty_delta              bigint comment '판매수량증감',

    agg_qty_delta          int comment '누계판매수량증감',

    price                  bigint comment '판매가',

    avg_price              float comment '평균단가',

    amount                 bigint comment '판매금액',

    agg_amount             bigint comment '누계판매금액',

    total_amount           bigint comment '판매총액',

    amount_delta           bigint comment '판매금액증감',

    agg_amount_delta       bigint comment '누계판매금액증감',

    market_share           float comment 'm/s',

    market_share_delta     float comment 'm/s증감',

    agg_market_share       float comment '누계m/s',

    agg_market_share_delta float comment '누계m/s증감',

    market_size            bigint comment '시장규모',

    hhi                    float comment 'hhi',

    deployment_rate        float comment '도입률',

    agg_deployment_rate    float comment '누적도입률',

    turnover_amount        float comment '회전량',

    price_index            float comment '가격지수',

    growth_rate            float comment '성장률',

    growth_cnt             float comment '성장횟수',

    decline_cnt            bigint comment '하락횟수',

    max_growth_rate        float comment '최대성장률',

    min_growth_rate        float comment '최소성장률',

    growth_index           float comment '성장지수',

    cagr                   float comment 'CAGR(3Y)',

    store_cnt              bigint comment '점포수',

    new_product            boolean comment '신제품여부',

    pt_yyyy                int comment 'RDB 데이터파티션값'

)

    stored as parquet

    location 'hdfs://csdlnn01:8020/ODS/PROD/POS_fe/prepdata/pos_fact_aggr_f_sales_slim'

    tblproperties ('bucketing_version' = '2', 'external.table.purge'='true')
```
7. Hive parquet 테이블로 insert
```python
def save_table(inserts : pyspark.sql.dataframe.DatFrame, batch_no, target_table, partition_count=10):

    start = time.time()

  

    inserts.repartition(partition_count).write.mode("append").insertInto(target_table, overwrite = False)

  

    # pyspark.sql.DataFrame.repartition -> pyspark.sql.dataframe.DatFrame

    # pyspark.sql.DataFrame.write -> pyspark.sql.DataFrameWriter -> insertInto(tableName[, overwrite]) : Inserts the content of the DataFrame to the specified table (Return None)

  

    end = time.time()

  

    result = f'{batch_no} started at: {datetime.fromtimestamp(start)}, ended at: {datetime.fromtimestamp(end)}, execution took: {end-start}'

  

    print(result)

    return result
```