# hello-world
Test Repository for GitHub
Test Branch for hello-world repository

def create_model_dataframes(model_number):
    # Filter analytics data for the given model
    prod_df = analytics_df[(analytics_df['pm'] == model_number) & (analytics_df['purpose'].str.endswith('P'))]
    test_df = analytics_df[(analytics_df['pm'] == model_number) & (analytics_df['purpose'].str.endswith('T'))]

    # Merge PROD and Test data on cusip and pm
    analytics_comp_df = pd.merge(
        prod_df, test_df,
        on=['cusip', 'pm', 'priceA'],
        suffixes=('_prod', '_test')
    )

    # Calculate differences and round values
    analytics_comp_df['diff OAS'] = (analytics_comp_df['oasA_test'] - analytics_comp_df['oasA_prod']).round(2)
    analytics_comp_df['Diff OAD'] = (analytics_comp_df['oacA_test'] - analytics_comp_df['oacA_prod']).round(2)
    analytics_comp_df['Diff OAC'] = (analytics_comp_df['oacA_test'] - analytics_comp_df['oacA_prod']).round(2)
    analytics_comp_df['Diff CPR 3m'] = (analytics_comp_df['cpr3m_test'] - analytics_comp_df['cpr3m_prod']).round(2)
    analytics_comp_df['Diff CPR 1y'] = (analytics_comp_df['cpr1y_test'] - analytics_comp_df['cpr1y_prod']).round(2)

    # Select and rename relevant columns for Analytics_Comp
    analytics_comp_df = analytics_comp_df[[
        'cusip', 'priceA',
        'oasA_test', 'oacA_test', 'cpr3m_test', 'cpr1y_test',
        'oasA_prod', 'oacA_prod', 'cpr3m_prod', 'cpr1y_prod',
        'diff OAS', 'Diff OAD', 'Diff OAC', 'Diff CPR 3m', 'Diff CPR 1y'
    ]]
    analytics_comp_df.columns = [
        'CUSIP', 'Price',
        'Test OAS', 'Test OAD', 'Test OAC', 'Test CPR 3m', 'Test CPR 1y',
        'Prod OAS', 'Prod OAD', 'Prod OAC', 'Prod CPR 3m', 'Prod CPR 1y',
        'Diff OAS', 'Diff OAD', 'Diff OAC', 'Diff CPR 3m', 'Diff CPR 1y'
    ]

    # Merge assumptions data
    summary_df = pd.merge(
        analytics_comp_df[['CUSIP', 'Price']],
        assumptions_df,
        left_on='CUSIP', right_on='tba_pricing_cusip'
    )

    # Calculate Ratio Bal and other differences
    summary_df['Ratio Bal'] = (summary_df['CurBal_TST'] / summary_df['CurBal_PROD']).round(2)
    summary_df['Diff wac'] = (summary_df['wac_TST'] - summary_df['wac_PROD']).round(2)
    summary_df['Diff wam'] = (summary_df['wam_TST'] - summary_df['wam_PROD']).round(2)
    summary_df['Diff wala'] = (summary_df['wala_TST'] - summary_df['wala_PROD']).round(2)
    summary_df['Diff fico'] = (summary_df['fico_TST'] - summary_df['fico_PROD']).round(2)
    summary_df['Diff sato'] = (summary_df['sato_TST'] - summary_df['sato_PROD']).round(2)
    summary_df['Diff wals'] = (summary_df['wals_TST'] - summary_df['wals_PROD']).round(2)
    summary_df['Diff dti'] = (summary_df['dti_TST'] - summary_df['dti_PROD']).round(2)
    summary_df['Diff oltv'] = (summary_df['oltv_TST'] - summary_df['oltv_PROD']).round(2)

    # Select and rename relevant columns for Summary
    summary_df = summary_df[[
        'CUSIP', 'Price', 'CurBal_TST', 'CurBal_PROD', 'Ratio Bal',
        'wac_TST', 'wam_TST', 'wala_TST', 'fico_TST', 'sato_TST', 'wals_TST', 'dti_TST', 'oltv_TST',
        'oasA_test', 'oacA_test', 'cpr3m_test', 'cpr1y_test',
        'oasA_prod', 'oacA_prod', 'cpr3m_prod', 'cpr1y_prod',
        'wac_PROD', 'wam_PROD', 'wala_PROD', 'fico_PROD', 'sato_PROD', 'wals_PROD', 'dti_PROD', 'oltv_PROD',
        'Diff OAS', 'Diff OAD', 'Diff OAC', 'Diff CPR 3m', 'Diff CPR 1y',
        'Diff wac', 'Diff wam', 'Diff wala', 'Diff fico', 'Diff sato', 'Diff wals', 'Diff dti', 'Diff oltv'
    ]]
    summary_df.columns = [
        'CUSIP', 'Price', 'Test Cur Bal', 'Prod Cur Bal', 'Ratio Bal',
        'Test wac', 'Test wam', 'Test wala', 'Test fico', 'Test sato', 'Test wals', 'Test dti', 'Test oltv',
        'Test OAS', 'Test OAD', 'Test CPR 3m', 'Test CPR 1y',
        'Prod OAS', 'Prod OAD', 'Prod CPR 3m', 'Prod CPR 1y',
        'Prod wac', 'Prod wam', 'Prod wala', 'Prod fico', 'Prod sato', 'Prod wals', 'Prod dti', 'Prod oltv',
        'Diff OAS', 'Diff OAD', 'Diff OAC', 'Diff CPR 3m', 'Diff CPR 1y',
        'Diff wac', 'Diff wam', 'Diff wala', 'Diff fico', 'Diff sato', 'Diff wals', 'Diff dti', 'Diff oltv'
    ]

    return analytics_comp_df, summary_df

# Create dataframes for each model
for model in [62, 64, 99]:
    analytics_comp_df, summary_df = create_model_dataframes(model)
    print(f"Analytics_Comp_{model}:\n", analytics_comp_df)
    print(f"\nSummary_{model}:\n", summary_df)
