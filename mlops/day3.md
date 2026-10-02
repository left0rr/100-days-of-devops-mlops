# Day 3 — Python Dependencies with uv

## 🎯 Task

Correct the dependency specification in `/root/code/fraud-detection/requirements.in` and use `uv` to compile it into a pinned `requirements.txt` lockfile.

## 🧠 Concepts

- Python dependencies
- requirements.in
- requirements.txt
- uv
- Dependency resolution
- Pinned versions
- Lockfiles
- Transitive dependencies
- Reproducible environments

## 🔧 Commands

# Go to the project directory
cd /root/code/fraud-detection

# Check the existing requirements.in
cat requirements.in

# Correct the requirements.in file
cat > requirements.in <<'EOF'
scikit-learn
mlflow
pandas
numpy
EOF

# Verify the file
cat requirements.in

# Compile requirements.in into a pinned requirements.txt
uv pip compile requirements.in -o requirements.txt

# Check the generated lockfile
cat requirements.txt

# Verify the top-level packages are pinned
grep -E '^(scikit-learn|mlflow|pandas|numpy)==' requirements.txt

## 📝 Key Takeaway

requirements.in contains the high-level dependencies that the project directly needs.

uv resolves those dependencies and their transitive dependencies, then creates requirements.txt with exact versions using ==.

Example:

requirements.in:
numpy
pandas

        ↓ uv pip compile

requirements.txt:
numpy==2.x.x
pandas==2.x.x
...other transitive dependencies...

This makes the Python environment reproducible across different machines.
