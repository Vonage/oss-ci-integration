# WhiteSource-CI-Integration

Github action for CI integration.

How to use the action in an organization:

1. Create a repository with all your config files in the following format:
   repoName/ProductName/ProjectName/wss-generated-file-CI.config
   In the config file change the API key to "<API_TOKEN>"
2. Define the following secrets:

- WHITESOURCE_GH_PAT: GH private access token (enable SSO if needed, defined in the organization level)
- WHITESOURCE_CONFIG_REPO: The path to your config files repository in the following format organization/repoName.git (defined in the organization level)
- WHITESOURCE_PRODUCT_NAME: The product name from the WS UI (define this variable in the repository level to support many products if needed)
- WHITESOURCE_API_KEY: Your organization WS API token (defined in the organization level)
- WHITESOURCE_NPM_TOKEN: NPM token to access private organizations if needed (defined in the organization level)
- WHITESOURCE_USER_KEY: Your WS user key

3. Onboarding new repository:

   Add the relevant GH action to the repository in the following path: .github/workflows/whitesourceAction.yml

a. Node projects:

1. Copy the [whitesourceAction.yml](./whitesourceAction.yml) file to the directory `.github/workflows/` in your repository.
2. Notice the TODOs in the file, and replace them with the appropriate values.

b. Other:

```yaml

name: WhiteSource CI integration
on:
pull_request:
branches: [ develop, release, master, main ]
schedule:
  - cron: '0 0 * * 0'
jobs:
build:
runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
      - name: Extract repository name
        run: echo "repository_name=$(echo ${{ github.repository }} | cut -d'/' -f2)" >> $GITHUB_ENV
        shell: bash
      - name: WhiteSource CI integration
        uses: Vonage/oss-ci-integration@v1.0
        env:
          WHITESOURCE_PRODUCT_NAME: ${{ secrets.WHITESOURCE_PRODUCT_NAME }}
          WHITESOURCE_PROJECT_NAME: ${{ env.repository_name }}
          WHITESOURCE_GH_PAT: ${{ secrets.WHITESOURCE_GH_PAT }}
          WHITESOURCE_CONFIG_REPO: ${{ secrets.WHITESOURCE_CONFIG_REPO }}
          WHITESOURCE_NPM_TOKEN: ${{ secrets.WHITESOURCE_NPM_TOKEN }}
          WHITESOURCE_API_KEY: ${{ secrets.WHITESOURCE_API_KEY }}
          WHITESOURCE_USER_KEY: ${{ secrets.WHITESOURCE_USER_KEY }}

      - name: policy Rejection Summary
        if: ${{ always() }}
        run: cat ./whitesource/policyRejectionSummary.json

```

c. Python projects: change the Python version if needed, and install required libraries in the 'Install dependencies' step

```yaml

# This is a basic workflow to help you get started with Actions

name: WhiteSource CI integration

# Controls when the action will run. Triggers the workflow on push or pull request

# events but only for the master branch

on:
pull_request:
branches: [ develop, release, master, main ]
schedule: - cron: '0 0 \* \* 0'

# A workflow run is made up of one or more jobs that can run sequentially or in parallel

jobs:

# This workflow contains a single job called "build"

build: # The type of runner that the job will run on
runs-on: ubuntu-latest
strategy:
matrix:
python-version: [3.6]

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Install python requierments
      - uses: actions/checkout@v2
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
           python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          pip install virtualenv pipenv

      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
        with:
          ref: ${{ github.ref }}
      - name: Extract repository name
        run: echo "repository_name=$(echo ${{ github.repository }} | cut -d'/' -f2)" >> $GITHUB_ENV
        shell: bash
      - name: WhiteSource CI integration
        uses: Vonage/oss-ci-integration@v1.0
        env:
          WHITESOURCE_PRODUCT_NAME: ${{ secrets.WHITESOURCE_PRODUCT_NAME }}
          WHITESOURCE_PROJECT_NAME: ${{ env.repository_name }}
          WHITESOURCE_GH_PAT: ${{ secrets.WHITESOURCE_GH_PAT }}
          WHITESOURCE_CONFIG_REPO: ${{ secrets.WHITESOURCE_CONFIG_REPO }}
          WHITESOURCE_NPM_TOKEN: ${{ secrets.WHITESOURCE_NPM_TOKEN }}
          WHITESOURCE_API_KEY: ${{ secrets.WHITESOURCE_API_KEY }}
          WHITESOURCE_USER_KEY: ${{ secrets.WHITESOURCE_USER_KEY }}

      - name: policy Rejection Summary
        if: ${{ always() }}
        run: cat ./whitesource/policyRejectionSummary.json

```
