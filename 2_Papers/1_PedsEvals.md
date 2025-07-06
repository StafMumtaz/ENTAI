# Automated Evaluation of Antibiotic Prescribing Guideline Concordance in Pediatric Sinusitis Clinical Notes

---

**Categorizations**

- Chart automation
- Surveillance
- Pediatrics
- Physician Evaluation
- Prompt engineering
- Open Source
- Good enough for real use
- Sinusitis

---

**Methods**

- Prompt engineering
    - few shot learning and chain of thought
- sample size = 300 random clinical notes from 10k papers  for children

Llama 3 70B-instruct with parameter-efficient fine-tuning and Llama 3.1 405B-instruct, but 405 ended up being better 

- Not done
    
    full fine tuning 
    
    RAG use
    
    self correction via multi agent interactions
    
- Key components of prompt engineering
    
    The role specifying the function the model should adopt when generating response, in our
    case a pediatrician.
    The context which describes the notes; specifically, the input note is a clinical note of a
    patient diagnosed with sinusitis who received antibiotics.
    The question the model should answer.
    The format in which we wanted the model to present its response.
    The text of the note to be classified
    The keyword Answer: to initiate the model's completion according to our specified format.
    
- Example of prompt engineering
    
    Prompt with conditional guidelines
    Role: You are a pediatrician who believes in very rarely prescribing antibiotics and likes to explain why they should
    not be normally prescribed for sinusitis.
    Definitions: A patient has fever if the patient has a body temperature above 102.2°F or 39°C. Nasal discharge
    quality can be a. clear and watery, b. thick and white, c. yellow, green, brown or grey, d. bloody, e. foul-smelling, f.
    thick and stringy, g. purulent. If a patient has nasal passages congested, then the patient has nas al discharges.
    Conditional guidelines: As a pediatrician you would consider that the prescription of an antibiotic was not
    justified unless one or more of the following rule is satisfied:
    Rule 1. If the patient had nasal discharge of any quality for 10 or more days without improvement, then the
    prescription of the antibiotics was justified.
    Rule 2. If the patient was coughing during the day for 10 or more days without improvement, then the
    prescription of the antibiotics was justified.
    Rule 3. If the patient experienced discomfort or pain in the areas around the sinus for 10 or more days without
    improvement, then the prescription of the antibiotics was justified.
    Rule 4. If the patient had nasal discharge of any quality, was recovering, but then experienced an increase of its
    severity or its reappearance, then the prescription of the antibiotics was justified.
    Rule 5. If the patient was coughing during the day, was recovering, but then experienced an increase of its
    severity or its reappearance, then the prescription of the antibiotics was justified.
    Rule 6. If the patient experienced discomfort or pain in the areas around the sinus, was recovering, but then
    experienced an increase in the pain/ discomfort severity or its reappearance, then the prescription of the
    antibiotics was justified.
    Rule 7. If the patient had fever and, on the same time, had purulent nasal discharge for 3 or more consecutive
    days, then the prescription of the antibiotics was justified.
    Rule 8. If the patient had fever and, on the same time, experienced discomfort or pain in the areas around the
    sinus, for 3 or more consecutive days, then the prescription of the antibiotics was justified.
    Instructions: Following is are clinical notes of patients diagnosed with sinusitis and for whom antibiotics were
    prescribed. By default, assume that the prescription of antibiotics was not appropriate unless you find evidence in
    the note indicating that at least one of the preceding rules was satisfied. Answer strictly starting by Yes, No
    • •• Insufficient. Then, give a short explanation of your decision justifying with spans extracted from the note when
    need ed.
    Examples - (note excerpt, answer, explanation, quotes) :
    Note: {note 1 excerpt} Answer: Yes. Explanation: The patient was coughing for two weeks which is more than 10
    days therefore the prescription of antibiotics was justified (Rule 2. is satisfied) Quote: "cough x 2 weeks"
    Note: {note 2 excerpt} Answer: No. Explanation: The patient had nasal passages congested for only 5 days
    which is less than 10 days (Rule 1. is not satisfied). The patient experienced pain in the areas around the sinus
    pain but no duration was specified (Rule 3. is not satisfied). Quote: "Thick yellow green congestion for about 5
    days"
    Note: {Note 3 excerpt} Answer: Insufficient. Explanation: The patient was coughing during the day between 7 to
    10 days. This duration is vague. If the patient was coughing for less than 10 days then the prescription of the
    antibiotics was not justified (Rule 2. is not satisfied), on the contrary, if the patient was coughing for 10 days
    then the prescription of the antibiotics was justified (Rule 2. is satisfied) Quote: "Cough? 7-10 days"
    Note to classify:
    Note: {textual content of the note to classify} Answer:
    

given adapted clinical guidelines for context 

few shot learning

iteratively adjusted prompt to optimize for outputs

- Temperature parameters at some of the lowest possible
    
     temperature of 0.001, top-p of 0.01, and top-k of 1
    

Model places 3 discrete categories: appropriate, inappropriate, ambiguous 

- only on binary + or - antibiotic, not stratified by class or duration
    - appropriate - 1+ criteria clearly met
    - inappropriate - 0 clearly
    - ambiguous - unclear
    - Criteria
        1. Persistent illness: nasal discharge (of any quality), daytime cough, or sinus pain/pressure
        lasting for ≥ 10 days without improvement
        2. Severe onset, i.e., concurrent fever (temperature ≥ 39°C/102.2°F) and purulent nasal
        discharge or sinus pain/pressure for at least 3 consecutive days
        3. Worsening course, i.e., worsening or new onset of nasal discharge, daytime cough, sinus
        pain/pressure, or fever after initial improvement

inclusion: cases of sinusitis acute or chronic + antibiotic prescription

exclusion: o/ chronic disease, set of antibiotics that would not be prescribed for sinusitis, o/ reason for antibiotics

Data was split: trained on 200 from 80% of providers → tested on 50 from same providers and 50 from o/ 20%

---

**Introduction** 

Antibiotic stewardship programs: ASPs - public health attempts at making antibiotic use more specific to prevent tolerance 

30% of antibiotic prescriptions are unnecessary 

---

**Results**

- Llama 3.1 405B instruct outperforms Llama 3 70B instruct model despite PEFT
- few shot learning and chain of thought improved results
- 95% identification for 150 + notes for antibiotic prescription
- 65% identification for 80 - notes for antibiotic prescription
- 0% identified for 15 ambiguously documented notes
- Thought it was good enough to deploy for real time assistance or retroactive review

---

**Recommendations**

Could be deployed as:  

1. Retrospective provider feedback, public health tracking, anomaly detection 
2. Real-time recommendation generator although not suitable with incomplete information 

---

**Limitations**

- Only data from one clinic
- Selection criteria required prescription, didn’t include people who might have called for them without

---

**Notes to self and Implications**

- Larger models are better, fine tuning doesn’t matter that much when the models get smart enough
- [https://pubmed.ncbi.nlm.nih.gov/34189181/](https://pubmed.ncbi.nlm.nih.gov/34189181/) is a paper from 2021 that uses NLP but almost certainly rule based or shallow statistical NLP, with traditional machine learning classifiers
    - So that tells us there was more rudimentary NLP that happened from before
- real short coming of this model is where notes are incomplete
- False positives were something you can make sense of and see the reasoning issues with and then correct with additional caveats to the prompting or guidelines it seems
- **all the ambiguity cases where model failed seem to do with how it handled time in a way that you could easily specify for correction in prompt**