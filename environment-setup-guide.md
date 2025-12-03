How to Configure a Secure Python Virtual Environment
Prerequisites: Python 3.10+, Terminal/Command Prompt access.

This guide outlines the standard procedure for setting up an isolated development environment. This ensures dependency integrity and prevents conflicts with the system-wide Python installation.

Step 1: Initialize the Environment
Navigate to your project directory and run the standard venv module.

Windows:

Bash

python -m venv venv
Mac/Linux:

Bash

python3 -m venv venv
Step 2: Activate the Environment
You must activate the environment before installing libraries. You will know it is active when you see (venv) preceding your command prompt.

Windows: .\venv\Scripts\activate

Mac/Linux: source venv/bin/activate

Step 3: Install Dependencies with Hash Checking
To ensure security compliance and prevent supply chain attacks, install packages using a hashed requirements.txt.

Bash

pip install --require-hashes -r requirements.txt
Note: If you encounter a hash mismatch error, do not force the installation. Contact the Security Operations team immediately.
