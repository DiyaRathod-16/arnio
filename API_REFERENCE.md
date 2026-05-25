# Arnio API Reference

A technical reference guide to the public classes and functions within the **Arnio** library.

> **Coverage note:** This reference covers all public exports in `arnio.__all__`. If you find a public export that is missing, please open an issue or pull request.

## Arnio API Reference Index

| Category              | Components                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| :-------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Core Class**        | [**`ArFrame`**](#arframe) � Properties: [`shape`](#shape), [`columns`](#columns), [`dtypes`](#dtypes) � [`is_empty`](#is_empty) � Methods: [`memory_usage`](#memory_usage), [`preview`](#preview), [`select_columns`](#select_columns), [`select_dtypes`](#select_dtypes)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **I/O**               | [`read_csv`](#read_csv) � [`scan_csv`](#scan_csv) � [`write_csv`](#write_csv) � [`sniff_delimiter`](#sniff_delimiter)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Cleaning**          | [`cast_types`](#cast_types) � [`clean`](#clean) � [`clip_numeric`](#clip_numeric) � [`combine_columns`](#combine_columns) � [`drop_columns`](#drop_columns) � [`drop_constant_columns`](#drop_constant_columns) � [`drop_duplicates`](#drop_duplicates) � [`drop_nulls`](#drop_nulls) � [`fill_nulls`](#fill_nulls) � [`filter_rows`](#filter_rows) � [`keep_rows_with_nulls`](#keep_rows_with_nulls) � [`normalize_case`](#normalize_case) � [`normalize_unicode`](#normalize_unicode) � [`rename_columns`](#rename_columns) � [`replace_values`](#replace_values) � [`round_numeric_columns`](#round_numeric_columns) � [`safe_divide_columns`](#safe_divide_columns) � [`strip_whitespace`](#strip_whitespace) � [`trim_column_names`](#trim_column_names) � [`validate_columns_exist`](#validate_columns_exist) |
| **Conversion**        | [`from_pandas`](#from_pandas) � [`to_pandas`](#to_pandas)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **Integration**       | [`ArnioPandasAccessor`](#arniopandasaccessor)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Pipeline**          | [`pipeline`](#pipeline) � [`register_step`](#register_step)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| **Data Quality**      | [`profile`](#profile) • [`suggest_cleaning`](#suggest_cleaning) • [`auto_clean`](#auto_clean) • [`check_quality_gates`](#check_quality_gates) • [`DataQualityReport`](#dataqualityreport) • [`ColumnProfile`](#columnprofile)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **Schema Validation** | [`Schema`](#schema) • [`Field`](#field) • [`validate`](#validate) • [`ValidationResult`](#validationresult) • [`ValidationIssue`](#validationissue) • [`Int64`](#int64) • [`Float64`](#float64) • [`String`](#string) • [`Bool`](#bool) • [`Email`](#email) • [`URL`](#url) • [`CountryCode`](#countrycode) • [`DateTime`](#datetime)                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| **Custom Exceptions** | [`ArnioError`](#arnioerror) • [`CsvReadError`](#csvreaderror) • [`TypeCastError`](#typecasterror) • [`UnknownStepError`](#unknownsteperror)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

Additional coverage is documented in the sections below: [I/O and Conversion](#io-and-conversion), [Cleaning Primitives](#cleaning-primitives), [Pipeline and Step Registry](#pipeline-and-step-registry), [Quality, Schema, and Validation](#quality-schema-and-validation), and [Exceptions](#exceptions).

```python
import arnio as ar

frame = ar.from_records([
    {"id": 1, "name": "alice", "score": 95},
    {"id": 2, "name": "bob", "score": 88},
])
baseline = ar.profile(frame)
current = ar.profile(ar.from_records([
    {"id": 1, "name": "alice", "score": 95},
    {"id": 2, "name": "bob", "score": None},
]))
schema = ar.Schema({"id": ar.Int64(), "name": ar.String()})
expected = ar.Schema({"id": ar.Int64()})
observed = ar.Schema({"id": ar.Int64(), "name": ar.String()})
```

---

## Quality, Schema, and Validation

### compare_profiles

`compare_profiles(baseline, current) -> ProfileComparison`

Compares two `DataQualityReport` profiles and flags per-column drift. It returns a `ProfileComparison` object with a `drift_report` dict and a `status_counts` dict.

Raises `TypeError` when either input is not a `DataQualityReport`, and `ValueError` when the two reports do not cover the same columns.

```python
comparison = ar.compare_profiles(baseline, current)
print(comparison.status_counts)
```

### CleanStepRecord

A data class representing a single recorded cleaning step applied during an `auto_clean` run. It records the step name and the keyword arguments used for that step.

```python
clean, explanation = ar.auto_clean(frame, explain=True)
print(explanation.steps[0])
```

### CleanExplanation

A data class that explains why a specific cleaning step was suggested or applied. It captures the target column or columns and a human-readable reason for the step.

```python
clean, explanation = ar.auto_clean(frame, explain=True)
print(explanation)
```

### ProfileComparison

Return type of `compare_profiles()`. It contains `drift_report`, `status_counts`, and a `to_dict()` method. `to_dict()` embeds full left and right profiles, including sample values, so use `profile.to_dict(redact_sample_values=True)` for safer sharing.

```python
comparison = ar.compare_profiles(baseline, current)
print(comparison.drift_report["score"])
```

### QualityGateIssue

Represents a single failed quality gate check returned by `check_quality_gates()`. It records the gate name, the observed value, the threshold, and a human-readable message.

```python
result = ar.check_quality_gates(baseline, current)
print(result.issues[0].message)
```

### QualityGateResult

Return type of `check_quality_gates()`. It holds the baseline and current profiles, the collected `issues`, and the evaluated `thresholds`. The object exposes `passed`, `summary()`, `to_dict()`, `to_markdown()`, and `raise_for_failures()`.

```python
result = ar.check_quality_gates(baseline, current)
print(result.passed)
```

### Date

`Date(nullable=True, unique=False, severity="error", required_if=None) -> Field`

Schema field type that validates strict `YYYY-MM-DD` calendar dates.

Raises `ValueError` or `TypeError` for invalid field options.

```python
schema = ar.Schema({"signup_date": ar.Date(nullable=False)})
```

### CurrencyCode

`CurrencyCode(nullable=True, unique=False) -> Field`

Schema field type that validates 3-letter uppercase ISO 4217 currency codes such as `USD`, `EUR`, and `INR`.

Raises `ValueError` or `TypeError` for invalid field options.

```python
schema = ar.Schema({"currency": ar.CurrencyCode()})
```

### LanguageCode

`LanguageCode(nullable=True, unique=False, severity="error", required_if=None) -> Field`

Schema field type that validates lowercase ISO 639-1 language codes such as `en`, `hi`, and `fr`.

Raises `ValueError` or `TypeError` for invalid field options.

```python
schema = ar.Schema({"language": ar.LanguageCode()})
```

### PhoneNumber

`PhoneNumber(nullable=True, unique=False, severity="error", required_if=None) -> Field`

Schema field type that validates common international and formatted phone number strings.

Raises `ValueError` or `TypeError` for invalid field options.

```python
schema = ar.Schema({"phone": ar.PhoneNumber(nullable=False)})
```

### Regex

`Regex(pattern, nullable=True, unique=False, severity="error", required_if=None) -> Field`

Schema field type that validates string values against a regular expression pattern.

Raises `re.error` when the pattern is invalid, plus the usual field-option validation errors.

```python
schema = ar.Schema({"user_code": ar.Regex(r"^USR-\d{4}$", nullable=False)})
```

### register_validator

`register_validator(name, fn) -> None`

Registers a named custom validator function for use in `Custom` schema fields. The function receives a single value and must return `bool`.

Raises `TypeError` if `fn` is not callable and `ValueError` if `name` is not a non-empty string.

```python
ar.register_validator("positive", lambda v: v > 0)
```

### Custom

`Custom(name, *, nullable=True, unique=False, severity="error") -> Field`

Schema field type that validates values with a named custom validator registered via `register_validator()`.

Raises `ValueError` if the validator name has not been registered, plus the usual field-option validation errors.

```python
ar.register_validator("positive", lambda v: v > 0)
schema = ar.Schema({"score": ar.Custom("positive", nullable=False)})
```

### SchemaDiffEntry

A single entry in a `SchemaDiff` result describing one difference between two schemas. It stores the column name, the kind of difference, and the expected and observed values.

```python
diff = ar.diff_schema(expected, observed)
print(diff.differences[0])
```

### SchemaDiff

Return type of `diff_schema()`. It holds a list of `SchemaDiffEntry` objects and exposes `summary()` and `to_markdown()` methods.

```python
diff = ar.diff_schema(expected, observed)
print(diff.summary())
```

### diff_schema

`diff_schema(expected, observed) -> SchemaDiff`

Compares two `Schema` objects and returns a `SchemaDiff` describing columns that were added, removed, or changed between the expected and observed schemas.

Raises the same schema-construction errors as `Schema` when either input cannot be interpreted as a schema.

```python
diff = ar.diff_schema(expected, observed)
print(diff.to_markdown())
```

### schema_to_dict

`schema_to_dict(schema) -> dict`

Serializes a `Schema` object to a plain Python `dict`. It is equivalent to `schema.to_json()` parsed as a dict.

Raises `TypeError` when the input is neither a `Schema`-like object nor a plain mapping.

```python
payload = ar.schema_to_dict(schema)
```

### schema_to_yaml

`schema_to_yaml(schema) -> str`

Serializes a `Schema` object to a YAML string. This is useful for storing data contracts in version-controlled config files.

Raises `TypeError` when the input cannot be converted to a schema structure.

```python
print(ar.schema_to_yaml(schema))
```

## Exceptions

### JsonlReadError

Raised by `read_jsonl()` when a line contains invalid JSON or the file cannot be parsed as JSON Lines. The error message includes the 1-based line number of the offending line.

```python
try:
    ar.read_jsonl("bad.jsonl")
except ar.JsonlReadError as exc:
    print(exc)
```

### PipelineStepError

Raised when a registered custom Python pipeline step raises an unhandled exception during execution. It wraps the original exception with step name and index context.

```python
def boom(df):
    raise RuntimeError("boom")

ar.register_step("boom", boom)
try:
    ar.pipeline(frame, [("boom",)])
except ar.PipelineStepError as exc:
    print(exc.step_name)
```

## I/O and Conversion

### read_csv_chunked

`read_csv_chunked(path, chunk_size=10000, **kwargs) -> Iterator[ArFrame]`

Reads a CSV file in chunks and yields `ArFrame` objects of up to `chunk_size` rows. It accepts the same keyword arguments as `read_csv`, which makes it useful for large files that should not be fully loaded into memory.

Raises the same file and parse errors as `read_csv`, plus `TypeError` or `ValueError` for invalid chunk sizes or CSV options.

```python
for chunk in ar.read_csv_chunked("huge.csv"):
    process(chunk)
```

### read_jsonl

`read_jsonl(path, nrows=None, encoding="utf-8") -> ArFrame`

Parses a JSON Lines (NDJSON) file into an `ArFrame`. Blank lines are skipped, missing keys become nulls, and mixed-type columns are coerced to string. Invalid JSON is reported with a 1-based line number.

Raises `JsonlReadError` when the file cannot be parsed, including invalid JSON, decode failures, or an empty data file. Raises `ValueError` or `TypeError` for invalid arguments.

```python
frame = ar.read_jsonl("events.jsonl", nrows=1000)
```

### write_parquet

`write_parquet(frame, path, compression="snappy", row_group_size=None) -> None`

Exports an `ArFrame` to a Parquet file via `pyarrow`. Accepted compression codecs are `"snappy"` (default), `"gzip"`, `"zstd"`, `"brotli"`, and `"none"`. Install the optional extra with `pip install arnio[parquet]`.

Raises `ImportError` if `pyarrow` is not installed. Raises `ValueError` or `TypeError` for unsupported paths, codecs, or row-group sizes.

```python
ar.write_parquet(frame, "output.parquet", compression="zstd")
```

### to_arrow

`to_arrow(frame) -> pyarrow.Table`

Converts an `ArFrame` to a `pyarrow.Table` for zero-copy interop with Arrow-native tools. Requires `pyarrow`.

Raises `ImportError` if `pyarrow` is not installed and `TypeError` if the input is not an `ArFrame`.

```python
table = ar.to_arrow(frame)
```

### from_records

`from_records(records, columns=None) -> ArFrame`

Builds an `ArFrame` from a list of dicts or a list of lists/tuples. When using list-of-dicts, column names are inferred from keys. When using list-of-lists, `columns` must be supplied. Missing keys in dict records are filled with `None`, nested values raise `TypeError`, and an empty list raises `ValueError`. Also available as `ArFrame.from_records`.

Raises `TypeError` or `ValueError` for invalid record shapes or column definitions.

```python
frame = ar.from_records([{"id": 1, "name": "alice"}, {"id": 2, "name": "bob"}])
```

## Cleaning Primitives

### normalize_whitespace

`normalize_whitespace(frame) -> ArFrame`

Collapses internal runs of whitespace in string columns to a single space and trims leading/trailing spaces.

Raises the usual input-validation errors for invalid frames or other malformed inputs.

```python
clean = ar.normalize_whitespace(frame)
```

### drop_empty_columns

`drop_empty_columns(frame) -> ArFrame`

Removes columns whose values are entirely null or empty strings. Strings containing only whitespace are treated as empty.

Raises the usual input-validation errors for invalid frames or malformed inputs.

```python
clean = ar.drop_empty_columns(frame)
```

### winsorize_outliers

`winsorize_outliers(frame, lower=0.05, upper=0.95, subset=None) -> ArFrame`

Clips extreme numeric values using lower and upper quantile bounds. Non-numeric columns are ignored unless explicitly listed in `subset`. It is also available as a pipeline step.

Raises `ValueError` for invalid quantile bounds and `TypeError` for invalid arguments.

```python
clean = ar.winsorize_outliers(frame, lower=0.05, upper=0.95)
```

### coalesce_columns

`coalesce_columns(frame, subset, output_column) -> ArFrame`

Returns the first non-null value from the columns listed in `subset` into a new column named `output_column`.

Raises the usual validation errors for missing columns or invalid `subset` values.

```python
clean = ar.coalesce_columns(frame, subset=["phone", "mobile"], output_column="contact")
```

### drop_columns_matching

`drop_columns_matching(frame, pattern) -> ArFrame`

Drops all columns whose names match the given regex pattern.

Raises `ValueError` or the underlying regex error if the pattern is invalid.

```python
clean = ar.drop_columns_matching(frame, pattern="^temp_")
```

### parse_bool_strings

`parse_bool_strings(frame) -> ArFrame`

Normalizes string values such as `"yes"`, `"no"`, `"true"`, `"false"`, `"y"`, `"n"`, `"1"`, and `"0"` into boolean values. Unsupported values are left unchanged.

Raises the usual input-validation errors for invalid frames or malformed inputs.

```python
clean = ar.parse_bool_strings(frame)
```

### standardize_missing_tokens

`standardize_missing_tokens(frame) -> ArFrame`

Replaces common missing-value sentinel strings such as `"N/A"`, `"NULL"`, `"none"`, and `"-"` with real nulls. This helper runs via the Python backend.

Raises the usual input-validation errors for invalid frames or malformed inputs.

```python
clean = ar.standardize_missing_tokens(frame)
```

## Pipeline and Step Registry

### register_duckdb

`register_duckdb(frame, conn, table_name) -> None`

Registers an `ArFrame` directly as a DuckDB relation under `table_name` on the given DuckDB connection. DuckDB is an optional dependency; install it with `pip install duckdb`.

Raises `TypeError` or `ValueError` if the frame or table name is invalid.

```python
import duckdb
ar.register_duckdb(frame, duckdb.connect(), "my_table")
```

### get_builtin_step_signatures

`get_builtin_step_signatures() -> dict[str, inspect.Signature]`

Returns a mapping of built-in step names to their `inspect.Signature` objects. Use this to inspect which keyword arguments a step accepts before assembling a pipeline.

Raises no Arnio-specific exceptions.

```python
signatures = ar.get_builtin_step_signatures()
print(signatures["drop_nulls"])
```

### list_steps

`list_steps() -> list[str]`

Returns the names of all currently registered pipeline steps, including both built-in C++ steps and any registered custom Python steps.

Raises no Arnio-specific exceptions.

```python
print(ar.list_steps())
```

### PipelineContext

A context object optionally passed to custom pipeline step functions. Available attributes include `step_name: str`, `step_index: int`, and `total_steps: int`. Opt in by declaring `context=None` in the step function signature.

```python
def annotate(df, context=None):
    print(context.step_name, context.step_index)
    return df
```

### reset_steps

`reset_steps() -> None`

Resets the step registry to built-in steps only, removing all registered custom Python steps. This is useful for test isolation.

Raises no Arnio-specific exceptions.

```python
ar.reset_steps()
```

## Prerequisites

```python
import arnio as ar

df = ar.read_csv("data.csv")
```

### ArFrame

| Property                            | Return Type       |
| :---------------------------------- | :---------------- |
| <a name="columns"></a>**columns**   | `list[str]`       |
| <a name="dtypes"></a>**dtypes**     | `dict[str, str]`  |
| <a name="shape"></a>**shape**       | `tuple[int, int]` |
| <a name="is_empty"></a>**is_empty** | `bool`            |

| Method                                            | Return Type |
| :------------------------------------------------ | :---------- |
| <a name="memory_usage"></a>**memory_usage()**     | `int`       |
| <a name="preview"></a>**preview()**               | `str`       |
| <a name="select_columns"></a>**select_columns()** | `ArFrame`   |
| <a name="select_dtypes"></a>**select_dtypes()**   | `ArFrame`   |

```python
print(f"Column Names: {df.columns}")
print(f"Data Types: {df.dtypes}")
print(f"Dataset Shape: {df.shape}")
print(f"Memory: {df.memory_usage()} bytes")
print(df.preview())
df = df.select_columns(columns=["id", "name"])
df = df.select_dtypes(include=["int64", "float64"])
```

### ColumnSummary

`ColumnSummary(name, dtype, nullable)`

Schema summary for a single column, typically returned by `ArFrame.schema_summary`. It stores the column name, inferred dtype, and nullability flag, and supports equality comparison and a readable `repr`.

```python
summary = ar.ColumnSummary("email", "string", True)
print(summary.name, summary.dtype, summary.nullable)
```

---

### read_csv

Loads a CSV, TSV, or TXT file into an `ArFrame`.

```python
df = ar.read_csv("data.csv")
```

### scan_csv

Return schema (column names + inferred types) without loading data.

```python
schema = ar.scan_csv("large_dataset.csv")
```

### write_csv

Writes an `ArFrame` to a CSV file via the C++ backend.

```python
ar.write_csv(frame, "output.csv")
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `frame` | `ArFrame` | required | The data frame to write |
| `path` | `str \| os.PathLike[str]` | required | Destination file path. Supports `.csv`, `.txt`, `.tsv` |
| `delimiter` | `str` | `","` | Single character field separator |
| `write_header` | `bool` | `True` | Whether to write the column header row |
| `line_terminator` | `str` | `"\n"` | Line terminator between rows |

#### Raises

| Error | When |
|-------|------|
| `ValueError` | File extension is not `.csv`, `.txt`, or `.tsv` |
| `ValueError` | `delimiter` is not exactly one character |
| `RuntimeError` | File cannot be opened or written |

#### Examples

```python
# Default comma-separated
ar.write_csv(frame, "output.csv")

# Tab-separated
ar.write_csv(frame, "output.tsv", delimiter="\t")

# Without header row
ar.write_csv(frame, "output.csv", write_header=False)

# Windows line endings
ar.write_csv(frame, "output.csv", line_terminator="\r\n")
```

### sniff_delimiter

Sniffs and returns the field delimiter character from a CSV file.

```python
delimiter = ar.sniff_delimiter("data.csv")
```

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `path` | `str \| os.PathLike[str]` | required | Path to the CSV file |
| `encoding` | `str` | `"utf-8"` | File encoding |
| `sample_size` | `int` | `2048` | Number of bytes to sample from the start of the file for sniffing |

#### Returns

`str`
The detected delimiter (one of `","`, `";"`, `"\t"`, `"|"`).

#### Raises

| Error | When |
|-------|------|
| `TypeError` | `encoding` is not a string, or `sample_size` is not an integer |
| `ValueError` | `sample_size` is <= 0, the encoding is unknown, or the delimiter is ambiguous / tied |
| `CsvReadError` | The file is empty, or contains binary data (NUL bytes) |
| `FileNotFoundError` | The file does not exist |

#### Examples

```python
# Sniff comma-separated file
delim = ar.sniff_delimiter("comma.csv")  # returns ","

# Sniff semicolon-separated file with custom sample size
delim = ar.sniff_delimiter("semicolon.csv", sample_size=1024)  # returns ";"
```

---

### cast_types

Converts specific columns to a new data type using a mapping dictionary.

```python
df = ar.cast_types(df, {"id": "float64"})
```

### clean

A high-level wrapper that applies `strip_whitespace`, `drop_nulls`, and `drop_duplicates` in a single call.

```python
df = ar.clean(df)
```

### clip_numeric

Clip numeric values to lower and/or upper bounds.

```python
df = ar.clip_numeric(df, lower=0, upper=100)
```

### combine_columns

Combine multiple columns into a single output column.

```python
df = ar.combine_columns(df, separator=",", output_column="combined_col")
```

### drop_constant_columns

Removes columns with only one unique value.

```python
df = ar.drop_constant_columns(df)
```

### drop_columns

Removes the requested columns while preserving the order of the remaining ones.

```python
frame = ar.drop_columns(frame, ["debug_col"])
```

### drop_duplicates

Removes identical rows from the dataset.

```python
df = ar.drop_duplicates(df, keep="first")
```

### drop_nulls

Excludes rows containing empty or null fields

```python
df = ar.drop_nulls(df, subset=["email"])
```

### fill_nulls

Replaces null entry values with a designated static value.

```python
df = ar.fill_nulls(df, 0, subset=["score"])
```

### filter_rows

Subsets rows matching an evaluation operator constraint.

```python
df = ar.filter_rows(df, column="age", op=">", value=18)
```

### keep_rows_with_nulls

Keep only rows that contain at least one null/empty value.

```python
df = ar.keep_rows_with_nulls(df)
```

### normalize_case

Adjusts text casing for consistency.

```python
df = ar.normalize_case(df, case_type="title")
```

### normalize_unicode

Normalize Unicode text columns.

```python
df = ar.normalize_unicode(df, subset=["uni_col"], form="NFC")
```

### rename_columns

Modifies headers using a translation dictionary mapping old names to new names.

```python
df = ar.rename_columns(df, {"old": "new"})
```

### replace_values

Replace values based on a mapping dict.

```python
df = ar.replace_values(df, {"old_value": "new_value"}, column="name")
```

### round_numeric_columns

Round numeric columns.

```python
df = ar.round_numeric_columns(df, decimals=2)
```

### safe_divide_columns

Divide one column by another.

```python
df = ar.safe_divide_columns(
    df,
    numerator="revenue",
    denominator="cost",
    output_column="ratio"
)
```

### strip_whitespace

Trims extra spaces from the beginning and end of text entries.

```python
df = ar.strip_whitespace(df)
```

### trim_column_names

Trims leading and trailing whitespace from column names.

```python
df = ar.trim_column_names(df)
```

### validate_columns_exist

Fail early when required columns are missing.

```python
df = ar.validate_columns_exist(df, ["age"])
```

---

### from_pandas

Converts a `pandas.DataFrame` into an Arnio `ArFrame`.

### to_pandas

Converts an `ArFrame` into a `pandas.DataFrame`

```python
import pandas as pd

pdf = pd.DataFrame(data)

af = ar.from_pandas(pdf)
df = ar.to_pandas(af)
```

---

### ArnioPandasAccessor

Run Arnio preparation helpers from an existing pandas DataFrame.

---

### pipeline

Apply a sequence of cleaning steps to an `ArFrame`.

```python
ops = [
    ("strip_whitespace",),
    ("normalize_case", {"case_type": "title"}),
    ("fill_nulls", {"value": 0, "subset": ["revenue"]}),
    ("fill_nulls", {"value": "Unknown", "subset": ["name"]}),
    ("drop_duplicates",),
]
df = ar.pipeline(df, ops)
```

```python
clean, metadata = ar.pipeline(df, ops, return_metadata=True)
print(metadata["step_timings"])
```

### register_step

Extend the pipeline by adding your own custom Python functions.

```python
def custom_func(df, column):
    pass

ar.register_step("custom_func", custom_func)
```

---

### profile

Analyze an `ArFrame` and get a structural `DataQualityReport`.

Key options:
- `sample_size`: number of non-null sample values stored per column.
- `approx_top_values`: enable approximate top values for high-cardinality string columns.
- `approx_top_values_min_unique`: minimum unique count to trigger approximation.
- `approx_top_values_min_ratio`: minimum unique ratio to trigger approximation.
- `approx_top_values_sample_size`: sample size for top-value estimation.

When `approx_top_values` is enabled, `top_values` counts/ratios are computed on
the sample, and `top_values_is_approximate`, `top_values_sample_count`, and
`top_values_sample_ratio` are included in each `ColumnProfile`.

### suggest_cleaning

Examine a report or frame and get a list of recommended cleaning steps.

### auto_clean

Profile the data and immediately apply repairs.

### check_quality_gates

Compare two `DataQualityReport` objects and return a pass/fail
`QualityGateResult` for CI or monitoring workflows.

```python
baseline = ar.profile(ar.read_csv("baseline.csv"))
current = ar.profile(ar.read_csv("current.csv"))

result = ar.check_quality_gates(
    baseline,
    current,
    max_row_count_delta_ratio=0.10,
    max_null_ratio_delta=0.05,
)

print(result.passed)
print(result.to_markdown())
```

### DataQualityReport

Summary of structural data quality metrics.

#### Methods:
* **`to_html(file_path: str | None = None) -> str`**: Generates a self-contained, offline-friendly, beautiful HTML dashboard report of your dataset's metrics, columns, and cleaning suggestions. Dynamically escapes all data values to prevent XSS. If `file_path` is provided, writes the HTML output to a file.
* **`to_markdown() -> str`**: Returns a GitHub-friendly markdown representation of the report.
* **`summary() -> dict`**: Returns a high-signal dictionary representation of the report metrics.

### ColumnProfile

Detailed health check for a single column.

```python
report = ar.profile(df)
summary = report.summary()
suggestions = ar.suggest_cleaning(df)

# Export the report as a beautiful, self-contained HTML file
html_report = report.to_html(file_path="quality_report.html")

safe = ar.auto_clean(df)
print(ar.to_pandas(safe))
```

---

#### Schema

The top-level container for validation rules.

#### Field

Defines the specific constraints for a single column.

#### validate

The primary function used to check an `ArFrame` against a `Schema`. It returns a `ValidationResult`.

#### <a name="validationresult"></a>ValidationResult / <a name="validationissue"></a>ValidationIssue

The objects returned after calling `validate()`.

**Row index convention:** `ValidationIssue.row_index` is **1-based** and refers to
data rows only — the CSV header is not counted. So `row_index=1` means the first
data row, `row_index=2` means the second, and so on.

```python
# CSV content:
# name,age        ← header (not counted)
# Alice,30        ← row 1
# Bob,-1          ← row 2  ← row_index=2 will appear here for a min violation

result = ar.validate(frame, {"age": ar.Int64(min=0)})
print(result.issues[0].row_index)  # 2
```

#### Field Type Helpers

Each helper maps to a specific data type rule.

| Function                                  | Description                                     |
| :---------------------------------------- | :---------------------------------------------- |
| <a name="int64"></a>**Int64**             | Validates whole numbers.                        |
| <a name="float64"></a>**Float64**         | Validates decimal numbers.                      |
| <a name="string"></a>**String**           | Validates text.                                 |
| <a name="bool"></a>**Bool**               | Validates True/False boolean values.            |
| <a name="email"></a>**Email**             | Specialized String validator for email formats. |
| <a name="url"></a>**URL**                 | Specialized String validator for web links.     |
| <a name="countrycode"></a>**CountryCode** | Validates uppercase ISO alpha-2 country-code.   |
| <a name="datetime"></a>**DateTime**       | Validates string timestamps.                    |

---

```python
user_schema = ar.Schema({
    "id": ar.Int64(unique=True, nullable=False),
    "name": ar.String(nullable=False),
    "revenue": ar.Float64(min=180, max=1000)
})
result = ar.validate(df, user_schema)
```

---

### Custom Exceptions

| Error Name                                                               | Meaning                                                 |
| :----------------------------------------------------------------------- | :------------------------------------------------------ |
| <a name="arnioerror"></a>[**ArnioError**](#arnioerror)                   | Base exception for all Arnio errors.                    |
| <a name="csvreaderror"></a>[**CsvReadError**](#csvreaderror)             | Triggered when a CSV file cannot be read.               |
| <a name="typecasterror"></a>[**TypeCastError**](#typecasterror)          | Raised when cast_types encounters an incompatible type. |
| <a name="unknownsteperror"></a>[**UnknownStepError**](#unknownsteperror) | Triggered when a pipeline step name is not registered   |

---
