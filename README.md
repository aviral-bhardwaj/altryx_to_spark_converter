# Alteryx to PySpark Converter

A powerful tool to automatically convert Alteryx workflows (.yxmd files) into production-ready PySpark code for Databricks. This converter intelligently analyzes Alteryx workflows and generates clean, idiomatic PySpark code with proper data flow tracking and container organization.

## 🚀 Features

### Core Capabilities
- **Container-Aware Conversion**: Preserves workflow organization with proper container hierarchy
- **Intelligent Connection Tracking**: Accurately tracks multiple outputs (Filter T/F, Join J/L/R, Unique U/D)
- **Advanced Expression Conversion**: Converts 95%+ of Alteryx expressions to PySpark equivalents
- **Topological Sorting**: Ensures tools are executed in correct dependency order
- **Workflow Validation**: Detects disconnected tools, circular dependencies, and structural issues
- **Unity Catalog Support**: Native support for Unity Catalog tables and Volumes

### Enhanced Features (Enhanced Version)
- **50+ Alteryx Function Mappings**: Comprehensive function library including date, string, math, and aggregation functions
- **Nested IF/THEN/ELSE Support**: Recursive parser handles complex conditional logic
- **Execution Plan Visualization**: Generates visual data flow diagrams
- **Environment Variable Support**: Configurable paths using environment variables
- **Validation Warnings**: Pre-generation checks for workflow structure
- **Comprehensive Logging**: Debug-friendly logging throughout conversion

## 📋 Supported Alteryx Tools

| Tool Type | Status | PySpark Equivalent | Notes |
|-----------|--------|-------------------|-------|
| **Data Input/Output** |
| Input Data | ✅ Full | `spark.read.table()` / `spark.read.format()` | Supports Unity Catalog, Volumes, and file paths |
| Output Data | ✅ Full | `.write.mode().saveAsTable()` | Configurable output destinations |
| Text Input | ✅ Full | `spark.createDataFrame()` | Preserves data exactly |
| Browse | ✅ Full | `display()` / logging | Visual output for debugging |
| **Data Preparation** |
| Select | ✅ Full | `.select()` / `.withColumnRenamed()` | Column selection and renaming |
| Filter | ✅ Full | `.filter()` with True/False outputs | Proper multi-output tracking |
| Formula | ✅ Full | `.withColumn()` | Complex expression conversion |
| Sort | ✅ Full | `.orderBy()` | Multiple column sorting |
| Sample | ✅ Full | `.sample()` / `.limit()` | Various sampling methods |
| Unique | ✅ Full | `.dropDuplicates()` with U/D outputs | Duplicate detection |
| Find Replace | ✅ Full | `.withColumn()` / `regexp_replace()` | Pattern-based replacement |
| **Data Joining** |
| Join | ✅ Full | `.join()` with J/L/R outputs | All join types, multi-key |
| Union | ✅ Full | `.union()` / `.unionByName()` | Column matching |
| **Data Aggregation** |
| Summarize | ✅ Full | `.groupBy().agg()` | Multiple aggregations |
| Cross Tab | ⚠️ Partial | `.groupBy().pivot()` | Basic pivot support |
| **Text Processing** |
| Text to Columns | ✅ Full | `.split()` / `regexp_extract()` | Delimiter-based splitting |
| RegEx | ⚠️ Partial | `regexp_extract()` / `regexp_replace()` | Basic regex operations |
| **In-DB Tools** |
| In-DB Select | ✅ Full | `.select()` | Same as regular Select |
| In-DB Filter | ✅ Full | `.filter()` | Same as regular Filter |
| In-DB Join | ✅ Full | `.join()` | Same as regular Join |
| In-DB Stream Out | ✅ Full | DataFrame pass-through | No special handling needed |
| **Other** |
| Macro | ⚠️ Limited | Custom implementation | Requires manual intervention |
| Python Tool | ❌ Not Supported | N/A | Use PySpark UDFs instead |
| R Tool | ❌ Not Supported | N/A | Use SparkR or PySpark UDFs |

**Legend**: ✅ Full Support | ⚠️ Partial Support | ❌ Not Supported

## 🔧 Installation & Setup

### Prerequisites
- Databricks Runtime 11.3 LTS or higher
- Unity Catalog enabled (recommended)
- Access to upload files to Databricks Volumes or DBFS

### Quick Start

1. **Upload the Converter to Databricks**:
   - Import `Alteryx_to_PySpark_Converter_Enhanced.py` as a Databricks notebook
   - Or use `Alteryx_to_PySpark_Converter.py` for the simpler version

2. **Upload Your Alteryx Workflow**:
   ```python
   # Upload .yxmd file to Volumes
   # Example path: /Volumes/catalog/schema/volume/workflows/my_workflow.yxmd
   ```

3. **Configure and Run**:
   ```python
   # Set your workflow path
   workflow_file_path = "/Volumes/catalog/schema/volume/workflows/my_workflow.yxmd"
   
   # Run all cells in the converter notebook
   ```

## 📖 Usage Guide

### Basic Usage (Simple Version)

```python
# 1. Import the converter
from Alteryx_to_PySpark_Converter import AlteryxWorkflowParser, PySparkCodeGenerator

# 2. Parse the workflow
workflow_file_path = "/Volumes/catalog/schema/volume/my_workflow.yxmd"
parser = AlteryxWorkflowParser(workflow_file_path)
workflow = parser.parse()

# 3. Generate PySpark code
generator = PySparkCodeGenerator(workflow)
generated_code = generator.generate()

# 4. Save or display the generated code
print(generated_code)
```

### Advanced Usage (Enhanced Version)

```python
# 1. Parse and validate
from Alteryx_to_PySpark_Converter_Enhanced import AlteryxWorkflowParser, PySparkCodeGenerator

parser = AlteryxWorkflowParser(workflow_file_path)
workflow = parser.parse()

# 2. Generate with validation
generator = PySparkCodeGenerator(workflow)

# Check for warnings before generation
validation_warnings = generator._validate_workflow()
if validation_warnings:
    print("⚠️ Validation Warnings:")
    for warning in validation_warnings:
        print(f"  - {warning}")

# 3. Generate code
generated_code = generator.generate()

# 4. Review execution plan
execution_plan = generator._generate_execution_plan()
print(execution_plan)

# 5. Save to file
with open("/Workspace/Users/your_user/converted_workflow.py", "w") as f:
    f.write(generated_code)
```

### Configuration Options

The converter supports environment variables for flexible configuration:

```python
# Set environment variables (optional)
import os

os.environ["INPUT_CATALOG"] = "raw_data"
os.environ["INPUT_SCHEMA"] = "bronze"
os.environ["OUTPUT_CATALOG"] = "processed_data"
os.environ["OUTPUT_SCHEMA"] = "silver"
os.environ["INPUT_VOLUME"] = "/Volumes/raw_data/bronze/landing"
os.environ["OUTPUT_VOLUME"] = "/Volumes/processed_data/silver/output"

# These will be picked up by the generated code
```

## 🎯 Output Structure

The converter generates a well-structured Databricks notebook with:

### 1. Header & Metadata
```python
# Databricks notebook source
# MAGIC %md
# MAGIC # Workflow Name
# MAGIC ## Converted from Alteryx Workflow
# MAGIC **Conversion Date**: 2024-01-15 10:30:00
# MAGIC 
# MAGIC ### Workflow Summary
# MAGIC - **Containers**: 5
# MAGIC - **Tools**: 42
# MAGIC - **Connections**: 58
```

### 2. Configuration Section
```python
# COMMAND ----------
# Configuration
config = {
    # User constants from Alteryx
    "reporting_date": "2024-01-01",
    "max_records": 1000000,
    
    # Environment paths
    "input_catalog": "raw",
    "output_catalog": "processed",
    # ... more configuration
}
```

### 3. Container-Organized Code
```python
# COMMAND ----------
# MAGIC %md
# MAGIC ## Container: Data Preparation
# MAGIC **Tools**: 8 | **Inputs**: External Source | **Outputs**: df_15_prepared

# COMMAND ----------
# === Input Data (Tool ID: 1) ===
df_1 = spark.read.table("catalog.schema.source_table")
log_df(df_1, "df_1 [Input Data]")

# COMMAND ----------
# === Filter (Tool ID: 2) | Active Records ===
filter_condition = (col("Status") == "Active") & (col("Amount") > 0)
df_2_true = df_1.filter(filter_condition)
df_2_false = df_1.filter(~filter_condition)
log_df(df_2_true, "df_2_true [Filter → True]")
log_df(df_2_false, "df_2_false [Filter → False]")
```

### 4. Execution Summary
```python
# COMMAND ----------
# Execution Summary
print("=" * 60)
print("Conversion Complete!")
print(f"Generated at: {datetime.now()}")
print("=" * 60)
```

## 🔍 Expression Conversion Examples

The converter handles complex Alteryx expressions:

### Simple Expressions
```python
# Alteryx: [CustomerID] = "C001"
# PySpark: col("CustomerID") == "C001"

# Alteryx: [Amount] > 100 AND [Status] = "Active"
# PySpark: (col("Amount") > 100) & (col("Status") == "Active")
```

### IF/THEN/ELSE Logic
```python
# Alteryx: IF [Amount] > 1000 THEN "High" ELSE "Low" ENDIF
# PySpark: when(col("Amount") > 1000, lit("High")).otherwise(lit("Low"))

# Alteryx: IF [Status] = "A" THEN 1 ELSEIF [Status] = "B" THEN 2 ELSE 0 ENDIF
# PySpark: when(col("Status") == "A", lit(1)).when(col("Status") == "B", lit(2)).otherwise(lit(0))
```

### Function Conversions
```python
# Alteryx: Upper([Name])
# PySpark: upper(col("Name"))

# Alteryx: DateTimeYear([OrderDate])
# PySpark: year(col("OrderDate"))

# Alteryx: Left([CustomerID], 3)
# PySpark: substring(col("CustomerID"), 1, 3)

# Alteryx: IFNULL([Amount], 0)
# PySpark: coalesce(col("Amount"), lit(0))
```

## ⚠️ Known Limitations

### Not Fully Supported
1. **Spatial Tools**: Spatial analysis tools require custom implementation
2. **Macros**: Custom macros need manual conversion
3. **Python/R Tools**: Require rewriting as PySpark UDFs
4. **Some Reporting Tools**: Chart, Map, Layout tools don't apply to PySpark
5. **Complex RegEx**: Some advanced regex patterns may need adjustment

### Manual Review Required
- Input/Output paths need verification
- Join keys should be validated
- Complex formulas should be tested
- Data types may need explicit casting

## 🐛 Troubleshooting

### Common Issues

**Issue**: "DataFrame not found" errors
- **Solution**: Check tool ordering and connection tracking. Run validation before generation.

**Issue**: Expression conversion failures
- **Solution**: Check the generated code for TODO comments. Some complex expressions may need manual adjustment.

**Issue**: Missing data after join
- **Solution**: Verify join keys match on both sides. Check for null values.

**Issue**: Slow performance
- **Solution**: Add `.cache()` for reused DataFrames. Consider broadcasting smaller tables.

### Debug Mode

Enable detailed logging:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)
```

## 📝 Best Practices

### Before Conversion
1. ✅ Validate your Alteryx workflow runs successfully
2. ✅ Document any custom macros or complex logic
3. ✅ Note input/output file paths
4. ✅ Review tool annotations for context

### After Conversion
1. ✅ Review all TODO comments
2. ✅ Validate input/output paths
3. ✅ Test with sample data first
4. ✅ Review execution plan
5. ✅ Add error handling where needed
6. ✅ Optimize for performance (caching, broadcasting)

### Code Quality
1. ✅ Use Unity Catalog for table management
2. ✅ Implement proper error handling
3. ✅ Add data quality checks
4. ✅ Document complex transformations
5. ✅ Use version control for generated code

## 🤝 Contributing

Contributions are welcome! Areas for improvement:
- Additional tool type support
- Enhanced expression conversion
- Performance optimizations
- Documentation improvements
- Test coverage

## 📄 License

This project is provided as-is for use in converting Alteryx workflows to PySpark.

## 📞 Support

For issues or questions:
1. Check the Troubleshooting section
2. Review validation warnings in generated code
3. Open an issue in the repository

## 🔄 Version History

### Version 2.0 (Enhanced) - Current
- ✨ 50+ function mappings
- ✨ Recursive IF/THEN/ELSE parser
- ✨ Workflow validation and cycle detection
- ✨ Execution plan visualization
- ✨ Unity Catalog and Volumes support
- ✨ Environment variable configuration

### Version 1.0 (Simple)
- ✅ Basic tool conversion
- ✅ Container organization
- ✅ Connection tracking
- ✅ Simple expression conversion

---

**Note**: This converter is designed to accelerate migration from Alteryx to Databricks. While it handles most common scenarios automatically, complex workflows may require manual adjustment. Always test generated code thoroughly before production use.