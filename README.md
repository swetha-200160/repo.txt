# My Python Project

This is a sample Python project structured for CI with Jenkins.

Files included:
- app/main.py : sample entrypoint
- tests/test_main.py : sample pytest test
- requirements.txt : project dependencies
- setup.py : packaging configuration
- Jenkinsfile : example Jenkins pipeline (Windows agent)
- .gitignore : files to ignore

Instructions:
1. Create a virtual environment: `python -m venv venv`
2. Activate venv and install deps: `pip install -r requirements.txt`
3. Run tests: `pytest -q`
4. Build package: `python -m pip wheel . -w dist` or `python setup.py sdist bdist_wheel`
