import pandas as pd

# Input and output files
input_file = "hyperscience_data.xlsx"
output_file = "processed_data.xlsx"

# Read Excel
df = pd.read_excel(input_file)

# Split combined column into workbench_sub_id and imp_id if needed
if "subworkbench_ipm" in df.columns:
    df[['workbench_sub_id', 'imp_id']] = df['subworkbench_ipm'].str.split('_', expand=True)

# Create TP/TN flag
df['TP_TN'] = df['confidence'].apply(lambda x: "TP" if pd.notnull(x) else "TN")

# Now pivot per layout
with pd.ExcelWriter(output_file, engine="openpyxl") as writer:
    for layout in df['layout_name'].unique():
        layout_df = df[df['layout_name'] == layout]

        # Create a nice column name that combines doc_id and sub_id
        layout_df['doc_sub'] = layout_df['doc_id'].astype(str) + " (" + layout_df['sub_id'].astype(str) + ")"

        # Pivot: FieldName in rows, Doc/Sub in columns, values = TP/TN
        pivot_df = layout_df.pivot_table(
            index="field_name",
            columns="doc_sub",
            values="TP_TN",
            aggfunc="first"
        )

        # Save per layout
        pivot_df.to_excel(writer, sheet_name=layout[:30])

print(f"Processed file saved as {output_file}")