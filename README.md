import os
import zipfile
from PIL import Image
import pytesseract

# Set your folder path where zip files are located
zip_folder_path = "path/to/your/folder"
search_text = "company name"

# Loop through each file in the folder
for zip_file_name in os.listdir(zip_folder_path):
    if zip_file_name.endswith(".zip"):
        zip_path = os.path.join(zip_folder_path, zip_file_name)
        
        # Extract zip file
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            extract_path = os.path.join(zip_folder_path, "temp_extracted")
            zip_ref.extractall(extract_path)

            # Loop through extracted files/folders
            for root, dirs, files in os.walk(extract_path):
                for file in files:
                    if file.endswith(".png"):
                        image_path = os.path.join(root, file)
                        try:
                            text = pytesseract.image_to_string(Image.open(image_path))
                            if search_text.lower() in text.lower():
                                print(f"✅ Found in: {image_path}")
                        except Exception as e:
                            print(f"❌ Error reading {image_path}: {e}")

        # Clean up temp folder (optional)
        import shutil
        shutil.rmtree(extract_path)