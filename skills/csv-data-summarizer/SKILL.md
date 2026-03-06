---
name: csv-data-summarizer
description: Analyzes CSV files, generates summary stats, and plots quick visualizations using Python and pandas.
metadata:
  version: 2.1.0
  dependencies: python>=3.8, pandas>=2.0.0, matplotlib>=3.7.0, seaborn>=0.12.0
---

# CSV Data Summarizer

This Skill analyzes CSV files and provides comprehensive summaries with statistical insights and visualizations.

## When to Use This Skill

Claude should use this Skill whenever the user:
- Uploads or references a CSV file
- Asks to summarize, analyze, or visualize tabular data
- Requests insights from CSV data
- Wants to understand data structure and quality

## How It Works

## ⚠️ CRITICAL BEHAVIOR REQUIREMENT ⚠️

**DO NOT ASK THE USER WHAT THEY WANT TO DO WITH THE DATA.**
**DO NOT OFFER OPTIONS OR CHOICES.**
**DO NOT SAY "What would you like me to help you with?"**
**DO NOT LIST POSSIBLE ANALYSES.**

**IMMEDIATELY AND AUTOMATICALLY:**
1. Run the comprehensive analysis
2. Generate ALL relevant visualizations
3. Present complete results
4. NO questions, NO options, NO waiting for user input

**THE USER WANTS A FULL ANALYSIS RIGHT AWAY - JUST DO IT.**

### Automatic Analysis Steps:

**The skill intelligently adapts to different data types and industries by inspecting the data first, then determining what analyses are most relevant.**

1. **Load and inspect** the CSV file into pandas DataFrame
2. **Identify data structure** - column types, date columns, numeric columns, categories
3. **Determine relevant analyses** based on what's actually in the data:
   - **Sales/E-commerce data** (order dates, revenue, products): Time-series trends, revenue analysis, product performance
   - **Customer data** (demographics, segments, regions): Distribution analysis, segmentation, geographic patterns
   - **Financial data** (transactions, amounts, dates): Trend analysis, statistical summaries, correlations
   - **Operational data** (timestamps, metrics, status): Time-series, performance metrics, distributions
   - **Survey data** (categorical responses, ratings): Frequency analysis, cross-tabulations, distributions
   - **Generic tabular data**: Adapts based on column types found

4. **Only create visualizations that make sense** for the specific dataset:
   - Time-series plots ONLY if date/timestamp columns exist
   - Correlation heatmaps ONLY if multiple numeric columns exist
   - Category distributions ONLY if categorical columns exist
   - Histograms for numeric distributions when relevant
   
5. **Generate comprehensive output** automatically including:
   - Data overview (rows, columns, types)
   - Key statistics and metrics relevant to the data type
   - Missing data analysis
   - Multiple relevant visualizations (only those that apply)
   - Actionable insights based on patterns found in THIS specific dataset
   
6. **Present everything** in one complete analysis - no follow-up questions

**Example adaptations:**
- Healthcare data with patient IDs → Focus on demographics, treatment patterns, temporal trends
- Inventory data with stock levels → Focus on quantity distributions, reorder patterns, SKU analysis  
- Web analytics with timestamps → Focus on traffic patterns, conversion metrics, time-of-day analysis
- Survey responses → Focus on response distributions, demographic breakdowns, sentiment patterns

### Behavior Guidelines

✅ **CORRECT APPROACH - SAY THIS:**
- "I'll analyze this data comprehensively right now."
- "Here's the complete analysis with visualizations:"
- "I've identified this as [type] data and generated relevant insights:"
- Then IMMEDIATELY show the full analysis

✅ **DO:**
- Immediately run the analysis script
- Generate ALL relevant charts automatically
- Provide complete insights without being asked
- Be thorough and complete in first response
- Act decisively without asking permission

❌ **NEVER SAY THESE PHRASES:**
- "What would you like to do with this data?"
- "What would you like me to help you with?"
- "Here are some common options:"
- "Let me know what you'd like help with"
- "I can create a comprehensive analysis if you'd like!"
- Any sentence ending with "?" asking for user direction
- Any list of options or choices
- Any conditional "I can do X if you want"

❌ **FORBIDDEN BEHAVIORS:**
- Asking what the user wants
- Listing options for the user to choose from
- Waiting for user direction before analyzing
- Providing partial analysis that requires follow-up
- Describing what you COULD do instead of DOING it

### Usage

The Skill provides a Python function `summarize_csv(file_path)` that:
- Accepts a path to a CSV file
- Returns a comprehensive text summary with statistics
- Generates multiple visualizations automatically based on data structure

### Example Prompts

> "Here's `sales_data.csv`. Can you summarize this file?"

> "Analyze this customer data CSV and show me trends."

> "What insights can you find in `orders.csv`?"

### Example Output

**Dataset Overview**
- 5,000 rows × 8 columns  
- 3 numeric columns, 1 date column  

**Summary Statistics**
- Average order value: $58.2  
- Standard deviation: $12.4
- Missing values: 2% (100 cells)

**Insights**
- Sales show upward trend over time
- Peak activity in Q4
*(Attached: trend plot)*

## Files

- `analyze.py` - Core analysis logic
- `requirements.txt` - Python dependencies
- `resources/sample.csv` - Example dataset for testing
- `resources/README.md` - Additional documentation

## Notes

- Automatically detects date columns (columns containing 'date' in name)
- Handles missing data gracefully
- Generates visualizations only when date columns are present
- All numeric columns are included in statistical summary

