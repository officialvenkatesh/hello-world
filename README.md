# hello-world
Test Repository for GitHub
Test Branch for hello-world repository

from openpyxl import load_workbook
from openpyxl.styles import PatternFill, Alignment, Font
from openpyxl.utils import get_column_letter

# List of Excel files to be combined
sheets = [
    '64_summary_df.xlsx', '64_analytics_comp_df.xlsx',
    '62_summary_df.xlsx', '62_analytics_comp_df.xlsx',
    '99_summary_df.xlsx', '99_analytics_comp_df.xlsx'
]

# Create a new Excel writer object
with pd.ExcelWriter('impact_analysis.xlsx', engine='openpyxl') as writer:
    for sheet in sheets:
        # Read each Excel file
        df = pd.read_excel(sheet)
        
        # Write each DataFrame to a separate sheet in the master file
        sheet_name = sheet.split('_')[0] + '_' + sheet.split('_')[1]
        df.to_excel(writer, sheet_name=sheet_name, index=False)

    # Save the writer
    writer.save()

# Load the workbook to apply formatting
wb = load_workbook('impact_analysis.xlsx')

# Define colors and styles
black_fill = PatternFill(start_color='000000', end_color='000000', fill_type='solid')
white_font = Font(color='FFFFFF')
light_purple_fill = PatternFill(start_color='E6E6FA', end_color='E6E6FA', fill_type='solid')
light_tan_fill = PatternFill(start_color='D2B48C', end_color='D2B48C', fill_type='solid')
center_alignment = Alignment(horizontal='center')

for sheet_name in wb.sheetnames:
    ws = wb[sheet_name]
    
    # Remove gridlines
    ws.sheet_view.showGridLines = False
    
    # Set width of all columns to 9
    for col in range(1, ws.max_column + 1):
        ws.column_dimensions[get_column_letter(col)].width = 9
    
    # Center align all cells
    for row in ws.iter_rows(min_row=1, max_row=ws.max_row, min_col=1, max_col=ws.max_column):
        for cell in row:
            cell.alignment = center_alignment
    
    # Blank out column A
    for row in ws.iter_rows(min_row=1, max_row=ws.max_row, min_col=1, max_col=1):
        for cell in row:
            cell.value = None

    # Specific formatting for summary sheets
    if 'summary' in sheet_name:
        # Header formatting
        for cell in ws['B1':'AV1'][0]:
            cell.fill = black_fill
            cell.font = white_font
        
        # Set specific column widths
        for col in range(5, 10):  # Columns E to I
            ws.column_dimensions[get_column_letter(col)].width = 14
        
        # Apply light purple background
        for col in range(10, 17):  # Columns J to Q
            for cell in ws[get_column_letter(col)]:
                cell.fill = light_purple_fill
        for col in range(23, 31):  # Columns W to AD
            for cell in ws[get_column_letter(col)]:
                cell.fill = light_purple_fill
        for col in range(36, 44):  # Columns AJ to AQ
            for cell in ws[get_column_letter(col)]:
                cell.fill = light_purple_fill
        
        # Format negative numbers in AJ to AV
        for row in ws.iter_rows(min_row=2, max_row=ws.max_row, min_col=36, max_col=48):
            for cell in row:
                if isinstance(cell.value, (int, float)) and cell.value < 0:
                    cell.font = Font(color='FF0000')
                    cell.number_format = '0.00;[Red](0.00)'

    # Specific formatting for analytics comp sheets
    if 'analytics_comp' in sheet_name:
        # Header formatting
        for cell in ws['B1':'U1'][0]:
            cell.fill = black_fill
            cell.font = white_font
        
        # Set specific column widths
        ws.column_dimensions['E'].width = 14  # Column E
        
        # Apply light tan background
        for col in range(17, 22):  # Columns Q to U
            for cell in ws[get_column_letter(col)]:
                cell.fill = light_tan_fill
        
        # Format negative numbers in Q to U
        for row in ws.iter_rows(min_row=2, max_row=ws.max_row, min_col=17, max_col=22):
            for cell in row:
                if isinstance(cell.value, (int, float)) and cell.value < 0:
                    cell.font = Font(color='FF0000')
                    cell.number_format = '0.00;[Red](0.00)'

# Save the formatted workbook
wb.save('impact_analysis.xlsx')

# Create dataframes for each model
for model in [62, 64, 99]:
    analytics_comp_df, summary_df = create_model_dataframes(model)
    print(f"Analytics_Comp_{model}:\n", analytics_comp_df)
    print(f"\nSummary_{model}:\n", summary_df)




NON-NEGOTIABLE STYLE RULES
1) Global colors:
   - Background: near-black (#0B0B0B or #0A0A0A)
   - Panels/containers: slightly lighter (#121212 / #151515)
   - Text: off-white (#E6E6E6)
   - Muted text: gray (#9AA0A6)
   - Accent: Bloomberg-like orange (#FF9900 or #FF8C00)
   - Borders/dividers: dark gray (#2A2A2A)
2) Typography:
   - Use a monospace font for all UI (e.g., "IBM Plex Mono", "JetBrains Mono", or "Courier New").
   - Slightly smaller font sizes than default; compact spacing.
3) Layout:
   - Dense dashboard look: use st.columns for grids, use bordered sections.
   - Replace big whitespace with tighter margins/padding.
4) Components:
   - Add a top header bar with app title on the left and small status text on right.
   - Put KPIs in small “terminal tiles” (numbers in orange).
   - Use sections with titles like “MARKET”, “RISK”, “CHARTS” in uppercase.
5) Charts:
   - Keep chart background black, gridlines subtle gray, axis labels off-white, series default to orange (or orange + a couple muted colors).
   - Avoid bright blue default chart themes.

IMPLEMENTATION REQUIREMENTS (STREAMLIT)
- Implement theme via:
  A) Create/modify .streamlit/config.toml to set base="dark" and primaryColor=accent orange
  AND
  B) Inject custom CSS using st.markdown(..., unsafe_allow_html=True) for:
     * background colors
     * sidebar styling
     * monospace font
     * buttons, sliders, selectbox accents
     * “panel” class for containers
- Do NOT change business logic. Only adjust styling/layout.
- Keep changes minimal, clean, and maintainable.
- Provide the final code changes as:
  1) Updated config.toml contents
  2) A Python snippet for CSS injection and reusable panel helper
  3) Any chart theme changes (matplotlib/plotly) in-place

DELIVERABLES
- Add a function `inject_terminal_theme()` that applies CSS globally.
- Add a helper `panel(title: str)` / context manager style to wrap sections in a styled container.
- Update charts to use black backgrounds and orange accents.


IMPORTANT: Use exact hex codes listed above. Use uppercase section headers. Ensure sidebar also matches dark theme. Ensure widget focus/hover states use orange. Verify readability: no pure white backgrounds anywhere.
