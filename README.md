import requests
import os

# List of PDF URLs (replace with your actual links)
pdf_urls = [
    "https://example.com/file1.pdf",
    "https://example.com/file2.pdf",
    "https://example.com/file3.pdf"
]

# Folder to save the downloaded PDFs
output_folder = "downloaded_pdfs"
os.makedirs(output_folder, exist_ok=True)

for i, url in enumerate(pdf_urls, start=1):
    try:
        response = requests.get(url)
        response.raise_for_status()  # Check if the request was successful

        # Extract filename from URL or use default
        filename = url.split("/")[-1]
        if not filename.endswith(".pdf"):
            filename = f"file_{i}.pdf"

        file_path = os.path.join(output_folder, filename)

        # Write content to file
        with open(file_path, "wb") as f:
            f.write(response.content)

        print(f"✅ Downloaded: {filename}")

    except requests.exceptions.RequestException as e:
        print(f"❌ Failed to download from {url}: {e}")