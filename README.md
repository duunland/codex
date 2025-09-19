 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/README.md b/README.md
index c22b6eeb8eed8f68fa6a446eb09aabbceca7f4cd..95fc44f1bb0b81bcd0a218030e16209ef71ecf26 100644
--- a/README.md
+++ b/README.md
@@ -1,34 +1,114 @@
-# Projektname
+# News Article Coding Agent
 
-Kurze Beschreibung:  
-Was macht dein Projekt? Warum existiert es? Zum Beispiel: „Dieses Projekt hilft dabei, Aufgaben zu verwalten“, oder „Ein kleines Web-Tool, das …“.
+This project provides a lightweight command line agent that downloads online news
+articles, extracts their main textual content and codes the text using a user
+provided codebook. Before coding, the agent determines whether the story matches
+the `Protest Event` definition from the codebook and logs the justification so
+that analysts can review every decision. The agent can optionally forward
+observations to the OpenAI Agents SDK so that results improve with each
+processed article.
 
----
+In addition to the CLI, a Streamlit application offers an interactive interface
+for uploading a codebook, processing URLs, and manually providing article text
+when automatic extraction is unsuccessful.
 
-## Inhaltsverzeichnis
+## Features
 
-- [Features](#features)  
-- [Installation](#installation)  
-- [Nutzung](#nutzung)  
-- [Beispiel](#beispiel)  
-- [Mitwirken](#mitwirken)  
-- [Lizenz](#lizenz)  
-- [Kontakt](#kontakt)
+- Fetches the HTML of each article using a configurable user agent header.
+- Attempts multiple extraction strategies (Trafilatura, Readability, BeautifulSoup,
+  and a custom parser) to recover article text.
+- Prompts the operator for manual text input when every automated strategy fails.
+- Applies a keyword based codebook to count occurrences per article.
+- Classifies whether each article matches the `Protest Event` code and explains why.
+- Records reviewable decisions and supports continuous learning via the OpenAI Agents SDK.
+- Outputs structured JSON that can be saved to disk or printed to the console.
+- Provides a Streamlit web UI for running the agent interactively.
 
----
+## Installation
 
-## Features
+Create and activate a Python virtual environment (Python 3.9+), then install the
+package and its dependencies:
 
-- Feature 1: Kurz und bündig  
-- Feature 2: Was dein Projekt besonders macht  
-- Feature 3: etc.
+```bash
+python -m venv .venv
+source .venv/bin/activate
+pip install -e .
+```
 
----
+Optional extras are available when you want to use the richer extraction
+strategies or the web interface:
 
-## Installation
+```bash
+# Install advanced extraction helpers
+pip install -e .[extractors]
+
+# Install dependencies required for the Streamlit app
+pip install -e .[app]
+```
+
+## Usage
 
-Diese Schritte zeigen, wie du das Projekt lokal aufsetzt.
+1. Create a JSON codebook. The example below defines two codes with a list of
+   keywords that should be matched in the article text.
+
+   ```json
+   {
+     "codes": [
+       {"name": "Economy", "description": "Economic coverage", "keywords": ["inflation", "economy"]},
+       {"name": "Politics", "description": "Political coverage", "keywords": ["election", "president"]}
+     ]
+   }
+   ```
+
+2. Run the agent with one or more article URLs:
+
+   ```bash
+   news-agent --codebook path/to/codebook.json "https://example.com/news/story"
+   ```
+
+   Use the `--output` flag to write the JSON result to a file:
 
-1. Repository klonen  
    ```bash
-   git clone https://github.com/dein-benutzername/dein-projektname.git
+   news-agent --codebook path/to/codebook.json --output results.json "https://example.com/news/story"
+   ```
+
+   The command will download each article, extract the text and output JSON
+   containing the protest event assessment, the extracted length, and the matched
+   keyword counts for every code. When automated extraction does not yield
+   content, the CLI will ask you to paste the article text manually (unless
+   `--no-manual` is supplied). Each response also contains a list of decisions so
+   that users can audit what happened.
+
+   To stream observations to an existing OpenAI agent for iterative improvement,
+   provide the model or agent identifier via `--openai-agent-id`:
+
+   ```bash
+   news-agent --codebook path/to/codebook.json --openai-agent-id <agent-id> \
+     "https://example.com/news/story"
+   ```
+
+   Use `--disable-learning` if you want to suppress all learning behaviour and
+   keep observations in memory only.
+
+### Streamlit app
+
+To launch the interactive UI, install the optional dependencies and run:
+
+```bash
+streamlit run -m news_agent.streamlit_app
+```
+
+The app mirrors the CLI workflow: upload your codebook, enter a URL, and provide
+manual text if none of the automatic extraction strategies succeed. It displays
+the protest event decision, the coding table, and the full list of reviewable
+decisions. Configure the sidebar to forward observations to the OpenAI Agents
+SDK or keep them on the local device. Results can be explored directly in the
+browser or downloaded as JSON.
+
+## Development
+
+Run the unit tests with `pytest`:
+
+```bash
+pytest
+```
 
EOF
)
