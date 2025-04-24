# 📊 Great Expectations: เงื่อนไขตรวจสอบข้อมูล (Expectations)

**Great Expectations (GX)** เป็นเครื่องมือ Open-Source สำหรับตรวจสอบคุณภาพข้อมูล เงื่อนไข (Expectations) ช่วยกำหนดกฎให้ข้อมูลในตาราง (เช่น `accounts`, `customer`) ตรงตามที่ต้องการ ด้านล่างนี้คือ **30 เงื่อนไข** ที่ใช้บ่อยใน GX เวอร์ชัน 0.18.22 จัดเป็น JSON format พร้อมคำอธิบายสั้นๆ
- **ตัวอย่าง 30 เงื่อนไข** ใน GX เวอร์ชัน 0.18.22 (JSON format พร้อมคำอธิบาย)
- **การตั้งค่า Data Sources** สำหรับ **PostgreSQL**, **MongoDB**, **Vertica**, **Kafka**, **Iceberg**, **Dremio**, และ **CSV**

---

## 🚀 วิธีใช้งาน

1. **ตั้งค่า Datasource**\
   แก้ไข `great_expectations.yml` เพื่อเชื่อมต่อกับ PostgreSQL (หรือ MongoDB ถ้าใช้)

   ```yaml
   datasource:
     my_postgres:
       class_name: Datasource
       execution_engine:
         class_name: SqlAlchemyExecutionEngine
         connection_string: postgresql://user:pass@localhost:5432/db
   ```

2. **เพิ่ม JSON เงื่อนไข**\
   คัดลอก JSON ด้านล่างไปใส่ใน `great_expectations/expectations/my_suite.json`

3. **สร้าง Checkpoint**\
   สร้าง `my_checkpoint.yml` เพื่อรันเงื่อนไขทั้งหมดและส่งผลไป DataHub

   ```yaml
   name: my_checkpoint
   validations:
     - expectation_suite_name: my_suite
   action_list:
     - name: store_validation_result
       action:
         class_name: StoreValidationResultAction
     - name: update_data_docs
       action:
         class_name: UpdateDataDocsAction
     - name: send_to_datahub
       action:
         class_name: DataHubValidationAction
         module_name: acryl_datahub_gx_plugin.action
         server_url: http://localhost:8080
   ```

4. **รัน Checkpoint**\
   ใช้ CLI เพื่อตรวจสอบข้อมูล

   ```bash
   great_expectations checkpoint run my_checkpoint
   ```

5. **ดูผลใน Data Docs**\
   เปิดหน้า HTML เพื่อเช็คว่าข้อมูลผ่านเงื่อนไขหรือไม่

   ```bash
   great_expectations docs build
   ```

**หมายเหตุ**: ถ้าใช้ DataHub ตรวจสอบว่า `acryl-datahub-gx-plugin==0.3.0` ติดตั้งแล้ว (จากปัญหา error ที่คุณเจอ)

---
## 🛠️ **การตั้งค่า Data Sources**

ตารางด้านล่างสรุป **Data Sources** ที่รองรับ พร้อมรายละเอียด:

| **Data Source** | **ประเภท** | **ตัวอย่างข้อมูล** | **Execution Engine** |
| --- | --- | --- | --- |
| PostgreSQL | Relational DB | `accounts`, `customer` | SqlAlchemyExecutionEngine |
| MongoDB | NoSQL DB | `transactions`, `logs` | PandasExecutionEngine |
| Vertica | Data Warehouse | `accounts_warehouse` | SqlAlchemyExecutionEngine |
| Kafka | Streaming | `account_events` | SparkDFExecutionEngine |
| Iceberg | Data Lake | `accounts_iceberg` | SparkDFExecutionEngine |
| Dremio | Data Lake | `accounts_lake` | SqlAlchemyExecutionEngine |
| CSV | File-based | `accounts_*.csv` | PandasExecutionEngine |

### 1. **PostgreSQL** 🗄️

เชื่อมต่อกับฐานข้อมูล **PostgreSQL** สำหรับตาราง เช่น `accounts`, `customer`

```yaml
my_postgres_datasource:
  class_name: Datasource
  execution_engine:
    class_name: SqlAlchemyExecutionEngine
    credentials:
      drivername: postgresql+psycopg2
      host: localhost
      port: "{}"
      username: {}
      password: {}
      database: {}
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetSqlDataConnector
      assets:
        {}:
          table_name: {}
          schema_name: {}
```

### 2. **MongoDB** 🍃

เชื่อมต่อกับ **MongoDB** สำหรับ collections เช่น `transactions`, `logs`

```yaml
my_mongodb_datasource:
  class_name: Datasource
  execution_engine:
    class_name: PandasExecutionEngine
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetMongoDataConnector
      mongodb_connection_string: mongodb://mongo_user:mongo_pass@eden206.kube.baac.or.th:27017
      database: {}
      assets:
        {}:
          collection_name: {}
        
```

### 3. **Vertica** 📈

เชื่อมต่อกับ **Vertica Data Warehouse** สำหรับตาราง เช่น `accounts_warehouse`

```yaml
my_vertica_datasource:
  class_name: Datasource
  execution_engine:
    class_name: SqlAlchemyExecutionEngine
    credentials:
      drivername: vertica+vertica_python
      host: {}
      port: "{}"
      username: {vertica_user}
      password: {vertica_pass}
      database: {}
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetSqlDataConnector
      assets:
        {}:
          table_name: {}
          schema_name: {}
```

### 4. **Kafka** 📡

เชื่อมต่อกับ **Kafka topics** เช่น `account_events`, `transaction_events`

```yaml
my_kafka_datasource:
  class_name: Datasource
  execution_engine:
    class_name: SparkDFExecutionEngine
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetKafkaDataConnector
      bootstrap_servers: http://localhost:8080
      topics:
        account_events: #ตัวอย่าง
          schema: '{"account_id": "string", "event_type": "string", "timestamp": "string"}'
      
```

### 5. **Iceberg** ❄️

เชื่อมต่อกับตาราง **Iceberg** ใน S3 เช่น `accounts_iceberg`

```yaml
my_iceberg_datasource:
  class_name: Datasource
  execution_engine:
    class_name: SparkDFExecutionEngine
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetFilesystemDataConnector
      base_directory: s3a://banking-bucket/iceberg/
      assets:
        accounts_iceberg:
          table_name: {}
          catalog: {}
          namespace: {}
          provider: aws
        transactions_iceberg: #ตัวอย่าง
          table_name: transactions_iceberg
          catalog: banking_catalog
```

### 6. **Dremio** 🌊

เชื่อมต่อกับ **Dremio Data Lake** สำหรับตาราง เช่น `accounts_lake`

```yaml
my_dremio_datasource:
  class_name: Datasource
  execution_engine:
    class_name: SqlAlchemyExecutionEngine
    credentials:
      drivername: arrow-flight
      host: {}
      port: "32010"
      username: dremio_user
      password: dremio_pass
      database: Banking_Lake
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetSqlDataConnector
      assets:
        accounts_lake: #ตัวอย่าง
          table_name: accounts_lake
          schema_name: lake
```

### 7. **CSV** 📄

อ่านไฟล์ **CSV** จาก directory เช่น `accounts_*.csv`

```yaml
my_csv_datasource:
  class_name: Datasource
  execution_engine:
    class_name: PandasExecutionEngine
  data_connectors:
    default_configured_data_connector_name:
      class_name: ConfiguredAssetFilesystemDataConnector
      base_directory: /data/csv_files/
      assets:
        accounts_csv:
          pattern: accounts_.*\.csv
        customer_csv:
          pattern: customer_.*\.csv
```

### **การตั้งค่า Checkpoint** 🔄

กำหนด **checkpoint** เพื่อตรวจสอบข้อมูลจากทุก data source

```yaml
checkpoints:
  my_checkpoint:
    config_version: 1.0
    class_name: Checkpoint
    validations:
      - batch_request:
          datasource_name: my_postgres_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: accounts
        expectation_suite_name: my_suite
      - batch_request:
          datasource_name: my_postgres_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: customer
        expectation_suite_name: customer_Accounts
      - batch_request:
          datasource_name: my_mongodb_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: transactions
        expectation_suite_name: transactions_suite
      - batch_request:
          datasource_name: my_vertica_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: accounts_warehouse
        expectation_suite_name: warehouse_suite
      - batch_request:
          datasource_name: my_kafka_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: account_events
        expectation_suite_name: kafka_suite
      - batch_request:
          datasource_name: my_iceberg_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: accounts_iceberg
        expectation_suite_name: iceberg_suite
      - batch_request:
          datasource_name: my_dremio_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: accounts_lake
        expectation_suite_name: dremio_suite
      - batch_request:
          datasource_name: my_csv_datasource
          data_connector_name: default_configured_data_connector_name
          data_asset_name: accounts_csv
        expectation_suite_name: csv_suite
    action_list:
      - name: store_validation_result
        action:
          class_name: StoreValidationResultAction
      - name: send_to_datahub
        action:
          module_name: datahub_gx_plugin.action
          class_name: DataHubValidationAction
          server_url: http://localhost:8080
```

---

## 📋 **รายการเงื่อนไข (Expectations)**

### 1. **ความสมบูรณ์ (Completeness)** ✅

ตรวจสอบว่าข้อมูลครบถ้วน ไม่ขาดหาย

```json
[
  {
    "expectation_type": "expect_column_values_to_not_be_null",
    "kwargs": { "column": "account_id" },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_not_be_null",
    "kwargs": { "column": "customer_id" },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_to_exist",
    "kwargs": { "column": "balance" },
    "meta": {}
  },
  {
    "expectation_type": "expect_table_row_count_to_be_between",
    "kwargs": { "min_value": 100, "max_value": 10000 },
    "meta": {}
  }
]
```

- `expect_column_values_to_not_be_null` **(account_id)**: รหัสบัญชีต้องไม่เป็น null\
  *ใช้เมื่อ*: ต้องการรหัสบัญชีครบทุกแถว
- `expect_column_values_to_not_be_null` **(customer_id)**: รหัสลูกค้าต้องไม่เป็น null\
  *ใช้เมื่อ*: รหัสลูกค้าต้องมีข้อมูล
- `expect_column_to_exist`: ต้องมีคอลัมน์ `balance`\
  *ใช้เมื่อ*: ต้องการคอลัมน์ที่จำเป็น
- `expect_table_row_count_to_be_between`: ตารางต้องมี 100-10,000 แถว\
  *ใช้เมื่อ*: ควบคุมปริมาณข้อมูล

---

### 2. **ความไม่ซ้ำ (Uniqueness)** 🔑

ตรวจสอบว่าข้อมูลไม่ซ้ำกัน

```json
[
  {
    "expectation_type": "expect_column_values_to_be_unique",
    "kwargs": { "column": "account_id" },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_unique_value_count_to_be_between",
    "kwargs": { "column": "customer_id", "min_value": 50, "max_value": 5000 },
    "meta": {}
  },
  {
    "expectation_type": "expect_multicolumn_values_to_be_unique",
    "kwargs": { "column_list": ["account_id", "customer_id"] },
    "meta": {}
  }
]
```

- `expect_column_values_to_be_unique`: รหัสบัญชีต้องไม่ซ้ำ\
  *ใช้เมื่อ*: รหัสบัญชีเป็นคีย์หลัก
- `expect_column_unique_value_count_to_be_between`: รหัสลูกค้ามี 50-5,000 ค่าไม่ซ้ำ\
  *ใช้เมื่อ*: ตรวจจำนวนลูกค้าที่ไม่ซ้ำ
- `expect_multicolumn_values_to_be_unique`: คู่ `account_id` และ `customer_id` ไม่ซ้ำ\
  *ใช้เมื่อ*: ตรวจสอบคีย์ผสม

---

### 3. **ช่วงของค่า (Value Ranges)** 📏

ควบคุมให้ค่าอยู่ในช่วงที่กำหนด

```json
[
  {
    "expectation_type": "expect_column_values_to_be_between",
    "kwargs": { "column": "balance", "min_value": 0, "max_value": 100000 },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_mean_to_be_between",
    "kwargs": { "column": "balance", "min_value": 1000, "max_value": 50000 },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_be_greater_than",
    "kwargs": { "column": "minimum_balance", "value": 0 },
    "meta": {}
  }
]
```

- `expect_column_values_to_be_between`: ยอดเงินอยู่ระหว่าง 0-100,000\
  *ใช้เมื่อ*: ควบคุมยอดเงิน
- `expect_column_mean_to_be_between`: ค่าเฉลี่ยยอดเงินอยู่ระหว่าง 1,000-50,000\
  *ใช้เมื่อ*: ตรวจแนวโน้มยอดเงิน
- `expect_column_values_to_be_greater_than`: ยอดขั้นต่ำต้องมากกว่า 0\
  *ใช้เมื่อ*: ยอดขั้นต่ำห้ามติดลบ

---

### 4. **รูปแบบข้อมูล (Format)** 📅

ตรวจสอบรูปแบบข้อมูล เช่น วันที่, รหัส

```json
[
  {
    "expectation_type": "expect_column_values_to_match_regex",
    "kwargs": { "column": "customer_id", "regex": "^\\d{8}$" },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_match_strftime_format",
    "kwargs": { "column": "created_date", "strftime_format": "%Y-%m-%d" },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_match_like_pattern",
    "kwargs": { "column": "account_type", "like_pattern": "%ing" },
    "meta": {}
  }
]
```

- `expect_column_values_to_match_regex`: รหัสลูกค้าเป็นตัวเลข 8 หลัก\
  *ใช้เมื่อ*: ควบคุมรูปแบบรหัสลูกค้า
- `expect_column_values_to_match_strftime_format`: วันที่เป็น "YYYY-MM-DD"\
  *ใช้เมื่อ*: วันที่ต้องถูกต้อง
- `expect_column_values_to_match_like_pattern`: ประเภทบัญชีลงท้ายด้วย "ing"\
  *ใช้เมื่อ*: ตรวจรูปแบบข้อความ

---

### 5. **ประเภทข้อมูล (Data Type)** 🗳️

ตรวจสอบประเภทข้อมูล

```json
[
  {
    "expectation_type": "expect_column_values_to_be_of_type",
    "kwargs": { "column": "balance", "type_": "float" },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_be_in_type_list",
    "kwargs": { "column": "account_type", "type_list": ["str", "varchar"] },
    "meta": {}
  }
]
```

- `expect_column_values_to_be_of_type`: ยอดเงินเป็น float\
  *ใช้เมื่อ*: ต้องการยอดเงินเป็นทศนิยม
- `expect_column_values_to_be_in_type_list`: ประเภทบัญชีเป็น string หรือ varchar\
  *ใช้เมื่อ*: ควบคุมประเภทข้อมูล

---

### 6. **ความสัมพันธ์ระหว่างคอลัมน์ (Cross-Column)** 🔗

ตรวจสอบตรรกะระหว่างคอลัมน์

```json
[
  {
    "expectation_type": "expect_column_pair_values_a_to_be_greater_than_b",
    "kwargs": { "column_A": "balance", "column_B": "minimum_balance" },
    "meta": {}
  },
  {
    "expectation_type": "expect_select_column_values_to_be_unique_within_record",
    "kwargs": { "column_list": ["account_id", "customer_id"] },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_pair_values_to_be_equal",
    "kwargs": { "column_A": "account_id", "column_B": "linked_account_id" },
    "meta": {}
  }
]
```

- `expect_column_pair_values_a_to_be_greater_than_b`: ยอดเงินมากกว่ายอดขั้นต่ำ\
  *ใช้เมื่อ*: ตรวจตรรกะยอดเงิน
- `expect_select_column_values_to_be_unique_within_record`: รหัสบัญชีและรหัสลูกค้าในแถวเดียวไม่ซ้ำ\
  *ใช้เมื่อ*: ป้องกันข้อมูลซ้ำในแถว
- `expect_column_pair_values_to_be_equal`: รหัสบัญชีเท่ากับรหัสบัญชีที่เชื่อมโยง\
  *ใช้เมื่อ*: ตรวจความสัมพันธ์รหัส

---

### 7. **โครงสร้างตาราง (Schema)** 🗂️

ตรวจสอบโครงสร้างตาราง

```json
[
  {
    "expectation_type": "expect_table_columns_to_match_ordered_list",
    "kwargs": { "column_list": ["account_id", "customer_id", "account_type", "balance", "created_date"] },
    "meta": {}
  },
  {
    "expectation_type": "expect_table_column_count_to_equal",
    "kwargs": { "value": 5 },
    "meta": {}
  },
  {
    "expectation_type": "expect_table_column_count_to_be_between",
    "kwargs": { "min_value": 4, "max_value": 6 },
    "meta": {}
  }
]
```

- `expect_table_columns_to_match_ordered_list`: ตารางมีคอลัมน์ตามลำดับ\
  *ใช้เมื่อ*: ควบคุมโครงสร้างตาราง
- `expect_table_column_count_to_equal`: ตารางมี 5 คอลัมน์\
  *ใช้เมื่อ*: ตรวจจำนวนคอลัมน์
- `expect_table_column_count_to_be_between`: ตารางมี 4-6 คอลัมน์\
  *ใช้เมื่อ*: อนุญาตให้คอลัมน์ผันแปร

---

### 8. **การกระจายของข้อมูล (Distribution)** 📈

วิเคราะห์การกระจายข้อมูล

```json
[
  {
    "expectation_type": "expect_column_distinct_values_to_be_in_set",
    "kwargs": { "column": "account_type", "value_set": ["saving", "current", "fixed"] },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_median_to_be_between",
    "kwargs": { "column": "balance", "min_value": 500, "max_value": 20000 },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_kl_divergence_to_be_less_than",
    "kwargs": {
      "column": "balance",
      "partition_object": { "bins": [0, 1000, 5000, 10000, 100000], "weights": [0.2, 0.3, 0.3, 0.2] },
      "threshold": 0.1
    },
    "meta": {}
  }
]
```

- `expect_column_distinct_values_to_be_in_set`: ประเภทบัญชีเป็น "saving", "current", "fixed"\
  *ใช้เมื่อ*: ควบคุมประเภทบัญชี
- `expect_column_median_to_be_between`: มัธยฐานยอดเงินอยู่ระหว่าง 500-20,000\
  *ใช้เมื่อ*: ตรวจการกระจายยอดเงิน
- `expect_column_kl_divergence_to_be_less_than`: การกระจายยอดเงินใกล้เคียงกับที่กำหนด\
  *ใช้เมื่อ*: วิเคราะห์ความคลาดเคลื่อน

---

### 9. **ความยาวและขนาด (Length and Size)** 📏

ตรวจสอบความยาวหรือขนาดข้อมูล

```json
[
  {
    "expectation_type": "expect_column_value_lengths_to_be_between",
    "kwargs": { "column": "customer_id", "min_value": 8, "max_value": 8 },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_be_null",
    "kwargs": { "column": "closed_date", "mostly": 0.9 },
    "meta": {}
  }
]
```

- `expect_column_value_lengths_to_be_between`: รหัสลูกค้ามีความยาว 8 อักขระ\
  *ใช้เมื่อ*: ควบคุมความยาวรหัส
- `expect_column_values_to_be_null`: วันที่ปิดบัญชีเป็น null อย่างน้อย 90%\
  *ใช้เมื่อ*: วันที่ปิดบัญชีส่วนใหญ่ควรว่าง

---

### 10. **การเปรียบเทียบชุดข้อมูล (Set Comparison)** 🗳️

เปรียบเทียบข้อมูลกับชุดที่กำหนด

```json
[
  {
    "expectation_type": "expect_column_values_to_be_in_set",
    "kwargs": { "column": "account_type", "value_set": ["saving", "current"] },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_distinct_values_to_contain_set",
    "kwargs": { "column": "account_type", "value_set": ["saving", "current"] },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_distinct_values_to_equal_set",
    "kwargs": { "column": "account_type", "value_set": ["saving", "current", "fixed"] },
    "meta": {}
  },
  {
    "expectation_type": "expect_column_values_to_not_be_in_set",
    "kwargs": { "column": "account_status", "value_set": ["closed", "suspended"] },
    "meta": {}
  }
]
```

- `expect_column_values_to_be_in_set`: ประเภทบัญชีเป็น "saving" หรือ "current"\
  *ใช้เมื่อ*: จำกัดประเภทบัญชี
- `expect_column_distinct_values_to_contain_set`: ค่าไม่ซ้ำในประเภทบัญชีมี "saving", "current"\
  *ใช้เมื่อ*: ต้องการประเภทที่กำหนด
- `expect_column_distinct_values_to_equal_set`: ค่าไม่ซ้ำในประเภทบัญชีมีแค่ "saving", "current", "fixed"\
  *ใช้เมื่อ*: กำหนดชุดประเภทแน่นอน
- `expect_column_values_to_not_be_in_set`: สถานะบัญชีไม่เป็น "closed" หรือ "suspended"\
  *ใช้เมื่อ*: กรองสถานะที่ไม่ต้องการ

---

## 🔧 **ขั้นตอนการใช้งาน Data Sources**

1. **บันทึกไฟล์** 📂

   - คัดลอก YAML จากส่วน **การตั้งค่า Data Sources** ไปใส่ใน `great_expectations/great_expectations.yml`

2. **แก้ไข Credentials** 🔒

   - แทนที่ `host`, `port`, `username`, `password` ด้วยค่าจริงของระบบ

3. **ติดตั้ง Dependencies** 🛠️

   ```bash
   pip install psycopg2 pymongo vertica-python kafka-python pyspark pyarrow pandas
   ```

4. **สร้าง Expectation Suites** 📝

   - สร้าง suites สำหรับแต่ละ data source เช่น `transactions_suite`, `kafka_suite`:

     ```bash
     great_expectations suite new
     ```

5. **รัน Checkpoint** ▶️

   - ตรวจสอบข้อมูล:

     ```bash
     great_expectations checkpoint run my_checkpoint
     ```

6. **ดูผลลัพธ์** 📊

   - สร้างเอกสารผลการตรวจสอบ:

     ```bash
     great_expectations docs build
     ```

---
