import pandas as pd

# Load your CSV
df = pd.read_csv('pages.csv')

# Clean column names if needed
df.columns = df.columns.str.strip()

# Output list
results = []

# Group by hash to find similar pages
for hash_value, group in df.groupby('Hash'):
    doc_ids = group['Doc_ID'].unique()
    
    # Only consider hashes with more than 1 document ID (i.e., duplicates across docs)
    if len(doc_ids) > 1:
        for doc_id in doc_ids:
            current_doc_pages = group[group['Doc_ID'] == doc_id]
            others = group[group['Doc_ID'] != doc_id]
            other_doc_ids = others['Doc_ID'].unique()
            
            for other_doc_id in other_doc_ids:
                matching_pages = others[others['Doc_ID'] == other_doc_id]['Page_No'].tolist()
                results.append({
                    'Doc ID': doc_id,
                    'Similar Doc ID': other_doc_id,
                    'Page No Matched': ','.join(map(str, sorted(set(matching_pages))))
                })

# Create final DataFrame and drop duplicate pairs (e.g., keep only one of 126-262 or 262-126)
result_df = pd.DataFrame(results)

# To avoid duplicate pairs (optional)
result_df['Pair'] = result_df.apply(lambda row: '-'.join(sorted([str(row['Doc ID']), str(row['Similar Doc ID'])])), axis=1)
result_df = result_df.drop_duplicates('Pair').drop(columns='Pair')

# Save result
result_df.to_csv('similar_doc_matches.csv', index=False)

print("✅ Done! Output saved to 'similar_doc_matches.csv'")