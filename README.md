pip install PyPDF2


import PyPDF2
import csv
import requests
from io import BytesIO

# URL of the PDF
pdf_url = "https://visionias.in/resources/material/?id=1021&type=ca"

def download_pdf(url):
    try:
        response = requests.get(url)
        response.raise_for_status()  # Check for request errors
        return BytesIO(response.content)
    except requests.RequestException as e:
        print(f"Error downloading PDF: {e}")
        return None

def extract_text_from_pdf(pdf_file):
    text = ""
    try:
        reader = PyPDF2.PdfFileReader(pdf_file)
        for page_num in range(reader.numPages):
            page = reader.getPage(page_num)
            text += page.extract_text() + "\n"
    except Exception as e:
        print(f"Error extracting text from PDF: {e}")
    return text

def process_text_to_articles(text):
    # This function needs to be customized based on the structure of your PDF
    # Here, we'll just create dummy data to illustrate the process
    articles = []
    lines = text.split("\n")
    s_no = 1.1
    for line in lines:
        if line.strip():  # Skip empty lines
            title = "Sample Title"  # Placeholder, extract the real title from line
            body = line  # Placeholder, extract the real body from line
            articles.append((s_no, title, body))
            s_no += 0.1  # Increment serial number for demonstration
    return articles

def write_articles_to_csv(articles, filename):
    try:
        with open(filename, mode='w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file, quoting=csv.QUOTE_MINIMAL)
            writer.writerow(['s_no', 'article_title', 'article_body'])
            for article in articles:
                writer.writerow(article)
    except IOError as e:
        print(f"Error writing to CSV: {e}")

def main():
    pdf_file = download_pdf(pdf_url)
    if pdf_file:
        text = extract_text_from_pdf(pdf_file)
        articles = process_text_to_articles(text)
        write_articles_to_csv(articles, 'articles.csv')
        print("CSV file created successfully.")

if __name__ == "__main__":
    main()


pip install pdfminer.six

import csv
import requests
from io import BytesIO
from pdfminer.high_level import extract_text
from pdfminer.pdfparser import PDFSyntaxError

# URL of the PDF
pdf_url = "https://visionias.in/resources/material/?id=10"

def download_pdf(url):
    try:
        response = requests.get(url)
        response.raise_for_status()  # Check for request errors
        return BytesIO(response.content)
    except requests.RequestException as e:
        print(f"Error downloading PDF: {e}")
        return None

def extract_text_from_pdf(pdf_file):
    try:
        # Extract text using pdfminer
        text = extract_text(pdf_file)
        return text
    except PDFSyntaxError as e:
        print(f"PDF syntax error: {e}")
        return ""
    except Exception as e:
        print(f"Error extracting text from PDF: {e}")
        return ""

def process_text_to_articles(text):
    # This function needs to be customized based on the structure of your PDF
    # Here, we'll create dummy data to illustrate the process
    articles = []
    lines = text.split('\n')
    s_no = 1.1
    for line in lines:
        if line.strip():  # Skip empty lines
            title = "Sample Title"  # Placeholder, extract the real title from line
            body = line  # Placeholder, extract the real body from line
            articles.append((s_no, title, body))
            s_no += 0.1  # Increment serial number for demonstration
    return articles

def write_articles_to_csv(articles, filename):
    try:
        with open(filename, mode='w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file, quoting=csv.QUOTE_MINIMAL)
            writer.writerow(['s_no', 'article_title', 'article_body'])
            for article in articles:
                writer.writerow(article)
    except IOError as e:
        print(f"Error writing to CSV: {e}")

def main():
    pdf_file = download_pdf(pdf_url)
    if pdf_file:
        text = extract_text_from_pdf(pdf_file)
        articles = process_text_to_articles(text)
        write_articles_to_csv(articles, 'articles.csv')
        print("CSV file created successfully.")

if __name__ == "__main__":
    main()
