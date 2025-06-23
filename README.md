import os
import zipfile
import imagehash
from PIL import Image
from io import BytesIO
from itertools import combinations
import pandas as pd
from collections import defaultdict

# Step 1: Read PNGs from ZIP without extracting
def get_document_pngs_from_zip(zip_path):
    doc_images = {}
    with zipfile.ZipFile(zip_path, 'r') as zip_ref:
        for file in zip_ref.namelist():
            if 'document_no152' in file and file.endswith('.png'):
                parts = file.split('/')
                doc_id = None
                for part in parts:
                    if part.startswith('document_no152'):
                        doc_id = part.replace('document_no', '')
                        break
                if doc_id:
                    doc_images.setdefault(doc_id, []).append((file, zip_path))
    return doc_images

# Step 2: Compare pages and track duplicates
def group_similar_docs(all_docs):
    similar_docs = defaultdict(lambda: {'similar_to': set(), 'duplicate_pages': set()})
    for (doc1, files1), (doc2, files2) in combinations(all_docs.items(), 2):
        matched_pages = set()
        for f1_path, f1_zip in files1:
            for f2_path, f2_zip in files2:
                try:
                    with zipfile.ZipFile(f1_zip, 'r') as z1, zipfile.ZipFile(f2_zip, 'r') as z2:
                        img1 = Image.open(BytesIO(z1.read(f1_path)))
                        img2 = Image.open(BytesIO(z2.read(f2_path)))
                        hash1 = imagehash.average_hash(img1)
                        hash2 = imagehash.average_hash(img2)
                        if hash1 - hash2 <= 2:
                            matched_pages.add(os.path.basename(f1_path))
                except Exception as e:
                    print(f"Error comparing {f1_path} and {f2_path}: {e}")
        if matched_pages:
            # Save both directions (doc1 ↔ doc2)
            similar_docs[doc1]['similar_to'].add(doc2)
            similar_docs[doc1]['duplicate_pages'].update(matched_pages)
            similar_docs[doc2]['similar_to'].add(doc1)
            similar_docs[doc2]['duplicate_pages'].update(matched_pages)
    return similar_docs

# Step 3: Output groups in main → similar list format
def create_grouped_output(similar_docs):
    output = []
    visited = set()

    for doc_id in similar_docs:
        if doc_id in visited:
            continue
        group = set()
        pages = set()

        def dfs(d):
            if d in visited:
                return
            visited.add(d)
            group.add(d)
            pages.update(similar_docs[d]['duplicate_pages'])
            for s in similar_docs[d]['similar_to']:
                dfs(s)

        dfs(doc_id)
        if len(group) > 1:
            main_doc = min(group)
            group.discard(main_doc)
            output.append([main_doc, ', '.join(sorted(group)), ', '.join(sorted(pages))])
    return output

# Step 4: Main script
def main():
    zip_folder = r"C:\Users\sudarshan\Desktop\YourMainFolder"  # 🔁 Change this path
    all_docs_combined = {}

    for zip_file in os.listdir(zip_folder):
        if zip_file.endswith(".zip"):
            zip_path = os.path.join(zip_folder, zip_file)
            docs = get_document_pngs_from_zip(zip_path)
            for doc_id, files in docs.items():
                all_docs_combined.setdefault(doc_id, []).extend(files)

    similar_docs = group_similar_docs(all_docs_combined)
    grouped_output = create_grouped_output(similar_docs)

    df = pd.DataFrame(grouped_output, columns=["Main Document", "Similar Documents", "Duplicate Pages"])
    df.to_csv("grouped_duplicates_output.csv", index=False)
    print("✅ Grouped output saved to 'grouped_duplicates_output.csv'")

# Run
if __name__ == "__main__":
    main()