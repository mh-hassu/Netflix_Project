# Netflix Data Engineering Project

A data engineering project that processes and analyzes Netflix titles data using Python, Pandas, and SQL Server.

## Project Overview

This project demonstrates a complete ETL (Extract, Transform, Load) pipeline for Netflix content data. It includes data ingestion from CSV to SQL Server, data cleaning, normalization, and analytical queries for business insights.

## Dataset

The dataset (`netflix_titles.csv`) contains information about movies and TV shows available on Netflix, including:
- **show_id**: Unique identifier for each title
- **type**: Movie or TV Show
- **title**: Title name
- **director**: Director(s)
- **cast**: Cast members
- **country**: Country of origin
- **date_added**: Date added to Netflix
- **release_year**: Original release year
- **rating**: Content rating (e.g., PG-13, TV-MA)
- **duration**: Duration in minutes or seasons
- **listed_in**: Genre categories
- **description**: Brief description

## Project Structure

```
Netflix project/
|-- netflix_titles.csv          # Raw dataset
|-- Netflix Project.ipynb       # Jupyter notebook for ETL pipeline
|-- create.sql                  # SQL table creation script
|-- netfliz sql code file.sql   # Data cleaning and analysis queries
|-- README.md                   # Project documentation
```

## Technologies Used

- **Python 3.x** with Pandas and SQLAlchemy
- **Microsoft SQL Server** (MSSQL)
- **ODBC Driver 17 for SQL Server**

## Setup & Installation

### Prerequisites

1. Python 3.x installed
2. Microsoft SQL Server installed and running
3. ODBC Driver 17 for SQL Server

### Install Python Dependencies

```bash
pip install pandas sqlalchemy pyodbc
```

### Database Configuration

Update the connection string in the Jupyter notebook to match your SQL Server configuration:

```python
engine = sal.create_engine('mssql://YOUR_SERVER_NAME/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
```

## ETL Pipeline

### 1. Data Extraction
The Jupyter notebook reads the Netflix titles CSV file using Pandas:
```python
df = pd.read_csv("netflix_titles.csv")
```

### 2. Data Loading
Data is loaded into SQL Server using SQLAlchemy:
```python
df.to_sql('netflix_raw', con=conn, index=False, if_exists='append', dtype={'title': NVARCHAR(200)})
```

### 3. Table Creation
The `create.sql` script defines the schema for the `netflix_raw` table with appropriate data types including NVARCHAR for handling foreign characters (Korean, Japanese, etc.).

## Data Processing

### Data Cleaning Steps
1. **Remove Duplicates**: Using CTEs and ROW_NUMBER() to identify and remove duplicate entries
2. **Handle NULL Values**: Populating missing country and duration values using director-based inference
3. **Date Conversion**: Converting string dates to proper DATE type
4. **Data Type Fixes**: Handling duration values that were incorrectly stored in rating column

### Data Normalization
The project creates normalized tables for:
- `netflix` - Main clean table
- `netflix_genre` - Normalized genre data (split from listed_in column)
- `netflix_country` - Normalized country data
- `netflix_directors` - Normalized director data

## Analysis Queries

The SQL file includes several analytical queries:

1. **Director Productivity**: Count of movies and TV shows per director for those who create both
2. **Comedy Movies by Country**: Finding the country with the most comedy movies
3. **Yearly Top Directors**: Directors with maximum movies released each year
4. **Average Movie Duration by Genre**: Calculating average movie length per genre
5. **Horror & Comedy Directors**: Finding directors who created both genres

## Usage

1. Execute `create.sql` to create the database table
2. Run the Jupyter notebook to load data into SQL Server
3. Execute queries from `netfliz sql code file.sql` for data analysis

## Example Query Results

### Top Director per Year
```sql
-- Find the director with the most movies released each year
WITH cte AS (
    SELECT nd.director, YEAR(date_added) as date_year, COUNT(n.show_id) as no_of_movies
    FROM netflix n
    INNER JOIN netflix_directors nd ON n.show_id = nd.show_id
    WHERE type = 'Movie'
    GROUP BY nd.director, YEAR(date_added)
)
```

### Average Duration by Genre
```sql
SELECT ng.genre, AVG(CAST(REPLACE(duration, ' min', '') AS int)) as avg_duration
FROM netflix n
INNER JOIN netflix_genre ng ON n.show_id = ng.show_id
WHERE type = 'Movie'
GROUP BY ng.genre
```

## Key Insights

- Handles international content with foreign characters (Korean, Japanese titles)
- Normalizes multi-valued attributes (genres, countries, directors)
- Provides comprehensive analytics for content strategy decisions

## License

This project uses the Netflix Titles dataset which is publicly available for educational purposes.