# Museum Exhibition Data Pipeline (Week 1)

A batch ETL (Extract, Transform, Load) pipeline that processes museum exhibition data and visitor feedback from AWS S3 into a normalized PostgreSQL database.

## Overview

This pipeline orchestrates three stages:
1. **Extract** - Downloads JSON exhibition files and CSV kiosk data from AWS S3
2. **Transform** - Combines and normalizes data into single CSV files
3. **Load** - Populates PostgreSQL tables using staging tables and foreign key relationships

The pipeline handles:
- Exhibition metadata (name, description, department, floor)
- Visitor ratings (0-4 scale) → `review` table
- Incident reports (help requests) → `incident` table

## Prerequisites

- Python 3.8+
- PostgreSQL database (schema must be created first)
- AWS S3 access with credentials
- Required packages: `boto3`, `psycopg2`, `pandas`, `python-dotenv`

## Installation

```bash
pip install -r requirements.txt
```

## Configuration

Create a `.env` file in the project directory:

```env
# AWS credentials
AWS_ACCESS_KEY=your_aws_access_key
AWS_SECRET_KEY=your_aws_secret_key

# Database configuration
DB_NAME=museum
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_HOST=your_db_host
DB_PORT=5432
```

## Usage

### Full pipeline with database load

```bash
python pipeline.py --push
```

### Transform only (no database push)

```bash
python pipeline.py
```

### Skip S3 extraction (use local files)

```bash
python pipeline.py --skip-extract --push
```

## Database Schema Setup (First Time Only)

```bash
psql -h $DB_HOST -U $DB_USER -d museum -f schema.sql
```

After initial setup, the schema is not recreated on subsequent pipeline runs.

## Data Flow

**Exhibition Data:**
- JSON files from S3 → normalized DataFrame → `combined_exhibition_data.csv`
- Loaded via staging table → populates `exhibition`, `department`, `floor`, `floor_assignment` tables

**Kiosk Data:**
- CSV files from S3 → concatenated → sorted by timestamp → `combined_museum_data.csv`
- Loaded via staging table → split into `review` (ratings) and `incident` (help requests)

## Project Structure

```
├── pipeline.py              # Main orchestrator
├── extract.py               # S3 download logic
├── transform.py             # Data combining and normalization
├── load.py                  # Database staging and insertion
├── schema.sql               # Database schema (one-time setup)
├── requirements.txt         # Python dependencies
├── bucket_data/             # Local S3 mirror
├── combined_data/           # Intermediate CSV files
└── pipeline.log             # Pipeline execution logs
```

## Logging

Logs are written to both the console and `pipeline.log`, including:
- Extract/transform/load progress
- File counts and data volume
- Database connection and insert operations
- Any errors or validation issues

## Notes

- S3 files must start with `lmnh` prefix (enforced in extract)
- Site IDs in CSVs are zero-indexed; exhibition IDs are one-indexed (automatic conversion)
- Incident detection: `val = -1` indicates a help request/emergency
- The `--push` flag is required to write data to the database
