# Day 1 — Python Virtual Environment

## 🎯 Task

Create an ML Python environment under `/root/code/`.

## 🧠 Concepts

- Python virtual environments
- Dependency isolation
- pip
- requirements.txt
- Reproducibility

## 🔧 Commands

```bash
cd /root/code

python -m venv ml-env

source ml-env/bin/activate

pip install numpy pandas scikit-learn matplotlib

pip freeze > requirements.txt