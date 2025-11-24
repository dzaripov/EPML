# EPML

Engineering Practices in Machine Learning


Установим (на arch) pipx и с помощью cookiecutter создадим проект и выберем в качестве пакетного менеджера uv, а линтер - ruff

 ```bash
 yay pipx
 pipx install cookiecutter-data-science
 ccds
 ```

Установим питон и создадим venv
```bash
uv sync
source .venv/bin/activate
```

Устанавливаем системный ruff, ty (type checker от astral) и bandit через uv:
```bash
uv tool install ruff
uv tool install ty
uv tool install bandit
uvx ruff check # linter через ruff
uvx ruff format # форматирование через ruff (заменяет и black, и isort)
uvx ty check # чекер типов - замена mypy
uv run pytest # тесты
uvx bandit -r epml
```

Дальше можно его установить в VS Code как расширение и настроить конфиг (`settings.json`), мой:
```json
"[python]": {

"editor.defaultFormatter": "charliermarsh.ruff",
"editor.formatOnSave": true,
"editor.codeActionsOnSave": {
"source.fixAll": "explicit",
"source.unusedImports": "explicit",
	}
},
"python.analysis.inlayHints.variableTypes": true,
"python.analysis.inlayHints.functionReturnTypes": true,
"python.analysis.typeCheckingMode": "off",
```

Добавим pre-commit hook

```bash
uv add --dev pre-commit
```

```
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v6.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-toml
      - id: check-added-large-files

  - repo: https://github.com/astral-sh/uv-pre-commit
    rev: 0.9.11
    hooks:
      - id: uv-lock

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.14.6
    hooks:
      - id: ruff
        args: [ --fix ]
      - id: ruff-format

  - repo: https://github.com/PyCQA/bandit
    rev: 1.9.2
    hooks:
      - id: bandit
        args: ["-c", "pyproject.toml"]
        additional_dependencies: ["bandit[toml]"]

  - repo: local
    hooks:
      - id: ty
        name: ty
        entry: uv run ty
        language: system
        types: [python]
        pass_filenames: true
```

и изменим pyproject.toml

```
# --- RUFF CONFIG ---
[tool.ruff]
line-length = 88
target-version = "py310"

[tool.ruff.lint]
select = [
    "E",  # pycodestyle errors
    "W",  # pycodestyle warnings
    "F",  # pyflakes
    "I",  # isort (сортировка импортов)
    "B",  # flake8-bugbear
    "UP", # pyupgrade (обновление синтаксиса)
]
ignore = []

# --- BANDIT CONFIG ---
[tool.bandit]
exclude_dirs = ["tests", ".venv"]
tests = ["B201", "B301"]
skips = ["B101"] # Игнорируем assert
```

Создадим репозиторий для теста прекоммитов

```
git init -b main
git remote add origin https://github.com/dzaripov/EPML_ITMO.git
git checkout -b hw1
git add .
```

Запуск pre-commit:

```bash
uv run pre-commit install
uv run pre-commit autoupdate # возможно понадобится чтобы обновить версии
uv run pre-commit run --all-files
```

Получаем положительный результат:
```
trim trailing whitespace...Passed
fix end of files...Passed
check yaml...Passed
check toml...Passed
check for added large files...Passed
uv-lock...Passed
ruff (legacy alias)...Passed
ruff format...Passed
bandit...Passed
ty...Passed
```


Создадим Dockerfile и .dockerignore:
```
#Dockerfile

FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
PYTHONUNBUFFERED=1

WORKDIR /app

COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-cache
ENV PATH="/app/.venv/bin:$PATH"
COPY . .

CMD ["python", "epml/modeling/train.py"]
```

```
#.dockerignore

.git
.gitignore
.venv
__pycache__
.pytest_cache
.ruff_cache
data
notebooks
tests
```

Закоммитим
```
git commit -m "init commit with proper env"
git push -u origin hw1
```

## Project Organization

```
├── LICENSE            <- Open-source license if one is chosen
├── Makefile           <- Makefile with convenience commands like `make data` or `make train`
├── README.md          <- The top-level README for developers using this project.
├── data
│   ├── external       <- Data from third party sources.
│   ├── interim        <- Intermediate data that has been transformed.
│   ├── processed      <- The final, canonical data sets for modeling.
│   └── raw            <- The original, immutable data dump.
│
├── docs               <- A default mkdocs project; see www.mkdocs.org for details
│
├── models             <- Trained and serialized models, model predictions, or model summaries
│
├── notebooks          <- Jupyter notebooks. Naming convention is a number (for ordering),
│                         the creator's initials, and a short `-` delimited description, e.g.
│                         `1.0-jqp-initial-data-exploration`.
│
├── pyproject.toml     <- Project configuration file with package metadata for
│                         epml and configuration for tools like black
│
├── references         <- Data dictionaries, manuals, and all other explanatory materials.
│
├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
│   └── figures        <- Generated graphics and figures to be used in reporting
│
├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
│                         generated with `pip freeze > requirements.txt`
│
├── setup.cfg          <- Configuration file for flake8
│
└── epml   <- Source code for use in this project.
    │
    ├── __init__.py             <- Makes epml a Python module
    │
    ├── config.py               <- Store useful variables and configuration
    │
    ├── dataset.py              <- Scripts to download or generate data
    │
    ├── features.py             <- Code to create features for modeling
    │
    ├── modeling
    │   ├── __init__.py
    │   ├── predict.py          <- Code to run model inference with trained models
    │   └── train.py            <- Code to train models
    │
    └── plots.py                <- Code to create visualizations
```

--------
