LLM Validation with a DSL - DemoThis project demonstrates a system that uses a Domain-Specific Language (DSL) to validate and correct the output of a Large Language Model (LLM).The goal is to show how a rules engine can act as a safety net, enforcing business logic on the structured data extracted by an LLM from a natural language query. This ensures the final output is both intelligent and compliant.The Rules EngineThis demo is built using the open-source business-rules library for Python.Description: It's a lightweight engine where rules are defined as data (JSON) and are completely separate from the execution code.Format: Rules consist of conditions (the "if" part) and actions (the "then" part). Our demo uses the actions to define calculation formulas.Documentation: For a full understanding of the library's capabilities, refer to the original repository: https://github.com/venmo/business-rulesHow to Use1. Prerequisites: Install OllamaThis demo uses a locally-run open-source LLM. You must have Ollama installed and have downloaded the Llama 3 8B model.Install Ollama: Download from https://ollama.com and run the installer.Download the Model: Open a terminal and run:ollama pull llama3:8b
2. Setup the Python EnvironmentThe demo runs inside a Jupyter Notebook. The following steps will set up the necessary Python environment.Navigate to the Demo Folder:cd demo_llm_validator
Create a Virtual Environment:python -m venv venv
Activate the Environment:On Windows (PowerShell): .\venv\Scripts\Activate.ps1On Mac/Linux: source venv/bin/activate(Note: On PowerShell, you may first need to run Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process)Install Dependencies:pip install -r requirements.txt
3. Run the DemoStart the Jupyter Server: In your active terminal, run:jupyter notebook
Open the Notebook: A new tab will open in your browser. Click on llm_validation_demo.ipynb.Run the Cells: Execute each cell in the notebook in order by selecting it and pressing Shift + Enter. The final cell will display the interactive demo UI.Example Run-through: German Tax CalculationThe demo uses the scenario of calculating a restaurant bill according to specific German tax laws. This is a good example because the calculation rules are non-negotiable, and it's a domain where an LLM might make a reasoning error.The DSL RulesThe "source of truth" for the calculation is defined in our rules. The logic is contained in the actions, as the calculations should run every time.# From Cell 1 in the notebook
tax_calculation_rules = [
    {
        "name": "Calculate Food Tax (7%)",
        "conditions": { "all": [] },
        "actions": [
            {
                "name": "calculate_tax_on_variable",
                "params": {
                    "input_variable_name": "net_food_cost",
                    "tax_rate": 0.07,
                    "output_variable_name": "food_tax"
                }
            }
        ]
    },
    # ... (other rules for drink tax and total bill) ...
]
The LLM PromptWhen the user enters an order, the system constructs a detailed prompt asking the LLM to both extract the base costs and attempt the full calculation. This gives the LLM a clear opportunity to make a reasoning error.# From Cell 4 in the notebook
prompt = f"""
You are a restaurant billing assistant. Your tasks are:
1. Extract the 'net_food_cost' and 'net_drink_cost' from the user's order.
2. Attempt to calculate the 'total_tax' (food tax is 7%, drink tax is 19%).
3. Attempt to calculate the 'final_bill'.
4. Respond ONLY with a JSON object containing these four fields.

User Order: "{user_query}"

JSON Response:
"""
System FlowLLM Processing: The user's order is sent to the local Ollama LLM with the prompt above.JSON Extraction: The system's code intelligently finds and parses the JSON block from the LLM's (potentially messy) text response. If it can't find a valid JSON object, it reports a hallucination error.DSL Engine Calculation: The system then uses the BillActions class to execute the formulas defined in our tax_calculation_rules on the data extracted by the LLM. This produces a separate, guaranteed-correct calculation.Comparison: The final output is a table that shows a side-by-side comparison of the LLM's calculated values and the DSL-enforced values, with a status
