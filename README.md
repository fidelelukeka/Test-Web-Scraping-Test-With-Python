# Python Web Scraper

## Description

This Python script is an asynchronous web scraper that retrieves content from multiple web pages. It uses the `asyncio` and `aiohttp` libraries to perform asynchronous HTTP requests, allowing for concurrent page retrieval and improved performance. The script also includes error handling and retry mechanisms for failed requests.

## Features

* Asynchronous retrieval of web page content.
* Handling of HTTP errors and unexpected exceptions.
* Retry mechanism with exponential backoff for initial failures.
* Configuration for maximum retry attempts and timeout.
* Utilization of logging for important information.

## Prerequisites

* Python 3.7+
* The following libraries:
    * aiohttp
    * asyncio
    * logging
    * time

## Installation

To install the dependencies, execute the following command:

```bash
pip install aiohttp


Usage
Clone this repository.
Ensure all dependencies are installed.
Run the main.py script.
python main.py


Code
import asyncio
import aiohttp
import logging
import time

# Logging configuration
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Define retry limit and timeout
MAX_RETRIES = 3
TIMEOUT = 10  # seconds

async def fetch_page(session, url, retry_count=0):
    """
    Asynchronously fetches the content of a web page.

    Args:
        session (aiohttp.ClientSession): The HTTP session to use.
        url (str): The URL of the page to fetch.
        retry_count (int, optional): The number of retrieval attempts. Defaults to 0.

    Returns:
        str: The content of the web page, or None if it fails after several attempts.
    """
    try:
        async with session.get(url, timeout=TIMEOUT) as response:
            response.raise_for_status()  # Raise an exception for HTTP error codes
            return await response.text()
    except aiohttp.ClientError as e:
        logging.error(f"HTTP client error for {url}: {e}")
        if retry_count < MAX_RETRIES:
            await asyncio.sleep(2 ** retry_count)
            logging.info(f"Retrying {url} (attempt {retry_count + 1})")
            return await fetch_page(session, url, retry_count + 1)
        else:
            logging.error(f"Failed to retrieve {url} after {MAX_RETRIES} attempts")
            return None
    except Exception as e:
        logging.error(f"Unexpected error while fetching {url}: {e}")
        return None

async def main(urls):
    """
    Main function to launch asynchronous scraping.

    Args:
        urls (list): The list of URLs to scrape.
    """
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_page(session, url) for url in urls]
        results = await asyncio.gather(*tasks)  # Execute all tasks in parallel
        return results

if __name__ == "__main__":
    # Example usage
    start_time = time.time()
    sample_urls = [
        "[https://www.google.com/maps?cid=10841271384594827821](https://www.google.com/maps?cid=10841271384594827821)",
        "[https://www.google.com/maps?cid=1519735334139957582](https://www.google.com/maps?cid=1519735334139957582)",
        "[https://www.google.com/maps?cid=17223482373573387535](https://www.google.com/maps?cid=17223482373573387535)",
    ]
    scraped_data = asyncio.run(main(sample_urls))
    end_time = time.time()

    if scraped_data:
        print(f"Retrieved data: {scraped_data}")
        print(f"Total execution time: {end_time - start_time:.2f} seconds")
    else:
        print("No data retrieved.")


Example Output
2024-07-24 10:00:00,000 - INFO - Retrieving page: [https://www.google.com/maps?cid=10841271384594827821](https://www.google.com/maps?cid=10841271384594827821)
2024-07-24 10:00:01,000 - INFO - Retrieving page:
