This repo is a companion to the [blogpost][1].

It was created using `snow init  --template example_streamlit` and can be
deployed either via `snow streamlit deploy` (provided `snowflake.yml` is edited)
or via `GIT REPOSITORY` integration.

# `snow`  deployment instructions

1. `git clone` this repo locally
2. Set the warehouse in `./snowflake.yml`
3. Run `snow streamlit deploy`

# `GIT REPOSITORY` deployment instructions

This sequence of SQL statements will:

- Create a role that will own the `STREAMLIT` object (needed because of [owner's rights][2])
- Create a dedicated database
- Create a dedicated warehouse
- Create an API integration
- Create a `GIT REPOSITORY` object
- Create a `STREAMLIT` object from the `GIT REPOSITORY`
- Share the `STREAMLIT` object with a role (`SYSADMIN`)

The complete sequence of steps is not strictly necessary, but this procedure can
be used to isolate applications using RBAC.

```sql
-- Optional: set up dedicated role to own the Streamlit app
USE ROLE useradmin;
CREATE ROLE IF NOT EXISTS sample_streamlit_owner_role;
GRANT ROLE sample_streamlit_owner_role TO ROLE sysadmin;
-- End of role setup

-- Optional: database setup
USE ROLE sysadmin;
CREATE DATABASE IF NOT EXISTS sample_streamlit_db;
-- End of database setup

-- Optional: if using a custom warehouse
CREATE WAREHOUSE IF NOT EXISTS sample_streamlit_wh WITH
    WAREHOUSE_SIZE = XSMALL
    INITIALLY_SUSPENDED = TRUE
;
GRANT USAGE ON WAREHOUSE sample_streamlit_wh to ROLE sample_streamlit_owner_role;
-- End of warehouse setup

USE ROLE accountadmin;
CREATE API INTEGRATION IF NOT EXISTS gh_sample_blogpost
    API_PROVIDER = GIT_HTTPS_API
    API_ALLOWED_PREFIXES = ('https://github.com/sfc-gh-vtimofeenko/blogpost-streamlit-in-snowflake-example')
    ENABLED = TRUE;

USE ROLE sysadmin;
CREATE GIT REPOSITORY IF NOT EXISTS sample_streamlit_db.public.sample_repo
    API_INTEGRATION = gh_sample_blogpost
    ORIGIN = 'https://github.com/sfc-gh-vtimofeenko/blogpost-streamlit-in-snowflake-example';

-- Optional, if using custom role
GRANT USAGE ON DATABASE sample_streamlit_db TO ROLE sample_streamlit_owner_role;
GRANT USAGE ON SCHEMA sample_streamlit_db.public TO ROLE sample_streamlit_owner_role;
GRANT READ ON GIT REPOSITORY sample_streamlit_db.public.sample_repo TO ROLE sample_streamlit_owner_role;
GRANT CREATE STREAMLIT ON SCHEMA sample_streamlit_db.public TO ROLE sample_streamlit_owner_role;
USE ROLE sample_streamlit_owner_role;
--

CREATE STREAMLIT IF NOT EXISTS sample_streamlit_db.public.sample_git_deployed_streamlit
    ROOT_LOCATION = '@sample_streamlit_db.public.sample_repo/branches/main'
    MAIN_FILE = '/streamlit_app.py'
    QUERY_WAREHOUSE = sample_streamlit_wh; -- Replace the warehouse if needed

-- Share the streamlit app with needed roles
GRANT USAGE ON STREAMLIT sample_streamlit_db.public.sample_git_deployed_streamlit TO ROLE SYSADMIN;
```
[1]: TODO
[2]: https://docs.snowflake.com/en/developer-guide/streamlit/owners-rights
