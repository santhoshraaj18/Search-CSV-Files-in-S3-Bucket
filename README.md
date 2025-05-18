# Search-CSV-Files-in-S3-Bucket

Here is a README.md file that explains the code and concepts used in your Jupyter Notebook:

# Searching CSV Files in an AWS S3 Bucket

This project provides a Python script designed to search for specific keywords within CSV files stored in an AWS S3 bucket. It leverages the `boto3` library to interact with S3 and the `tabulate` library for presenting search results in a readable format. The script is structured to work efficiently by using multithreading to process multiple CSV files concurrently.

## Concepts Used

*   **AWS S3:** Amazon Simple Storage Service (S3) is a scalable object storage service. This script interacts with an S3 bucket to access and read CSV files.
*   **Boto3:** The AWS SDK for Python (Boto3) allows Python developers to create, configure, and manage AWS services, such as S3.
*   **CSV Processing:** The `csv` module in Python is used to parse and read data from CSV files.
*   **Regular Expressions (`re`):** Regular expressions are used in the `clean_input` function to clean and standardize the input search sentence.
*   **String Manipulation:** Standard Python string methods are used for tasks like converting text to lowercase, splitting strings, and checking for substring presence.
*   **`io.StringIO`:** This class is used to treat a string in memory as a file, allowing the `csv` module to read directly from the data fetched from S3 without needing to save it to a local file first.
*   **Multithreading (`concurrent.futures.ThreadPoolExecutor`):** To improve performance when searching across multiple CSV files, the script uses a thread pool to execute the `search_csv` function concurrently for different files. This allows the script to read and process multiple files simultaneously.
*   **Tabulate:** The `tabulate` library is used to format the search results into a clear, grid-like table for easy readability.
*   **Error Handling (Implicit):** The code includes a `try-except` block when decoding the S3 object data to handle potential `UnicodeDecodeError` and fall back to a different encoding (ISO-8859-1) if UTF-8 fails. It also includes a `try-except` block around the future result to catch errors during file processing and prevent the entire script from stopping.

## Setup and Installation

1.  **Install required libraries:**
Use code with caution
bash pip install boto3 pip install tabulate

2.  **Configure AWS Credentials:**
    You need to provide your AWS access key ID, secret access key, and the region name where your S3 bucket is located. Replace the placeholder values in the script with your actual credentials. **Note:** Storing credentials directly in code is not recommended for production environments. Consider using AWS IAM roles or environment variables for better security.
Use code with caution
python aws_access_key_id = 'YOUR_AWS_ACCESS_KEY_ID' aws_secret_access_key = 'YOUR_AWS_SECRET_ACCESS_KEY' region_name = 'YOUR_AWS_REGION_NAME'

3.  **Specify S3 Bucket Name:**
    Replace the placeholder `bucket_name` with the name of your S3 bucket.
Use code with caution
python bucket_name = 'your-s3-bucket-name'

4.  **Specify Search Keywords:**
    Provide the sentence containing the keywords you want to search for. The `clean_input` function will process this sentence to extract individual keywords.
Use code with caution
python input_sentence = "your search terms here"

## How to Run the Script

1.  Save the code as a Python file (e.g., `search_s3_csv.py`) or run it directly in a Jupyter Notebook or Colab environment.
2.  Ensure you have completed the Setup and Installation steps.
3.  Execute the Python script.
Use code with caution
bash python search_s3_csv.py

## Code Explanation

*   **`clean_input(sentence)`:**
    *   Takes a sentence as input.
    *   Uses regular expressions (`re.sub`) to replace multiple spaces with a single space and `strip()` to remove leading/trailing whitespace.
    *   Converts the sentence to lowercase.
    *   Splits the cleaned sentence into a list of keywords based on spaces.
    *   Returns the list of keywords.

*   **`search_csv_s3(bucket_name, keywords, aws_access_key_id, aws_secret_access_key, region_name)`:**
    *   Initializes a `boto3` S3 client using the provided credentials and region.
    *   Lists all objects in the specified S3 bucket using `s3.list_objects_v2`.
    *   Filters the list to include only objects with a '.csv' extension (case-insensitive).
    *   Constructs a list of S3 file paths (e.g., "s3://bucket-name/file.csv").
    *   Sets the maximum number of concurrent threads for processing files.
    *   Uses `concurrent.futures.ThreadPoolExecutor` to create a thread pool.
    *   Submits the `search_csv` function to the executor for each file path, allowing multiple files to be processed simultaneously.
    *   Iterates through the completed futures using `concurrent.futures.as_completed`.
    *   Retrieves the result of each future. If a result is returned (meaning keywords were found), it's appended to `all_results`.
    *   Includes a `try-except` block to catch and suppress errors that might occur during the processing of an individual file.
    *   Returns a list of results, where each result is a tuple containing the file path and a list of matching rows (including the header).

*   **`search_csv(file_path, keywords, aws_access_key_id, aws_secret_access_key, region_name)`:**
    *   Initializes a `boto3` S3 client (re-initializing here is less efficient if called repeatedly for the same credentials and region; ideally, the client would be created once in `search_csv_s3` and passed to this function).
    *   Parses the bucket name and key from the S3 file path.
    *   Retrieves the S3 object data using `s3.get_object`.
    *   Attempts to decode the object's body as UTF-8. If a `UnicodeDecodeError` occurs, it falls back to decoding as ISO-8859-1.
    *   Uses `io.StringIO` to wrap the decoded data, making it readable by the `csv` module.
    *   Creates a `csv.reader` to iterate over the rows of the CSV data.
    *   Reads the header row and adds it to the `results` list.
    *   Iterates through the remaining rows:
        *   For each row, it checks if *all* the specified keywords are present in *any* of the columns (case-insensitive).
        *   If all keywords are found in a row, the row is added to the `results` list.
    *   If the `results` list contains more than just the header row (meaning at least one matching row was found), it returns a tuple of the file path and the `results` list.
    *   If no matching rows are found (only the header is in `results`), it returns `None`.

## Example Usage

The script demonstrates how to use the functions with example AWS credentials, bucket name, and search sentence. It then iterates through the results and prints them in a formatted table for each file where matches were found.
Use code with caution
python

Example usage:
bucket_name = 'test2bucket13052024' # Replace with your S3 bucket name input_sentence = "Mumbai" keywords = clean_input(input_sentence)

AWS credentials
aws_access_key_id = 'YOUR_AWS_ACCESS_KEY_ID' aws_secret_access_key = 'YOUR_AWS_SECRET_ACCESS_KEY' region_name = 'YOUR_AWS_REGION_NAME'

search_results = search_csv_s3(bucket_name, keywords, aws_access_key_id, aws_secret_access_key, region_name)

for result in search_results: if result: file_path, results = result print("Results for file:", file_path) print(tabulate(results, headers="firstrow", tablefmt="grid")) print()

Remember to replace the placeholder values in the example usage section with your actual AWS credentials, bucket name, and the keywords you want to search for
