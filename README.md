import requests
from bs4 import BeautifulSoup
import sqlite3
from datetime import datetime

# Define constants
BASE_URL = "https://indianexpress.com/"
BUSINESS_PATH = "/business/"
DB_NAME = "articles.db"

# Initialize SQLite database
def init_db():
    conn = sqlite3.connect(DB_NAME)
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS articles (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            author TEXT,
            publication_date TEXT,
            content TEXT NOT NULL
        )
    ''')
    conn.commit()
    return conn

# Fetch HTML content of a URL
def fetch_url(url):
    response = requests.get(url)
    response.raise_for_status()
    return response.text

# Extract article links from the homepage
def extract_article_links(homepage_html):
    soup = BeautifulSoup(homepage_html, 'html.parser')
    article_links = []
    
    for a_tag in soup.find_all('a', href=True):
        href = a_tag['href']
        if BUSINESS_PATH in href:
            # Ensure we have the full URL
            if not href.startswith('http'):
                href = BASE_URL + href
            article_links.append(href)
    
    return list(set(article_links))  # Remove duplicates

# Extract article details
def extract_article_details(article_html):
    soup = BeautifulSoup(article_html, 'html.parser')
    
    # Title
    title_tag = soup.find('h1', class_='article-title')
    title = title_tag.get_text(strip=True) if title_tag else 'No Title'
    
    # Author
    author_tag = soup.find('span', class_='author')
    author = author_tag.get_text(strip=True) if author_tag else 'No Author'
    
    # Publication date
    date_tag = soup.find('time', class_='article-date')
    publication_date = date_tag.get_text(strip=True) if date_tag else 'No Date'
    
    # Content
    content_tag = soup.find('div', class_='article-content')
    content = content_tag.get_text(strip=True) if content_tag else 'No Content'
    
    return {
        'title': title,
        'author': author,
        'publication_date': publication_date,
        'content': content
    }

# Save article details to the database
def save_article_to_db(conn, article_details):
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO articles (title, author, publication_date, content)
        VALUES (?, ?, ?, ?)
    ''', (article_details['title'], article_details['author'], article_details['publication_date'], article_details['content']))
    conn.commit()

def main():
    conn = init_db()
    
    # Fetch homepage
    homepage_html = fetch_url(BASE_URL)
    
    # Extract article links
    article_links = extract_article_links(homepage_html)
    
    for link in article_links:
        print(f"Processing: {link}")
        try:
            # Fetch article page
            article_html = fetch_url(link)
            
            # Extract details
            article_details = extract_article_details(article_html)
            
            # Save to database
            save_article_to_db(conn, article_details)
        
        except Exception as e:
            print(f"Failed to process {link}: {e}")
    
    conn.close()
    print("Scraping completed!")

if __name__ == "__main__":
    main()


    import sqlite3

# Function to initialize the SQLite database and create the schema
def create_database_schema(db_name):
    """
    Creates a SQLite database with the specified schema.
    
    Args:
    db_name (str): The name of the SQLite database file.
    """
    try:
        # Connect to the SQLite database (or create it if it doesn't exist)
        conn = sqlite3.connect(db_name)
        cursor = conn.cursor()
        
        # Create the articles table if it doesn't already exist
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS articles (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL,
                author TEXT,
                publication_date TEXT,
                content TEXT NOT NULL
            )
        ''')
        
        # Commit the changes and close the connection
        conn.commit()
        print(f"Database schema created successfully in '{db_name}'.")
        
    except sqlite3.Error as e:
        print(f"An error occurred while creating the database schema: {e}")
    
    finally:
        if conn:
            conn.close()

# Usage example
if __name__ == "__main__":
    create_database_schema("articles.db")

    
