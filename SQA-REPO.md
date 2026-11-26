# SQA REPO - WooSam-FALL2026-COMP5710 Project

**Team:** WooSam - Woosol Kim (wzk0029@auburn.edu), Sam Germanson (stg0025@auburn.edu)

## 1. Completion

All activities are completed. (project uploaded, README with team members, fuzzing (4.a), forensics (4.b), continuous integration (4.c), and SQA REPO)

## 2. How to identify our implementation

### 4.a Fuzzing 'fuzz.py'

'fuzz.py' sends 2000 random inputs plus a deeply nested input to each of 10 methods and records any uncaught exception as a bug. The results go to 'fuzz_report.md' and the inputs that reproduce the bugs are in 'fuzz_repros/'

**Methods fuzzed:**
- **parser:** 'KeyMiner', 'getKeyRecursively', 'getValuesRecursively', 'getValsFromKey', 'getSingleDict4MultiDocs', 'find_json_path_keys'
- **scanner:** 'checkIfValidSecret', 'scanForSecrets', 'getItemFromSecret'
- **graphtaint:** 'constructHelmString'

**Bugs found (6):**

| Method | Bug |
|--------|-----|
| 'parser.keyMiner' | 'RecursionError' on deeply nested dicts |
| 'parser.getValuesRecursively' | 'RecursionError' on deeply nested dicts |
| 'parser.getSingleDict4MultiDocs' | 'IndexError' on empty lists |
| 'parser.getSingleDict4MultiDocs' | 'TypeError' on a 'None' key |
| 'parser.find_json_path_keys' | 'TypeError': 'None' passed to a regex |
| 'graphtaint.constructHelmString' | 'TypeError': 'str + int' |

### 4.b Forensics - 'forensics.py', 'run_forensics.py'

Each of 10 methods carries the '@forensics.forensic' decorator, which logs every CALL, RETURN, EXCEPTION and EVENT to 'forensics.log'. Logging never changes program behavior. Because SLI-KUBE hunts for secrets, raw values are never logged, only their type, length and a salted hash. Every run also logs the Python version, OS, GitHub run ID and commit SHA. 'run_forensics.py' exercises all 10 methods and writes a summary to 'forensics_logs/summary.md'.

**Methods instrumented:**
- **parser:** 'checkIfWeirdYAML', 'checkParseError', 'loadMultiYAML', 'checkIfValidK8SYaml', 'getSingleDict4MultiDocs'
- **scanner:** 'getYAMLFiles', 'checkIfValidSecret', 'scanForSecrets', 'scanForTruePID', 'scanForDefaultNamespace'
### 4.c Continuous integration — '.github/workflows/ci.yml'
 
The workflow runs on every push and pull request to 'main'. It runs lint, unit tests, 'fuzz.py' and 'run_forensics.py', posts the fuzzing and forensics results to the run's job summary, and uploads them as the 'sqa-reports' artifact. The lint step also caught a pre-existing bug: 'main.py' uses 'Path' and 'typer' without importing them.
 
### Screenshots
 
- ['local-fuzz.png'](screenshots/local-fuzz.png): fuzzing run locally
- ['local-forensics.png'](screenshots/local-forensics.png): forensics run locally
- ['ci-run.png'](screenshots/ci-run.png): a successful CI run
- ['ci-summary.png'](screenshots/ci-summary.png): fuzzing and forensics results in CI
- ['ci-artifacts.png'](screenshots/ci-artifacts.png): the uploaded 'sqa-reports' artifact
## 3. How to replicate
 
```bash
git clone https://github.com/wzk0029/WooSam-FALL2026-COMP5710-project.git
cd WooSam-FALL2026-COMP5710-project
python -m pip install ruamel.yaml pandas numpy sarif-om jschema-to-python typer
python fuzz.py
python run_forensics.py TEST_ARTIFACTS --max-files 10
```
 
To run CI, push to 'main' or use **Actions → SQA CI → Run workflow**.
 
## 4. What we learned
 
We learned that passing unit tests doesn't mean code is robust, since fuzzing found 6 crashes the existing test never triggered. We also learned that forensic logging in a security tool has to avoid recording sensitive values, so we logged only types, lengths and hashes. Fianlly, running everything in GitHub Actions showed us how CI catches problems automatically, like the missing imports in main.py.
