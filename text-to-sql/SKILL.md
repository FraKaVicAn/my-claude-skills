---
name: text-to-sql
description: Convert natural language questions to SQL queries using Claude — schema-aware, supports complex joins, aggregations, and multi-table queries
origin: anthropics/claude-cookbooks/capabilities/text_to_sql
tools: Read, Write, Edit, Bash
---

# Text-to-SQL Skill

Let users ask questions in plain English and get back correct SQL. Claude uses your schema to generate safe, accurate queries.

## When to Activate

- Building a natural language interface to a database
- Helping non-technical users query data
- Generating SQL for complex analytical questions
- Validating or explaining existing SQL queries

## Basic Implementation

```python
import anthropic
import sqlite3

client = anthropic.Anthropic()

def get_schema(db_path: str) -> str:
    """Extract CREATE TABLE statements from a SQLite database."""
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    cursor.execute("SELECT sql FROM sqlite_master WHERE type='table' AND sql IS NOT NULL")
    tables = cursor.fetchall()
    conn.close()
    return "\n\n".join(t[0] for t in tables)

def natural_language_to_sql(question: str, schema: str) -> str:
    """Convert a natural language question to SQL."""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        system=f"""You are an expert SQL query generator.

Database schema:
{schema}

Rules:
- Generate only valid SQL for this schema
- Use proper JOINs when querying multiple tables
- Add appropriate WHERE clauses for filtering
- Use aliases for readability
- Output ONLY the SQL query, no explanation
- Never use DROP, DELETE, UPDATE, or INSERT unless explicitly asked
- For ambiguous questions, choose the most reasonable interpretation""",
        messages=[{"role": "user", "content": question}]
    )
    sql = response.content[0].text.strip()
    # Strip markdown code fences if present
    if sql.startswith("```"):
        sql = sql.split("\n", 1)[1].rsplit("```", 1)[0].strip()
    return sql

def query_database(db_path: str, sql: str) -> list[dict]:
    """Execute SQL and return results as list of dicts."""
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()
    cursor.execute(sql)
    results = [dict(row) for row in cursor.fetchall()]
    conn.close()
    return results
```

## Full Pipeline with Explanation

```python
def ask_database(db_path: str, question: str, explain: bool = False) -> dict:
    schema = get_schema(db_path)
    sql = natural_language_to_sql(question, schema)

    result = {
        "question": question,
        "sql": sql,
    }

    try:
        result["data"] = query_database(db_path, sql)
        result["success"] = True
    except Exception as e:
        result["success"] = False
        result["error"] = str(e)

    if explain:
        # Ask Claude to explain the results
        explain_response = client.messages.create(
            model="claude-haiku-4-5",
            max_tokens=512,
            messages=[{
                "role": "user",
                "content": f"Question: {question}\nSQL: {sql}\nResults: {result.get('data', [])[:5]}\n\nExplain these results in plain English."
            }]
        )
        result["explanation"] = explain_response.content[0].text

    return result
```

## With Tool Use (SQL Execution Tool)

```python
sql_tool = {
    "name": "execute_sql",
    "description": "Execute a SQL query against the database and return results.",
    "input_schema": {
        "type": "object",
        "properties": {
            "sql": {"type": "string", "description": "The SQL query to execute"},
            "explanation": {"type": "string", "description": "Brief explanation of what this query does"}
        },
        "required": ["sql"]
    }
}

def interactive_sql_agent(question: str, schema: str, db_path: str) -> str:
    """Agent that can run multiple queries to answer complex questions."""
    messages = [{"role": "user", "content": f"Schema:\n{schema}\n\nQuestion: {question}"}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6", max_tokens=1024,
            system="You are a SQL expert. Use the execute_sql tool to query the database. Run multiple queries if needed.",
            tools=[sql_tool], messages=messages
        )
        if response.stop_reason == "end_turn":
            return next(b.text for b in response.content if b.type == "text")

        messages.append({"role": "assistant", "content": response.content})
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                try:
                    data = query_database(db_path, block.input["sql"])
                    content = str(data[:20])  # limit rows
                except Exception as e:
                    content = f"Error: {e}"
                tool_results.append({"type": "tool_result", "tool_use_id": block.id, "content": content})
        messages.append({"role": "user", "content": tool_results})
```

## Schema Tips

Always provide:
- Full `CREATE TABLE` statements with column types
- Foreign key relationships
- Sample values for enum/categorical columns
- Table descriptions if names aren't self-explanatory

```python
schema_with_context = f"""{schema}

Additional context:
- 'orders.status' values: 'pending', 'shipped', 'delivered', 'cancelled'
- 'products.category' values: 'electronics', 'clothing', 'food'
- Dates are stored as ISO strings (YYYY-MM-DD)
"""
```

## Links

- [Cookbook: capabilities/text_to_sql/](https://github.com/anthropics/claude-cookbooks/tree/main/capabilities/text_to_sql)
- [Cookbook: misc/how_to_make_sql_queries.ipynb](https://github.com/anthropics/claude-cookbooks/blob/main/misc/how_to_make_sql_queries.ipynb)
