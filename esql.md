# Elasticsearch Query Language

This document contains snippets for the Elasticsearch Query Language (ES|QL) that can be used to help in hunting and detection scenarios.
ES|QL is similar to Kusto Query Language (KQL), but it's specific to Elasticsearch.

## Grok

With ES|QL you can use the `grok` processor to extract fields from unstructured data which is already stored in Elasticsearch.
The `grok` processor is a powerful tool to extract fields from unstructured data and will be familiar from Logstash or Elasticsearch Pipelines

For example you can extract Data from an existing field with a regular expression and store it in a new field.

```esql
from logs-windows.powershell*
| where powershell.file.script_block_text is not null
| grok powershell.file.script_block_text """(?<base64_data>([A-Za-z0-9+/]+={1,2}$|[A-Za-z0-9+/]{100,}))"""
| where base64_data is not null
```
