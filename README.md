# MLOps CI/CD Pipeline Using GitHub Actions

## Project Overview

This repository demonstrates the design and implementation of an end-to-end Continuous Integration and Continuous Deployment (CI/CD) pipeline for a Machine Learning application using **GitHub Actions**.

In modern MLOps (Machine Learning Operations), software engineering discipline is applied to machine learning lifecycles. Automating testing and deployment workflows ensures that every model alteration, code refactoring, or dependency update is automatically verified for correctness and baseline performance before progressing toward deployment.

The pipeline automates:
1. **Continuous Integration (CI):** On every code push or pull request targeting the `main` branch, the workflow provisions an automated environment, installs project dependencies, and runs unit tests via `pytest` to evaluate model accuracy against quality gates.
2. **Continuous Deployment (CD):** When tests pass successfully on the `main` branch, the deployment stage automatically triggers, simulating a continuous deployment process. If tests fail or if the event originates from a pull request, deployment is safely bypassed.

---

## Technologies Used

- **Python 3.11**: Programming language and runtime environment.
- **Scikit-learn**: Machine learning library used for dataset extraction, train-test splitting, model training, and performance metrics.
- **Logistic Regression**: Linear classification algorithm configured with `max_iter=200` to categorize Iris species.
- **Pytest**: Automated testing framework used to implement regression tests and enforce quality gates.
- **Git**: Distributed version control system tracking local development history.
- **GitHub**: Remote repository host providing collaboration and pull request workflows.
- **GitHub Actions**: Cloud-native CI/CD orchestration platform executing test and deployment runners.
- **Conda**: Package and environment management system utilized for local development.

---

## Dataset

The project utilizes the classic built-in **Iris Dataset** from `sklearn.datasets`:
- **Total Samples**: 150 instances (balanced across classes).
- **Input Features (4 features)**:
  - Sepal length (cm)
  - Sepal width (cm)
  - Petal length (cm)
  - Petal width (cm)
- **Target Classes (3 flower species)**:
  - Iris setosa (0)
  - Iris versicolor (1)
  - Iris virginica (2)
- **Data Partitioning**: Split using an 80/20 train-test ratio (`test_size=0.2`) with a fixed seed (`random_state=42`) to guarantee reproducible test evaluations across environments.

---

## Project Structure

```text
mlops-cicd/
├── .github/
│   └── workflows/
│       └── ci-cd.yml         # GitHub Actions CI/CD pipeline definition
├── src/
│   └── model.py              # ML model training and evaluation script
├── tests/
│   └── test_model.py         # Pytest test suite asserting model accuracy
├── app.py                    # Entry point application script
├── README.md                 # Project documentation
└── requirements.txt          # Production and testing dependencies
```

### File Responsibilities:
- **`src/model.py`**: Encapsulates `train_model()`, which loads data, trains the `LogisticRegression` model, computes test accuracy, and prints the evaluated score when run as a standalone script.
- **`tests/test_model.py`**: Automates regression testing by importing `train_model()` and asserting that accuracy meets or exceeds the required threshold of $0.80$ ($80\%$).
- **`app.py`**: Serves as the main application entry point that executes model training and outputs status messages.
- **`requirements.txt`**: Declares required Python packages (`scikit-learn` and `pytest`).
- **`.github/workflows/ci-cd.yml`**: Defines the multi-stage CI/CD pipeline containing automated testing and simulated deployment jobs.

---

## Environment Setup

Follow these Windows PowerShell commands to activate the Conda environment and install required packages:

1. **Open PowerShell and navigate to the project directory:**
   ```powershell
   cd "D:\SEM 5\MLAO\Assignment 7\mlops-cicd"
   ```

2. **Activate the existing Conda environment:**
   ```powershell
   conda activate mlops-cicd-env
   ```

3. **Install project dependencies:**
   ```powershell
   pip install -r requirements.txt
   ```

---

## Running the Application

### 1. Execute the Model Training Script
To directly train the model and display the accuracy:
```powershell
python src/model.py
```
**Expected Output:**
```text
Model Accuracy: 0.xxxx
```

### 2. Execute the Main Application
To run the primary application entry point:
```powershell
python app.py
```
**Expected Output:**
```text
ML application started.
Model Accuracy: 0.xxxx
```

---

## Automated Testing

Automated testing is configured using **Pytest** to prevent performance regressions from entering the codebase.

- The test file `tests/test_model.py` defines `test_model_accuracy()`.
- It executes `train_model()` and validates that the resulting evaluation metric satisfies:
  $$\text{Accuracy} \ge 0.80$$

To run the unit tests locally with detailed verbosity:
```powershell
python -m pytest -v
```

**Expected Test Output:**
```text
============================= test session starts =============================
collecting ... collected 1 item

tests/test_model.py::test_model_accuracy PASSED                          [100%]

============================== 1 passed in x.xxs ==============================
```

---

## GitHub Actions CI/CD Pipeline

The automated workflow is declared in [`.github/workflows/ci-cd.yml`](.github/workflows/ci-cd.yml) under the workflow name **MLOps CI/CD Pipeline**.

### Triggers
The workflow triggers automatically on:
- `push` events to the `main` branch.
- `pull_request` events targeting the `main` branch.

### Pipeline Stages

```text
Code Push / Pull Request
           │
           ▼
┌──────────────────────────────────────┐
│       Job 1: Test ML Application     │
│  - Checkout repository               │
│  - Setup Python 3.11                 │
│  - Install dependencies              │
│  - Run pytest (accuracy >= 0.80)     │
└──────────────────┬───────────────────┘
                   │
           Did tests pass?
           ├── No  ──► Pipeline Fails (Deploy Skipped)
           └── Yes
                   │
         Is event a push to main?
           ├── No (Pull Request) ──► Deploy Skipped
           └── Yes (Push to main)
                   │
                   ▼
┌──────────────────────────────────────┐
│      Job 2: Deploy Application       │
│  - Checkout repository               │
│  - Simulate deployment via echo      │
└──────────────────────────────────────┘
```

1. **Test Job (`test` / "Test ML Application")**:
   - Runs on an `ubuntu-latest` virtual runner.
   - Checks out the repository using `actions/checkout@v4`.
   - Configures Python 3.11 via `actions/setup-python@v5`.
   - Upgrades `pip` and installs packages declared in `requirements.txt`.
   - Runs `python -m pytest -v`.

2. **Deploy Job (`deploy` / "Deploy Application")**:
   - Runs on `ubuntu-latest`.
   - **Job Dependency**: Configured with `needs: test`. In GitHub Actions, downstream jobs run only if dependent jobs finish with a `success` status.
   - **Branch Guard**: Protected by `if: github.ref == 'refs/heads/main'`. This guarantees that deployment executes only when code is pushed directly or merged into `main`, and is skipped during pull requests.

---

## Deployment Simulation

In this academic assignment, deployment is intentionally simulated to demonstrate orchestration logic without requiring cloud infrastructure overhead:
- The deployment step executes shell echo commands:
  ```bash
  echo "Deployment started..."
  echo "ML application deployed successfully."
  ```
- *Clarification*: The project is not connected to external cloud hosting (such as AWS, Azure, GCP, or container registries). This simulated stage represents where container build, registry push, or serverless deployment commands would execute in production.

---

## Test Failure Demonstration

To illustrate CI/CD enforcement and quality gating:

1. **Introducing a Failure Condition**:
   - In `tests/test_model.py`, the assertion is temporarily modified to an unachievable threshold:
     ```python
     assert accuracy >= 0.99
     ```
2. **Pipeline Behavior During Failure**:
   - The `test` job executes on GitHub Actions. Because the model achieves standard Iris classification performance below $0.99$, the test raises an `AssertionError`.
   - The `test` job is marked as **FAILED**.
   - Because `deploy` specifies `needs: test`, GitHub Actions automatically halts the pipeline and marks the `deploy` job as **SKIPPED**.
3. **Restoring the Valid Quality Gate**:
   - The assertion is restored to its proper threshold:
     ```python
     assert accuracy >= 0.80
     ```
   - Pushing the fix allows the `test` job to succeed and unlocks the subsequent `deploy` job.

This scenario proves that non-compliant models cannot inadvertently reach the deployment stage.

---

## CI/CD Benefits in Machine Learning

1. **Automated Validation**: Code quality and model behavior are verified consistently on clean virtual runners without manual intervention.
2. **Early Error Detection**: Syntax errors, broken dependencies, and performance drops are exposed at commit time before merging.
3. **Reproducible Environments**: Standardized runner configurations (`ubuntu-latest`, Python 3.11) remove the "works on my machine" discrepancy.
4. **Guaranteed Deployment Safety**: Hard quality gates ensure untested or underperforming models are blocked from production environments.

---

## Conclusion

This project successfully demonstrates an automated MLOps CI/CD pipeline using GitHub Actions for a machine learning model. By pairing version-controlled Python ML code with automated unit tests and conditional multi-stage workflows, it establishes an agile, reproducible, and fault-tolerant release cycle suitable for continuous integration and continuous deployment.
