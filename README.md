This repo is a companion to the [blogpost][1].

It was created using `snow init  --template example_streamlit` and can be
deployed either via `snow streamlit deploy` (provided `snowflake.yml` is edited)
or via `GIT REPOSITORY` integration.

# `snow`  deployment instructions

1. `git clone` this repo locally
2. Set the warehouse in `./snowflake.yml`
3. Run `snow streamlit deploy`

# `GIT REPOSITORY` deployment instructions

As a single sequence of SQL statements:

```
USE ROLE ACCOUNTADMIN; -- or a different role with CREATE API INTEGRATION privilege

CREATE OR REPLACE API INTEGRATION sample_git_integration
    API_PROVIDER = GIT_HTTPS_API
    API_ALLOWED_PREFIXES = ('https://github.com/<gitHubOrgName>')
    ENABLED = TRUE;

CREATE OR REPLACE GIT REPOSITORY <path.to.gitRepository>
    API_INTEGRATION = sample_git_integration
    ORIGIN = 'https://github.com/<gitHubOrgName>/<repoName>';

CREATE OR REPLACE STREAMLIT <path.to.Streamlit.app>
    ROOT_LOCATION = '@<path.to.gitRepository>/branches/main'
    MAIN_FILE = '/streamlit_app.py'; -- as an example
```
[1]: TODO
