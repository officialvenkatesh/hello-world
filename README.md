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
