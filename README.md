# ATS Resume Scorer

I built this app to help people see how their resume may perform in an Applicant Tracking System (ATS) and how closely it matches a job description. It has a Streamlit interface, a FastAPI backend, NLP models, and Groq-powered resume parsing.

> **Note:** This project is set up for local development. You’ll need your own Supabase and Groq credentials to use resume analysis and account features. Use sample or redacted resumes when trying it out.

## Features

- Upload a PDF, DOC, or DOCX resume (up to 5 MB) and get an overall ATS score.
- See how the resume scores for formatting, keywords, content, skills, and ATS compatibility.
- Add a job description to see matching keywords and skills, along with potential gaps.
- Read suggestions for improving the resume, then download a text summary or PDF report.
- Sign in with Supabase to revisit or delete past analyses. Google sign-in is available when configured.

## How it works

1. The API reads the text from the uploaded resume.
2. Groq identifies resume details and, if you provide one, key requirements in the job description.
3. spaCy and Sentence Transformers help compare skills and meaning, not just matching words.
4. The app shows the score, category breakdown, job match, and suggestions.
5. If you’re signed in, the analysis can be saved to your Supabase history. You can also download a report.

## Technology

| Area | Tools |
| --- | --- |
| User interface | Streamlit |
| API | FastAPI, Uvicorn, Pydantic |
| NLP and similarity | spaCy (`en_core_web_md`), Sentence Transformers (`all-MiniLM-L6-v2`), RapidFuzz |
| Structured extraction | Groq API (Llama 3.3 70B Versatile) |
| Authentication and history | Supabase Auth and REST API |
| Resume parsing and reports | pdfplumber, python-docx, Jinja2, WeasyPrint |

## Repository layout

```text
backend/                 FastAPI app, routes, analysis services, and report templates
frontend/                Streamlit app, views, and UI components
jupyter notebooks/       Research and dataset-preparation notebooks
requirements.txt         Python dependencies for both services
```

## Run locally

### Requirements

- Windows, macOS, or Linux
- Python 3.12 recommended
- A Supabase project and a Groq API key for full resume analysis

### 1. Get the source and create an environment

```bash
git clone https://github.com/prithvi1320/ai-resume-ats.git
cd ai-resume-ats
python -m venv .venv
```

Activate the environment:

```bash
# Windows PowerShell
.\.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

If Windows PowerShell blocks activation, allow it for the current terminal only, then activate:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

### 2. Install dependencies and the spaCy model

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m spacy download en_core_web_md
```

The Sentence Transformers model is downloaded automatically on the first backend start and cached for subsequent runs. WeasyPrint may require additional system libraries on Linux; see the [WeasyPrint installation guide](https://doc.courtbouillon.org/weasyprint/stable/first_steps.html#installation).

### 3. Configure environment variables

Create a `.env` file in the repository root and fill it with credentials from your own services:

```dotenv
SUPABASE_URL=https://YOUR_PROJECT_ID.supabase.co
SUPABASE_KEY=YOUR_SUPABASE_SERVICE_ROLE_KEY
SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
SUPABASE_JWT_SECRET=YOUR_SUPABASE_JWT_SECRET
GROQ_API_KEY=YOUR_GROQ_API_KEY
```

Get the Supabase values from your project settings and the Groq key from [Groq Console](https://console.groq.com/). Keep the service role key private; it has elevated database access. The frontend reads Supabase URL and anon key from this root `.env`. Google sign-in is optional and must also be configured in Supabase Auth.

For analysis history, create an `analyses` table in Supabase with columns compatible with the data sent by the backend: `id` (UUID primary key), `user_id` (UUID), `filename` (text), `ats_score` (numeric), `keyword_match` (numeric), `missing_keywords` (JSONB), `created_at` (timestamp with time zone), and `analysis_result` (JSONB). The backend accesses this table through the service role key.

### 4. Start the API

In a terminal at the repository root with `.venv` activated:

```bash
uvicorn backend.main:app --reload --host 127.0.0.1 --port 8000
```

Keep this terminal running. The API is available at `http://localhost:8000`; interactive endpoint documentation is at `http://localhost:8000/docs`.

### 5. Start the Streamlit app

Open a second terminal, go to the repository root, and activate `.venv` in that terminal too. Then run:

```bash
streamlit run frontend/streamlit_app.py
```

Open the local URL printed by Streamlit, usually `http://localhost:8501`. Keep both terminals running while using the application. Press `Ctrl+C` in each terminal to stop its service.

## Configuration notes

- Resume analysis uses the Groq API to extract structured resume information, so a valid `GROQ_API_KEY` is required for analysis.
- Supabase credentials enable sign-up/sign-in and user history. The app expects the `analyses` table described above for persistent history.
- The backend defaults to `http://localhost:8000` for frontend API requests. To use another backend URL, set Streamlit secrets at `frontend/.streamlit/secrets.toml`:

  ```toml
  [backend]
  url = "http://localhost:8000"
  ```

- Job descriptions can be pasted into the app or uploaded as `.txt`; resume uploads support PDF, DOC, and DOCX.

## API overview

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/api/v1/health` | Check whether NLP models loaded |
| `POST` | `/api/v1/analyze-resume` | Analyze an uploaded resume and optional job description |
| `GET` | `/api/v1/history` | List the signed-in user's analyses |
| `DELETE` | `/api/v1/history/{analysis_id}` | Delete a history entry |
| `POST` | `/api/v1/generate-pdf` | Generate a PDF report |

Except for the health check, API routes require a Supabase access token in the `Authorization: Bearer <token>` header.

## Security

- Never commit `.env`, Streamlit secrets, or API keys.
- Use only your own service credentials. Do not publish a Supabase service role key.
- Test with sample or redacted resumes before using documents containing personal data.

## Resume project description

**ATS Resume Scorer** | Python, FastAPI, Streamlit, spaCy, Sentence Transformers, Supabase, Groq

Built a web app that reads PDF, DOC, and DOCX resumes and gives an ATS score across five categories. Added job-description matching with keyword and semantic comparisons, resume feedback, Supabase sign-in and analysis history, and downloadable reports.

## License

No license file is currently included. Add a license before permitting reuse or redistribution.
