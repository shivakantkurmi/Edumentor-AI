# EduMentor AI: Personalized Learning Assistant

EduMentor AI is a generative AI-powered learning assistant built with the Gemini 1.5 Flash API. It creates personalized study plans, explains concepts, and generates quizzes based on user inputs like topic, difficulty, and learning style, using prompt engineering, Retrieval-Augmented Generation (RAG), and a Gradio interface.

This repository contains the notebook (`edumentor-ai.ipynb`) and is also available on Kaggle: [Kaggle Notebook](https://www.kaggle.com/code/shivakantkurmi/edumentor-ai).

## Prerequisites

- Python 3.11 or higher
- Jupyter Notebook (for local use) or a Kaggle account (for Kaggle use)
- A Google API key from [Google AI Studio](https://aistudio.google.com/)

## Setup Instructions

Run the notebook on **Kaggle** (recommended for simplicity) or **locally**. Follow the steps below to add your `GOOGLE_API_KEY` securely.

### Option 1: Run on Kaggle

1. **Fork the Notebook**:
   - Visit the Kaggle notebook: [Kaggle Notebook](https://www.kaggle.com/code/shivakantkurmi/edumentor-ai).
   - Click **"Copy & Edit"** to create your own editable version in your Kaggle account.

2. **Add Your API Key to Kaggle Secrets**:
   - In your forked notebook, go to the **Notebook options** panel (right sidebar).
   - Under **Secrets**, click **"Add a new secret"**.
   - Set:
     - **Label**: `GOOGLE_API_KEY`
     - **Value**: Your Google API key (obtained from [Google AI Studio](https://aistudio.google.com/))
   - Save and ensure the secret is enabled for your notebook.
   - The notebook uses `UserSecretsClient` to securely access this key.

3. **Run the Notebook**:
   - Execute all cells by clicking **Run All** or run each cell individually.
   - The notebook installs dependencies automatically and uses your `GOOGLE_API_KEY` to interact with the Gemini API.

### Option 2: Run Locally

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/shivakantkurmi/Edumentor-AI.git
   cd Edumentor-AI
   ```

2. **Install Dependencies**:
   Install the required packages:
   ```bash
   pip install -r requirements.txt
   ```
   The `requirements.txt` includes:
   ```
   google-generativeai
   langchain
   langchain-community
   langchain-huggingface
   faiss-cpu
   gradio
   pandas
   seaborn
   jupyter
   python-dotenv
   kaggle
   ```

3. **Add Your API Key**:
   Obtain a Google API key from [Google AI Studio](https://aistudio.google.com/) and set it using one of these methods:

   **Method 1: Environment Variable**  
   - **Linux/Mac**:
     ```bash
     export GOOGLE_API_KEY="your-api-key"
     ```
   - **Windows (Command Prompt)**:
     ```bash
     set GOOGLE_API_KEY=your-api-key
     ```
   - Add to `~/.bashrc`, `~/.zshrc`, or System Environment Variables for persistence.

   **Method 2: .env File**  
   - Create a `.env` file in the project directory:
     ```
     GOOGLE_API_KEY=your-api-key
     ```
   - The notebook loads this using `python-dotenv`. Ensure `.env` is in `.gitignore`.

4. **Run the Notebook**:
   ```bash
   jupyter notebook edumentor-ai.ipynb
   ```
   Execute the cells to generate study plans, explanations, or quizzes.

## Troubleshooting

- **API Key Error**: Ensure `GOOGLE_API_KEY` is set correctly in Kaggle Secrets (for Kaggle) or environment/.env file (for local). Verify the key is valid in Google AI Studio.
- **Dependency Issues**: Rerun `pip install -r requirements.txt` or use Python 3.11. Check for conflicts (e.g., `pylibcugraph-cu12`) and update packages if needed.
- **Kaggle Secrets**: Confirm the secret is enabled in the notebook’s settings.
- **Clear Outputs**: For local use, clear outputs if errors occur:
  ```bash
  jupyter nbconvert --ClearOutputPreprocessor.enabled=True --inplace edumentor-ai.ipynb
  ```

## Notes

- Your Google API key is private and should not be shared or committed to the repository.
- On Kaggle, fork the notebook and add your own `GOOGLE_API_KEY` to Kaggle Secrets.
- Locally, use a `.env` file or environment variable to securely manage your API key.

## License

This project is licensed under the [MIT License](LICENSE).

---

Built by Shivakant Kurmi
