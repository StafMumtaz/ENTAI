# Progress Log

        Here is the link to the google sheets page where I’m reading abstracts and rating them. In column H, I place a 1 along papers that I can see myself reading all the way through, along with comments. https://docs.google.com/spreadsheets/d/1vNf1ti7D969UbPPMl8MrlMtOFUSxt1lP6hmMga4kZ_k/edit?usp=sharing

> This is a paper on *natural language processing* in *otolaryngology* broadly, seems to have the right amount of papers in the field
> 

---

1. **Paper selection**

I first chose the papers I would use by giving some input parameters to PubMed. I used the following and received 177 results: 

```bash
("natural language processing" OR "NLP" OR "text mining" OR "language model") AND ("rhinology" OR "sinus" OR "sinusitis" OR "nasal" OR "rhinosinusitis" OR "paranasal" OR "nose" OR "CRS" OR "ENT" OR "otolaryngology" OR "ear nose throat" OR "laryngology" OR "otology")
```

I then distilled this down to about ~60 papers that I could read, 20 of which I would read quite closely. To automate this process, I used AI to read through the title and abstract of all papers to determine which are immediately irrelevant to my study: 

---

You can just use shortcut **ENT** to navigate to folder but otherwise: 

```bash
cd ~/Documents/Mustafa/7_Research/2-ENTai/5_Project1
```

- *Set environment variable to ENT to navigate there quickly*
    
    ```bash
    alias ENT='cd ~/Documents/Mustafa/7_Research/2-ENTai/5_Project1'
    source ~/.zshrc    # or ~/.bashrc, depending on your shell
    ```
    

`2_PubMed_Processor.py` will first visualize the csv file I’ve taken from Zotero. 

- $Code$
    
    ```bash
    #Mustafa Mumtaz
    #ENT Project 1
    #Natural Language Processing in Otolaryngology
    # Import the pandas library for working with data
    import pandas as pd
    
    # Set the filename of your CSV export from Zotero
    input_filename = "1_PubMed_Sources.csv"
    
    # Read the CSV file into a pandas DataFrame
    df = pd.read_csv(input_filename)
    
    # Print information about the DataFrame
    print("DataFrame shape (rows, columns):", df.shape)
    print(f"Original DataFrame has {len(df.columns)} columns")
    
    print("\nColumn names:")
    for i, column in enumerate(df.columns):
        print(f"{i}: {column}")
    
    # Display the first 5 rows to see what the data looks like
    print("\nFirst 5 rows:")
    print(df.head(5))
    
    # Create a duplicate DataFrame with only the first 40 columns
    df_modified = df.iloc[:, :40]
    
    # Verify the column reduction worked
    print(f"\nModified DataFrame has {len(df_modified.columns)} columns")
    
    # Print the remaining column names to verify
    print("\nRemaining columns in modified DataFrame:")
    for i, column in enumerate(df_modified.columns):
        print(f"{i}: {column}")
    
    #Create function to save df_modified to a csv and html and then execute it
    def save_dataframe(df, base_filename="3_PubMed_Sources_Modified"):
        """Save the dataframe to both CSV and HTML files"""
        csv_filename = f"{base_filename}.csv"
        html_filename = f"{base_filename}.html"
        
        df.to_csv(csv_filename, index=False)
        df.to_html(html_filename, index=False)
        
        print(f"\nDataFrame saved to {csv_filename} and {html_filename}")
    save_dataframe(df_modified)
    print("\nFiles saved successfully!")
    
    #Remove superfluous columns
    cols_to_remove = [
        1, 6, 9, 14, 16, 19, 
        *range(21, 28),  # 21 to 27
        *range(30, 35), # 30 to 34
        *range(36, 39)  # 36 to 38
    ]
    
    df_modified.drop(df_modified.columns[cols_to_remove], axis=1, inplace=True)
    
    # Print info about removed columns
    print(f"Removed {len(cols_to_remove)} columns: {cols_to_remove}")
    print(f"Modified DataFrame now has {len(df_modified.columns)} columns")
    
    # Print the remaining column names to verify
    print("\nRemaining columns in modified DataFrame:")
    for i, column in enumerate(df_modified.columns):
        print(f"{i}: {column}")
    
    # Save the updated DataFrame 
    save_dataframe(df_modified)
    
    #Create dataframe for Claude API interaction and give info about it 
    df_claude_input = df_modified.iloc[:, [1, 3, 4, 7]]
    print(f"Claude input DataFrame has {len(df_claude_input.columns)} columns")
    
    # Print the column names to verify
    print("\nColumns in Claude input DataFrame:")
    for i, column in enumerate(df_claude_input.columns):
        print(f"{i}: {column}")
    
    # Display the first few rows to verify content
    print("\nFirst 3 rows of Claude input DataFrame:")
    print(df_claude_input.head(3))
    
    # Save the Claude input DataFrame to CSV
    df_claude_input.to_csv("4_ClaudeInput.csv", index=False)
    print("\nSaved Claude input DataFrame to 4_ClaudeInput.csv")
    
    ```
    
- It creates a copy of the Zotero output, and then distills it down to what will be fed into the Claude API, a file named `4_Claude_Input.csv`
- This includes Date of Publication, Title, Publication Title and Abstract

---

`5_ClaudeInterface.py` will access the API and append a column indicating inclusion or exclusion. 

> **Prompt to Claude**
> 
> 
> ```
> I'm conducting a systematic review specifically focused on natural language processing and language models in the field of otolaryngology (ENT).
> 
> Please review the following paper and determine if it meets BOTH of these criteria:
> 1. It must focus specifically on natural language processing, NLP, or language models (not just general AI, machine learning, or other AI applications). Of note, as long as it does primarily utilize a language based AI model, I am fine with including a very large range of applications.
> 2. It must be directly relevant to otolaryngology/ENT (ear, nose, throat) practice or research. If the article primarily seems to be about other fields, it should not be included.
> 
> Title: {title}
> Abstract: {abstract}
> 
> Based solely on the title and abstract, should this paper be included in my systematic review?
> Respond with ONLY "1" if it meets BOTH criteria, or "0" if it fails to meet either criterion.
> ```
> 
- $Code$
    
    ```bash
    #Mustafa Mumtaz
    #ENT Project 1
    #Natural Language Processing in Otolaryngology
    #Script 2: Start Processing CSV with Claude
    
    # Add these imports at the top of your file
    import pandas as pd
    import time
    import os.path
    from anthropic import Anthropic
    
    #---------- CLAUDE API INTEGRATION ----------#
    print("\n\nStarting Claude API integration...")
    
    #Set up Claude API Client
    client = Anthropic(api_key="redacted")
    
    # Create a function to determine if a paper should be included
    def should_include_paper(title, abstract):
        # Construct the prompt for Claude
        prompt = f"""
        I'm conducting a systematic review specifically focused on natural language processing and language models in the field of otolaryngology (ENT).
    
        Please review the following paper and determine if it meets BOTH of these criteria:
        1. It must focus specifically on natural language processing, NLP, or language models (not just general AI, machine learning, or other AI applications). Of note, as long as it does primarily utilize a language based AI model, I am fine with including a very large range of applications. 
        2. It must be directly relevant to otolaryngology/ENT (ear, nose, throat) practice or research. If the article primarily seems to be about other fields, it should not be included. 
        
        Title: {title}
        Abstract: {abstract}
    
        Based solely on the title and abstract, should this paper be included in my systematic review?
        Respond with ONLY "1" if it meets BOTH criteria, or "0" if it fails to meet either criterion.
        """
        
        # Call the Claude API
        try:
            response = client.messages.create(
                model="claude-3-7-sonnet-20250219",
                max_tokens=5,
                temperature=0,
                system="You are helping a medical student conduct a systematic review of natural language processing in otolaryngology. Respond only with '1' for papers that meet ALL inclusion criteria or '0' for papers that fail to meet any criterion.",
                messages=[
                    {"role": "user", "content": prompt}
                ]
            )
            
            # Extract the response
            answer = response.content[0].text.strip()
            
            # Convert to binary (default to 0 if anything unexpected)
            return 1 if answer == "1" else 0
        except Exception as e:
            print(f"Error with API call: {e}")
            return 0
    
    # Read the Claude input CSV again to ensure we have the latest data
    df_claude_input = pd.read_csv("4_ClaudeInput.csv")
    
    # Add a new column for inclusion decision if it doesn't exist
    if 'include' not in df_claude_input.columns:
        df_claude_input['include'] = 0
    
    # Check if there's a checkpoint file to resume from
    checkpoint_file = "5_ClaudeOutput_checkpoint.csv"
    start_index = 0
    
    if os.path.exists(checkpoint_file):
        print(f"Found checkpoint file. Resuming from previous run...")
        df_checkpoint = pd.read_csv(checkpoint_file)
        # Find the last processed row
        processed_rows = df_checkpoint['include'].notna().sum()
        if processed_rows > 0:
            # Copy already processed decisions
            df_claude_input.iloc[:processed_rows, -1] = df_checkpoint.iloc[:processed_rows, -1]
            start_index = processed_rows
            print(f"Resuming from paper {start_index + 1}")
    
    # Get column names for safer referencing
    columns = df_claude_input.columns.tolist()
    title_col = columns[1]  # Title is the 2nd column
    abstract_col = columns[3]  # Abstract is the 4th column
    
    # Process each row
    print("Processing papers with Claude API...")
    for i in range(start_index, len(df_claude_input)):
        row = df_claude_input.iloc[i]
        title = row[title_col]
        abstract = row[abstract_col]
        
        # Handle missing abstracts
        if pd.isna(abstract):
            abstract = "No abstract available."
        
        # Print progress
        print(f"Processing paper {i+1}/{len(df_claude_input)}: {title[:50]}...")
        
        # Get Claude's decision
        include = should_include_paper(title, abstract)
        
        # Update the DataFrame
        df_claude_input.at[i, 'include'] = include
        
        # Save checkpoint after each paper
        df_claude_input.to_csv(checkpoint_file, index=False)
        
        # Add a small delay to avoid hitting API rate limits
        time.sleep(0.5)
    
    # Save the final updated DataFrame
    df_claude_input.to_csv("6_ClaudeOutput.csv", index=False)
    print("\nProcessing complete! Results saved to 6_ClaudeOutput.csv")
    
    # Print a summary of results
    included_count = df_claude_input['include'].sum()
    print(f"\nSummary:")
    print(f"Total papers: {len(df_claude_input)}")
    print(f"Papers to include: {included_count}")
    print(f"Papers to exclude: {len(df_claude_input) - included_count}")
    ```
    

> **Output**
> 
> 
> Total papers: 177
> 
> Papers to exclude: 109
> 
> Papers to include: 68
> 

<aside>
<img src="https://www.notion.so/icons/cellular_red.svg" alt="https://www.notion.so/icons/cellular_red.svg" width="40px" />

**TBD**: I will return to this to create a statistical test to make sure the model was accurate in its inclusion and exclusion. 

</aside>

---

I then went manually and read every abstract, rating relevance of the paper to my study on a subjective scale out of 10. My criteria was as follows: 

1. Journal Quality
2. Taste
    - Degree to which it aligns with the following questions:
        1. Is this a large review of natural language processing artificial intelligence being used in ENT, in that does this paper have information that overlaps most with the kind of information I am trying to ascertain?
        2. Is there a unique use of NLP in this paper, that seems useful and interesting?
        3. Is this paper the best quality representation of the specific use of NLP that I am interested in? 
        4. Is this paper unlike other papers I have already used in the case that I have enough of a certain kind of paper? 
3. Date published 

---

**2. Verifying Paper Selection**

I want to verify that paper selection was valid. I will use a Bernoulli trial Binomial Test to do so, blinding myself to the papers in question and seeing if my judgement aligns with the AI using script `8_OutputVerification.py`

- $Sample \space of\space Output\space from \space \space Code$
    
    ```bash
    Total papers: 177
    Included by AI: 68 (38.4%)
    Excluded by AI: 109 (61.6%)
    
    Validation sample:
      Papers AI included: 7
      Papers AI excluded: 13
      Total sample size: 20
    
    Files created successfully!
    1. Review '9_BlindedAbstracts.csv' and mark your decisions (1=include, 0=exclude)
    2. After review, run the analysis script to compare with AI decisions
    
    Next steps:
    1. Open '9_BlindedAbstracts.csv' in Excel or your preferred editor
    2. For each paper, enter 1 (include) or 0 (exclude) in the 'your_decision' column
    3. Save the file
    4. Run '11_AnalyzeValidation.py' to see the validation results
    
    Statistical power notes:
    With n=20 and expected 95% accuracy:
    - Can detect accuracy ≥ 80% with >99% power
    - 95% CI margin of error: ±10-15%
    - Minimum detectable agreement for κ > 0.6: ~85%
    (base) mustafamumtaz@MacBook-Pro-8 5_Project1 % 
    ```
    
    ```bash
    Results:
    Total papers reviewed: 20
    Agreements: 20
    Disagreements: 0
    Raw accuracy: 100.0%
    
    Binomial test (H0: accuracy ≤ 80%):
    P-value: 0.0115
    95% Confidence interval: [88.1%, 100.0%]
    
    Cohen's Kappa: 1.000
    
    Confusion Matrix:
                     AI Include  AI Exclude
    You Include          7          0     
    You Exclude          0          13    
    
    Sensitivity (AI detecting your inclusions): 100.0%
    Specificity (AI detecting your exclusions): 100.0%
    
    Perfect agreement! No disagreements to review.
    ```
    

Result: The AI matched my own subjective judgement for validity for 20/20 of the papers in question, confirming perfect alignment. 

- Output from Test
    
    ```bash
    Multi-Criteria Paper Evaluation using OpenAI GPT-4.1-mini
    With automatic retry logic for rate limits
    ============================================================
    ✓ Loaded 3136 papers from 4_APIinput.csv
    ✓ Testing first 5 papers
    
    Processing paper 1/5
    Title: How To Use Zotero (A Complete Beginner's Guide)...
      Evaluating nlp_ent_relevant... [0]
      Evaluating real_world_app... [0]
      Evaluating substantive_study... [0]
      Evaluating research_tool... [0]
      Evaluating data_insights... [0]
      Evaluating clinical_decision... [0]
      Evaluating emr_integration... [0]
    
    Processing paper 2/5
    Title: Innovations in Otolaryngology Using LLM for Early Detection ...
      Evaluating nlp_ent_relevant... [0]
      Evaluating real_world_app... [1]
      Evaluating substantive_study... [1]
      Evaluating research_tool... [0]
      Evaluating data_insights... [1]
      Evaluating clinical_decision... [1]
      Evaluating emr_integration... [0]
    
    Processing paper 3/5
    Title: Comparison of ChatGPT-4, Copilot, Bard and Gemini Ultra on a...
      Evaluating nlp_ent_relevant... [1]
      Evaluating real_world_app... [0]
      Evaluating substantive_study... [1]
      Evaluating research_tool... [0]
      Evaluating data_insights... [0]
      Evaluating clinical_decision... [0]
      Evaluating emr_integration... [0]
    
    Processing paper 4/5
    Title: Leveraging FDA Labeling Documents and Large Language Model t...
      Evaluating nlp_ent_relevant... [0]
      Evaluating real_world_app... [1]
      Evaluating substantive_study... [1]
      Evaluating research_tool... [1]
      Evaluating data_insights... [1]
      Evaluating clinical_decision... [0]
      Evaluating emr_integration... [0]
    
    Processing paper 5/5
    Title: Voice EHR: introducing multimodal audio data for health....
      Evaluating nlp_ent_relevant... [0]
      Evaluating real_world_app... [1]
      Evaluating substantive_study... [1]
      Evaluating research_tool... [0]
      Evaluating data_insights... [1]
      Evaluating clinical_decision... [0]
      Evaluating emr_integration... [1]
    
    ============================================================
    SUMMARY STATISTICS
    ============================================================
    nlp_ent_relevant     1/5 papers (20.0%)
    real_world_app       3/5 papers (60.0%)
    substantive_study    4/5 papers (80.0%)
    research_tool        1/5 papers (20.0%)
    data_insights        3/5 papers (60.0%)
    clinical_decision    1/5 papers (20.0%)
    emr_integration      1/5 papers (20.0%)
    
    API Calls attempted: 35
    Successful calls: 35
    Duration: 40.0 seconds
    Estimated cost: $0.0073
    
    ✓ Results saved to 6_TestResults.csv
      (Papers with -1 values had persistent API failures)
    
    ============================================================
    PAPERS MEETING CORE INCLUSION CRITERIA (NLP + ENT)
    ============================================================
    
    ```
    

---

**3. Choosing 20 Close Studies**
