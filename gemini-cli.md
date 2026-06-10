---
### The No-Installer, 100% Reliable Solution

We are going to use an official, standalone script to run Gemini directly. You don't need to fix Python or Winget for this to work.

#### Step 1: Open PowerShell as Administrator

Right-click your **Windows Terminal** or **PowerShell** icon and select **Run as administrator**.

#### Step 2: Paste this command to download and run the Gemini CLI

Copy and paste this entire block into your terminal and press **Enter**:

```powershell
# 1. Create a tools directory
New-Item -ItemType Directory -Force -Path "C:\GeminiCLI"

# 2. Download a lightweight, standalone Python engine that doesn't need installation
Invoke-WebRequest -Uri "https://www.python.org/ftp/python/3.11.9/python-3.11.9-embed-amd64.zip" -OutFile "C:\GeminiCLI\python.zip"
Expand-Archive -Path "C:\GeminiCLI\python.zip" -DestinationPath "C:\GeminiCLI\engine"

# 3. Download the pip installer bootstrap
Invoke-WebRequest -Uri "https://bootstrap.pypa.io/get-pip.py" -OutFile "C:\GeminiCLI\get-pip.py"

# 4. Initialize the standalone engine
& "C:\GeminiCLI\engine\python.exe" "C:\GeminiCLI\get-pip.py"

# 5. Enable site-packages so the engine can see installed tools
Add-Content -Path "C:\GeminiCLI\engine\python311._pth" -Value "import site"

# 6. Install the LLM tool directly into our isolated folder
& "C:\GeminiCLI\engine\Scripts\pip.exe" install llm llm-gemini

```

---

#### Step 3: Link your API Key

Now that our isolated engine is built, run this command to save your Gemini API key:

```powershell
& "C:\GeminiCLI\engine\Scripts\llm.exe" keys set gemini

```

*It will ask you to input your key. Paste your API key from Google AI Studio and hit **Enter**.*

---

#### Step 4: Create a permanent shortcut command

To make sure you don't have to type `C:\GeminiCLI\...` every time, run this final command to create a simple shortcut named `gemini`:

```powershell
Function gemini { & "C:\GeminiCLI\engine\Scripts\llm.exe" -m gemini-2.5-flash $args }

```

### 🚀 Try it out!

You can now talk to Gemini instantly from this terminal window like this:

```powershell
gemini "Write a quick batch script to clear my DNS cache"

```
