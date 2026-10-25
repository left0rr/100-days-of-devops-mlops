# MLOps Day 9 — Cookiecutter Templates & Jinja2

## 1. What is Cookiecutter?

Cookiecutter is a tool for generating new projects from reusable templates. Instead of manually creating the same files and folders for every ML project, you define a template once and Cookiecutter generates a customized project from it.

**How it works:**

1. Reads `cookiecutter.json` to get variable names, defaults, and choices.
2. Prompts for values, unless `--no-input` is used.
3. Uses Jinja2 to render template file contents and directory names.
4. Creates a new project populated with the selected values.

Example:

```bash
cookiecutter /root/code/mlops-template/
```

Generate a project without interactive prompts:

```bash
cookiecutter /root/code/mlops-template/ \
  --no-input \
  project_name=churn-model \
  author=xFusionCorp \
  python_version=3.11 \
  ml_framework=sklearn
```

## 2. Required template structure

```text
mlops-template/
├── cookiecutter.json
└── {{cookiecutter.project_name}}/
    ├── README.md
    ├── requirements.txt
    ├── data/
    ├── models/
    ├── src/
    └── tests/
```

**What each part does:**

- `cookiecutter.json` — defines template variables, defaults, and multiple-choice options.
- `{{cookiecutter.project_name}}/` — a dynamic directory name that becomes the selected project name.
- `README.md` — project documentation, rendered with the selected values.
- `requirements.txt` — Python dependencies, potentially selected conditionally by the framework.
- `data/` — datasets and data-processing inputs.
- `models/` — saved or trained model artifacts.
- `src/` — application and ML source code.
- `tests/` — automated tests.

Empty directories are often preserved in version control or templates using placeholder files such as `.gitkeep`.

## 3. Configure `cookiecutter.json`

Example:

```json
{
  "project_name": "my-ml-project",
  "author": "xFusionCorp",
  "python_version": "3.11",
  "ml_framework": ["sklearn", "pytorch", "tensorflow"]
}
```

Cookiecutter interprets ordinary values as defaults. A list defines choices for an interactive prompt.

For this template:
- `project_name` defaults to `my-ml-project`.
- `author` defaults to `xFusionCorp`.
- `python_version` defaults to `3.11`.
- `ml_framework` offers `sklearn`, `pytorch`, or `tensorflow`.

The values can be accessed in template files through the `cookiecutter` object.

## 4. Jinja2 template syntax

Cookiecutter uses **Jinja2**, a templating engine that replaces variables and evaluates conditional logic.

### Variables: `{{ ... }}`

Double curly braces output a value.

```jinja2
# {{ cookiecutter.project_name }}

Created by {{ cookiecutter.author }}.
```

If the project name is `churn-model` and the author is `xFusionCorp`, the generated README becomes:

```markdown
# churn-model

Created by xFusionCorp.
```

Variable names are **case-sensitive**. `cookiecutter.Author` is not the same as `cookiecutter.author`.

### Logic: `{% ... %}`

`{% ... %}` contains template instructions rather than text to print.

Example conditional dependencies:

```jinja2
{% if cookiecutter.ml_framework == "sklearn" %}
scikit-learn
{% elif cookiecutter.ml_framework == "pytorch" %}
torch
{% elif cookiecutter.ml_framework == "tensorflow" %}
tensorflow
{% endif %}
```

- `{% if condition %}` — tests the first condition.
- `{% elif condition %}` — tests another condition if earlier ones were false.
- `{% else %}` — optional fallback for all remaining cases.
- `{% endif %}` — closes the conditional block.
- `==` — compares two values for equality. A single `=` is not a comparison operator.

Only the matching dependency is written to the generated file.

**Remember the three main delimiters:**

| Syntax | Purpose |
|---|---|
| `{{ ... }}` | Output a variable or expression |
| `{% ... %}` | Execute logic, such as conditions or loops |
| `{# ... #}` | Add a template comment that is not rendered |

Jinja2 also renders template directory names, so `{{cookiecutter.project_name}}/` becomes `churn-model/`.

## 5. Debugging and verification

Inspect the template files:

```bash
cat cookiecutter.json
cat '{{cookiecutter.project_name}}/README.md'
cat '{{cookiecutter.project_name}}/requirements.txt'
```

Common errors:
- Incorrect variable capitalization.
- Using `=` instead of `==` in a condition.
- Missing `{% endif %}`.
- Missing required variables in `cookiecutter.json`.
- Incorrect template directory names.

Verify the generated project:

```bash
cat /root/code/churn-model/README.md
cat /root/code/churn-model/requirements.txt
find /root/code/churn-model -maxdepth 2 -print
```

Expected for the sklearn configuration:
- README mentions `churn-model` and `xFusionCorp`.
- Requirements contain `scikit-learn`.
- The project contains `data/`, `models/`, `src/`, and `tests/`.

## Key takeaway

Cookiecutter defines **what to generate**, `cookiecutter.json` defines **which values to use**, and Jinja2 defines **how those values change file contents and directory names**. This makes it easy to create consistent, repeatable project scaffolding for ML teams.