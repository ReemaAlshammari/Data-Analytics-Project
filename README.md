# Data-Engineering-Project
My university data science projects.
import pandas as pd 
import sqlite3
import requests

# Define file paths
CSV_FILE_PATH = r"C:\Users\Reema\healthcare.csv"
API_URL = "https://data.cdc.gov/resource/hksd-2xuw.json"
NEW_DB_PATH = r"C:\Users\Reema\healthcare_etl.db"
TABLE_NAME = "health_data"

def extract_csv(file_path):
    # Read CSV file
    try:
        df = pd.read_csv(file_path)
        df.columns = df.columns.str.lower()
        print("CSV Data Extracted Successfully")
        return df
    except Exception as e:
        print(f"Error reading CSV file: {e}")
        return pd.DataFrame()

def extract_api(api_url):
    # Fetch API data
    try:
        response = requests.get(api_url, timeout=10)
        response.raise_for_status()
        data = response.json()

      
        if isinstance(data, list):
            api_df = pd.DataFrame(data)
        elif isinstance(data, dict):
            api_df = pd.DataFrame([data])
        else:
            api_df = pd.DataFrame()

        api_df.columns = [str(col).lower() for col in api_df.columns]
        api_df = api_df.drop_duplicates()
        print("API Data Extracted Successfully")
        return api_df

    except Exception as e:
        print(f"Warning: Could not fetch data from API - {e}")
        return pd.DataFrame()



def clean_data(csv_df, api_df):
    # Clean and merge data
    if csv_df.empty and api_df.empty:
        print("No data to clean")
        return pd.DataFrame()

    
    csv_df.columns = [str(col).lower() for col in csv_df.columns]
    api_df.columns = [str(col).lower() for col in api_df.columns]


    print("Before Cleaning:")
    print("CSV Columns:", csv_df.columns.tolist())
    print("API Columns:", api_df.columns.tolist())

    csv_df = csv_df.drop_duplicates()
    api_df = api_df.drop_duplicates()

    merged_df = pd.concat([csv_df, api_df], ignore_index=True)
    merged_df.fillna("Unknown", inplace=True)

    print("Data Cleaned and Merged")
    return merged_df


def load_to_sqlite(df, db_path, table_name):
    # Load data into SQLite
    try:
        conn = sqlite3.connect(db_path)
        df.to_sql(table_name, conn, if_exists="replace", index=False)
        conn.close()
        print("Data Loaded into SQLite Database")
    except Exception as e:
        print(f"Error loading data into SQLite: {e}")



def main():
    print("Starting ETL process...")
    # Extract data
    csv_data = extract_csv(CSV_FILE_PATH)
    api_data = extract_api(API_URL)
    # Clean data
    cleaned_data = clean_data(csv_data, api_data)
    # Load into SQLite
    load_to_sqlite(cleaned_data, NEW_DB_PATH, TABLE_NAME)
    # Export cleaned data to CSV
    cleaned_data.to_csv(r"C:\Users\Reema\cleaned_healthcare.csv", index=False)
    print("Cleaned data saved as CSV file")
    print("ETL process completed")

if __name__ == "__main__":
    main()
