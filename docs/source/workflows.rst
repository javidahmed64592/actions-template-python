Workflows
=========

This document details the CI/CD workflows and reusable actions to build and release Python applications.
They run automated code quality checks to ensure code remains robust, maintainable, and testable.

The following actions can be referenced from other repositories using ``javidahmed64592/|repository_name|/actions/{category}/{action}@main``.

----

Setup Actions
~~~~~~~~~~~~~

**setup-uv-python**

- **Description:** Sets up Python with uv.
- **Location:** ``setup-uv-python/action.yml``
- **Steps:**

  - Installs uv with caching enabled
  - Sets up Python using the version specified in ``.python-version``
  - Caches dependencies based on ``uv.lock`` for faster builds

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/setup/setup-uv-python@main

----

**install-python-core**

- **Description:** Installs core Python dependencies from pyproject.toml using uv.
- **Location:** ``install-python-core/action.yml``
- **Steps:**

  - Uses the ``setup-uv-python`` action
  - Runs ``uv sync`` to install only core dependencies

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/setup/install-python-core@main

----

**install-python-dev**

- **Description:** Installs dev Python dependencies from pyproject.toml using uv.
- **Location:** ``install-python-dev/action.yml``
- **Steps:**

  - Uses the ``setup-uv-python`` action
  - Runs ``uv sync --extra dev`` to install core and dev dependencies

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/setup/install-python-dev@main

----

**install-python-docs**

- **Description:** Installs documentation Python dependencies from pyproject.toml using uv.
- **Location:** ``install-python-docs/action.yml``
- **Steps:**

  - Uses the ``setup-uv-python`` action
  - Runs ``uv sync --extra docs`` to install core and docs dependencies

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/setup/install-python-docs@main

----

**check-frontend-exists**

- **Description:** Check if the frontend directory exists in the repository to conditionally execute frontend CI and build jobs.
- **Location:** ``check-frontend-exists/action.yml``
- **Outputs:**

  - ``exists``: Indicates whether the frontend directory exists

- **Steps:**

  - Checks for directory named ``<repository name>-frontend``
  - Sets output to ``true`` or ``false`` based on directory existence
  - Logs message if directory doesn't exist

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/setup/check-frontend-exists@main
       id: check-frontend

----

**setup-node**

- **Description:** Set up Node.js with npm cache and install dependencies.
- **Location:** ``setup-node/action.yml``
- **Steps:**

  - Uses the ``setup-node`` action
  - Install ``npm`` packages

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/setup/setup-node@main

----

CI Actions
~~~~~~~~~~

**validate-pyproject**

- **Description:** Validate pyproject.toml structure.
- **Location:** ``validate-pyproject/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Runs ``uv run validate-pyproject pyproject.toml`` to validate TOML structure

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/validate-pyproject@main

----

**ruff**

- **Description:** Run Ruff linting checks on the codebase.
- **Location:** ``ruff/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Runs ``uv run -m ruff check`` to lint the code

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/ruff@main

----

**mypy**

- **Description:** Run Mypy type checking on the codebase.
- **Location:** ``mypy/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Runs ``uv run -m mypy .`` to perform static type checking

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/mypy@main

----

**pytest**

- **Description:** Run Pytest tests with coverage reporting.
- **Location:** ``pytest/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Runs ``uv run -m pytest --cov-report html --cov-report term`` to execute tests with coverage
  - Uploads HTML coverage report as artifact named ``backend-coverage-report``
  - Fails if coverage drops below the threshold configured in ``pyproject.toml``

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/pytest@main

----

**bandit**

- **Description:** Run Bandit security checks on the codebase.
- **Location:** ``bandit/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Runs ``uv run bandit -r $PACKAGE_NAME -f json -o bandit-report.json`` to scan for security vulnerabilities
  - Uploads JSON report as artifact named ``bandit-report``

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/bandit@main

----

**pip-audit**

- **Description:** Run pip-audit to check for known vulnerabilities in dependencies.
- **Location:** ``pip-audit/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Runs ``uv run pip-audit --desc`` to check dependencies for known CVEs

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/pip-audit@main

----

**version-check**

- **Description:** Check version consistency across pyproject.toml and uv.lock.
- **Location:** ``version-check/action.yml``
- **Steps:**

  - Uses the ``install-python-dev`` action
  - Extracts version from ``pyproject.toml`` using ``uv run ci-pyproject-version``
  - Verifies ``uv.lock`` version matches using ``uv run ci-uv-lock-version``
  - Optionally checks additional version files via ``additional-versions`` input
  - Fails if any version mismatch is detected

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/version-check@main

**Advanced usage with additional version files:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/ci/version-check@main
       with:
         additional-versions: '[{"name": "package.json", "version": "1.2.3"}]'

----

**frontend**

- **Description:** Runs frontend validation checks (type-checking, linting, formatting, testing) and uploads coverage reports.
- **Location:** ``frontend/action.yml``
- **Steps:**

  - Uses the ``setup-node`` action
  - Runs ``npm run type-check`` for type checking
  - Runs ``npm run lint`` for static code analysis
  - Runs ``npm run format`` to tidy up scripts
  - Runs ``npm run test:coverage`` for frontend unit tests
  - Uploads unit test coverage report

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/ci/frontend@main

----

Build Actions
~~~~~~~~~~~~~

**build-wheel**

- **Description:** Build Python wheel package and upload as artifact.
- **Location:** ``build-wheel/action.yml``
- **Steps:**

  - Uses the ``install-python-core`` action
  - Runs ``uv build`` to create the wheel
  - Inspects wheel contents using ``unzip -l``
  - Uploads wheel as artifact with name ``{PACKAGE_NAME}_wheel``

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/build/build-wheel@main

----

**verify-structure**

- **Description:** Download and verify the structure of the built wheel package.
- **Location:** ``verify-structure/action.yml``
- **Steps:**

  - Uses the ``install-python-core`` action
  - Downloads the wheel artifact (named ``{PACKAGE_NAME}_wheel``)
  - Installs the wheel using ``uv pip install``
  - Verifies that ``site-packages`` and the package directory exist
  - Optionally verifies additional directories specified in inputs
  - Fails if any required directory is missing

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/build/verify-structure@main

**Advanced usage with additional checks:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/build/verify-structure@main
       with:
         expected-directories: |
           static

----

**build-frontend**

- **Description:** Build the frontend and upload the build artifacts for use in other jobs.
- **Location:** ``build-frontend/action.yml``
- **Steps:**

  - Uses the ``setup-node`` action
  - Runs ``npm run build`` to build frontend static files
  - Uploads static files as build artifact

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/build/build-frontend@main

----

Docs Actions
~~~~~~~~~~~~

**build-docs**

- **Description:** Build Sphinx documentation and upload as artifact.
- **Location:** ``build-docs/action.yml``
- **Steps:**

  - Uses the ``install-python-docs`` action
  - Runs ``uv run sphinx-build -M clean docs/source/ docs/build/`` to clean previous builds
  - Runs ``uv run sphinx-build -M html docs/source/ docs/build/`` to build HTML documentation
  - Uploads built documentation as artifact named ``documentation``

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/|repository_name|/actions/docs/build-docs@main

----

**publish-docs**

- **Description:** Deploy Sphinx documentation to GitHub Pages.
- **Location:** ``publish-docs/action.yml``
- **Outputs:**

  - ``page_url``: URL of the deployed GitHub Pages site

- **Steps:**

  - Downloads the ``documentation`` artifact
  - Uploads HTML files to GitHub Pages
  - Deploys to GitHub Pages
  - Returns the deployed page URL as output

**Usage:**

.. code-block:: yaml

   steps:
     - id: publish
       uses: javidahmed64592/|repository_name|/actions/docs/publish-docs@main
     - run: echo "Deployed to ${{ steps.publish.outputs.page_url }}"

----

Docker Actions
~~~~~~~~~~~~~~

**build-start-services**

- **Description:** Creates ``.env`` file from ``.env.example`` and starts Docker Compose services.
- **Location:** ``build-start-services/action.yml``
- **Steps:**

  - Moves ``.env.example`` to ``.env``
  - Runs ``docker compose [build-args] up --build -d``
  - Sleeps for ``wait-seconds`` to allow services to start

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/build-start-services@main
       with:
         wait-seconds: "5"
         build-args: ""

----

**show-logs**

- **Description:** Shows logs from a Docker Compose container.
- **Location:** ``show-logs/action.yml``
- **Steps:**

  - Displays logs using ``docker compose logs`` for the repository name

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/show-logs@main

----

**check-containers**

- **Description:** Checks the server health by polling the `/api/health` endpoint.
- **Location:** ``check-containers/action.yml``
- **Steps:**

  - Polls the server's health endpoint at regular intervals until it returns a successful response or times out

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/check-containers@main
       with:
         port: 443
         num-retries: "5"
         timeout-seconds: "5"

----

**stop-services**

- **Description:** Stops Docker Compose services and removes volumes and orphans.
- **Location:** ``stop-services/action.yml``
- **Steps:**

  - Stops Docker Compose services and removes volumes and orphans

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/stop-services@main

----

**prepare-release**

- **Description:** Gets the package version, creates a release directory with essential files, builds a tarball, and uploads it as a workflow artifact.
- **Location:** ``prepare-release/action.yml``
- **Steps:**

  - Extracts version via ``uv run ci-pyproject-version``
  - Creates release directory and copies ``docker-compose.yml``, ``README.md``, ``LICENSE``, ``.env.example``
  - Creates compressed tarball
  - Uploads tarball as artifact named ``{PACKAGE_NAME}_release``

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/prepare-release@main

----

**check-repo-name**

- **Description:** Compares the repository name against the package name in ``pyproject.toml`` to prevent template-derived repositories from accidentally publishing releases.
- **Location:** ``check-repo-name/action.yml``
- **Outputs:**

  - ``should_release`` - ``"true"`` if names match, ``"false"`` otherwise
  - ``package_name`` - Package name extracted from ``pyproject.toml``

- **Steps:**

  - Extracts package name via ``uv run ci-pyproject-name``
  - Compares repository name with package name
  - Sets outputs based on match result and package name

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/check-repo-name@main
       id: check_repo_name

----

**get-version**

- **Description:** Extracts the version and tag from ``pyproject.toml``.
- **Location:** ``get-version/action.yml``
- **Outputs:**

  - ``version`` - Version string (e.g. ``1.2.3``)
  - ``tag`` - Version tag (e.g. ``v1.2.3``)

- **Steps:**

  - Extracts version via ``uv run ci-pyproject-version``

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/get-version@main
       id: get_version

----

**check-tag**

- **Description:** Checks if a Git tag already exists for the given version to avoid duplicate releases.
- **Location:** ``check-tag/action.yml``
- **Outputs:**

  - ``tag_exists`` - ``"true"`` if the tag exists, ``"false"`` otherwise

- **Steps:**

  - Checks if tag exists using GitHub API

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/check-tag@main
       id: check_tag
       with:
         tag: ${{ steps.get_version.outputs.tag }}

----

**generate-release-notes**

- **Description:** Substitutes placeholders in ``RELEASE-NOTES.md`` with actual values.
- **Location:** ``generate-release-notes/action.yml``
- **Inputs:**

  - ``version`` - Release version (without the v prefix)
  - ``package_name`` - Package name from ``pyproject.toml``

- **Steps:**

  - Substitutes ``{{VERSION}}``, ``{{PACKAGE_NAME}}``, and ``{{REPOSITORY}}`` placeholders
  - Repository is derived from GitHub context

**Usage:**

.. code-block:: yaml

   steps:
     - uses: javidahmed64592/python-template-server/.github/actions/docker/generate-release-notes@main
       with:
         version: ${{ steps.get_version.outputs.version }}
         package_name: ${{ steps.check_repo_name.outputs.package_name }}
