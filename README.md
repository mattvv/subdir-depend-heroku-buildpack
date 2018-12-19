# subdir-depend-heroku-buildpack
Configure Heroku project by file listing allowing use of subfolders etc

This is based on https://github.com/timanovsky/subdir-heroku-buildpack

## What it does

This buildpack looks for two files inside a user specified folder:
1. MANIFEST_GLOBAL: This specifies which files and folders you would like to keep from the root directory
2. MANIFEST_MV_TO_ROOT: This specifies any files which need to be moved to the root directory such as a Procfile

## Instructions

1. Add MANIFEST_GLOBAL and MANIFEST_MV_TO_ROOT files to the sub directory containing your project. This sub directory is the `PROJECT_PATH`.
2. Add this buildpack as the first buildpack in the chain
3. Set `PROJECT_PATH` environment variable in Heroku to point to the corresponding sub directory

## How it works

The buildpack looks for the MANIFEST files in the subdirectory you configured, copies and moves the required files. Then normal Heroku slug compilation proceeds.

## Example

Consider a repository containing multiple Python projects with shared data and library code:

    - data/
        - data.csv
    - project1/
        - Procfile
        - requirements.txt
        - app.py
    - project2/
        - Procfile
        - requirements.txt
        - app.py
    - shared_library/
        - utils.py

You wish to deploy only project1 to Heroku. However it is a requirement that the projects Procfile is located in the root directory, while at the same time you have a shared library and dataset.

To deploy project1 to Heroku one would only need to add MANIFEST_GLOBAL and MANIFEST_MV_TO_ROOT to the project1 sub directory where MANIFEST_GLOBAL contains

	data/
	project1/
	shared_library/

and MANIFEST_MV_TO_ROOT contains

	project1/Procfile
	project1/requirements.txt

The directory structure should now look like

    - data/
        - data.csv
    - project1/
        - Procfile
        - requirements.txt
        - app.py
        - MANIFEST_GLOBAL
        - MANIFEST_MV_TO_ROOT
    - project2/
        - Procfile
        - requirements.txt
        - app.py
    - shared_library/
        - utils.py

Therefore the deployed build on Heroku will be simply

	- Procfile
	- requirements.txt
	- data/
	    - data.csv
	- project1/
	    - app.py
	    - MANIFEST_GLOBAL
	    - MANIFEST_MV_TO_ROOT
	- shared_library/
	    - utils.py

### Using gunicorn from a different folder

A standard Procfile for gunicorn will look like:

	web: gunicorn sankey_app:server

Since your app is being run from a sub directory you need to tell gunicorn to change to this sub directory first. Your procfile should look like

	web: gunicorn --chdir ./project1 sankey_app:server

