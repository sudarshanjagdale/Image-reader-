
import os
import zipfile
import hashlib

classification_folder = 'classification'  # path to your folder
png_hash_map = {}

def hash_file_content(file_data):
    hasher = hashlib.md5()
    hasher.update(file_data)
    return hasher.hexdigest()

def check_png_in_zip(zip_path):
    with zipfile.ZipFile(zip_path, 'r') as zf:
        for name in zf.namelist():
            if name.lower().endswith('.png') and not name.endswith('/'):
                with zf.open(name) as file:
                    file_data = file.read()
                    file_hash = hash_file_content(file_data)
                    if file_hash in png_hash_map:
                        png_hash_map[file_hash].append((zip_path, name))
                    else:
                        png_hash_map[file_hash] = [(zip_path, name)]

# Traverse through classification folder
for root, dirs, files in os.walk(classification_folder):
    for file in files:
        if file.endswith('.zip'):
            zip_file_path = os.path.join(root, file)
            check_png_in_zip(zip_file_path)

# Output duplicates
print("\n🔁 Duplicate PNG files (based on content):")
for file_hash, locations in png_hash_map.items():
    if len(locations) > 1:
        print(f"\nHash: {file_hash}")
        for zip_path, file_path in locations:
            print(f" - {zip_path} -> {file_path}")
