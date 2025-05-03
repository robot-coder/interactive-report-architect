# README.md

# Multi-Agent Interactive Reporting System

This project develops an intelligent agent system that leverages LlamaIndex to create multiple Python tools, integrates with MCP servers such as Playwright for web automation, and coordinates multiple agents to generate real-time, interactive reports. The system is modular, robust, and designed to handle complex data collection, processing, and visualization tasks.

## Features

- Utilizes LlamaIndex for efficient data indexing and retrieval
- Automates web interactions and data extraction via Playwright
- Coordinates multiple agents for complex problem solving
- Generates real-time, interactive reports with visualization tools
- Handles errors gracefully to ensure robustness

## Requirements

Ensure you have Python 3.8+ installed. Install dependencies with:

```bash
pip install -r requirements.txt
```

## Files

- `main.py`: Entry point and core logic
- `requirements.txt`: List of required libraries
- `README.md`: This documentation

## Usage

1. Install dependencies:

```bash
pip install -r requirements.txt
```

2. Run the main script:

```bash
python main.py
```

---

# main.py

import asyncio
from typing import List, Dict, Any
from llama_index import GPTIndex, SimpleIndex
from playwright.async_api import async_playwright, Browser, Page
from bs4 import BeautifulSoup
import matplotlib.pyplot as plt
import requests

# Placeholder for MCP server SDK or client import
# from mcp_server_sdk import MCPClient

class WebAutomationAgent:
    """
    Handles web automation tasks using Playwright.
    """

    def __init__(self) -> None:
        self.browser: Browser = None

    async def launch_browser(self) -> None:
        """
        Launches the Playwright browser.
        """
        try:
            self.playwright = await async_playwright().start()
            self.browser = await self.playwright.chromium.launch(headless=True)
        except Exception as e:
            print(f"Error launching browser: {e}")

    async def close_browser(self) -> None:
        """
        Closes the Playwright browser.
        """
        try:
            if self.browser:
                await self.browser.close()
            if hasattr(self, 'playwright'):
                await self.playwright.stop()
        except Exception as e:
            print(f"Error closing browser: {e}")

    async def fetch_page_content(self, url: str) -> str:
        """
        Navigates to a URL and returns the page content.
        """
        try:
            page: Page = await self.browser.new_page()
            await page.goto(url)
            content: str = await page.content()
            await page.close()
            return content
        except Exception as e:
            print(f"Error fetching page content from {url}: {e}")
            return ""

class DataExtractor:
    """
    Extracts data from HTML content.
    """

    @staticmethod
    def parse_html(content: str) -> Dict[str, Any]:
        """
        Parses HTML content and extracts relevant data.
        """
        try:
            soup = BeautifulSoup(content, 'html.parser')
            # Example: extract all text
            text = soup.get_text()
            return {"text": text}
        except Exception as e:
            print(f"Error parsing HTML content: {e}")
            return {}

class DataVisualizer:
    """
    Generates visualizations from data.
    """

    @staticmethod
    def plot_data(data: Dict[str, Any]) -> None:
        """
        Creates and saves a simple plot based on data.
        """
        try:
            # Placeholder: generate a simple bar chart
            values = [len(data.get("text", ""))]
            labels = ['Text Length']
            plt.bar(labels, values)
            plt.title('Data Summary')
            plt.savefig('report.png')
            plt.close()
        except Exception as e:
            print(f"Error generating plot: {e}")

class ReportGenerator:
    """
    Creates interactive reports based on collected data.
    """

    def __init__(self, index: SimpleIndex) -> None:
        self.index = index

    def generate_report(self, data: Dict[str, Any]) -> str:
        """
        Generates a report string.
        """
        report = f"Data Length: {len(data.get('text', ''))}\n"
        report += "Summary:\n"
        report += data.get('text', '')[:500]  # first 500 characters
        return report

async def main() -> None:
    """
    Main function orchestrating the multi-agent system.
    """
    # Initialize agents
    web_agent = WebAutomationAgent()
    await web_agent.launch_browser()

    # Example URLs to fetch data from
    urls = ["https://example.com", "https://example.org"]
    extracted_data = []

    for url in urls:
        content = await web_agent.fetch_page_content(url)
        data = DataExtractor.parse_html(content)
        extracted_data.append(data)

    await web_agent.close_browser()

    # Build or update LlamaIndex with extracted data
    try:
        index = SimpleIndex()
        for data in extracted_data:
            index.insert(data)
    except Exception as e:
        print(f"Error creating index: {e}")
        index = None

    # Generate visualizations
    for data in extracted_data:
        DataVisualizer.plot_data(data)

    # Generate reports
    if index:
        report_gen = ReportGenerator(index)
        for data in extracted_data:
            report = report_gen.generate_report(data)
            print(report)

    # Placeholder: send reports to MCP server or store them
    # mcp_client = MCPClient()
    # await mcp_client.send_report(report)

if __name__ == "__main__":
    asyncio.run(main())

# requirements.txt

llama_index
mcp_server_sdk_or_client
playwright
beautifulsoup4
matplotlib
requests