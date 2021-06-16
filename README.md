eBoxing application for LCBA
====

Light Contact Boxing

<a href="https://github.com/pydanny/cookiecutter-django">
    <img src="https://img.shields.io/badge/built%20with-Cookiecutter%20Django-ff69b4.svg" />
</a>


## Development

1. Install [Docker](https://docs.docker.com/install/) and [Docker-Compose](https://docs.docker.com/compose/).

2. set COMPOSE_FILE environment variable (or add `-f local.yml` to all docker-compose commands)

    `export COMPOSE_FILE=local.yml`

3. Build your virtual machines with the following shell command

    `docker-compose build`

4. Make the initial database migrations

    `docker-compose run --rm django python manage.py migrate`

5. Start your virtual machines with the following shell command

    `docker-compose up -d`

6. Check everything is fine for example

    `docker-compose ps`

    `docker-compose logs django`

7. Create an admin account with

    Important, you have to create an account in order to be able to load the data in the next step!

    `docker-compose exec django /entrypoint python manage.py createsuperuser`

     **Windows/git bash tip:**
      `docker-compose exec django /entrypoint`  works from command.exe or powershell but fails from git bash.
      As git bash tries to convert any absolute linux path to its windows install path ("C:/Program Files/Git/...)
     under git bash use ../entrypoint instead, also prefix docker-compose commands with winpty as below
      `winpty docker-compose exec django ../entrypoint python manage.py createsuperuser`

8. Load test anonymized lcba data with

    `docker-compose exec django /entrypoint python manage.py loaddata  lcba-anonymized.json`

9. Create/update the Haystack search index

    `docker-compose exec django /entrypoint python manage.py rebuild_index`

10. Check these out
 - Main Django eBoxing web site at: http://localhost:8000/
 - Django admin at: http://localhost:8000/admin
 - eBoxing API at : http://localhost:8000/api/

 - Access local apps using traefik reverse proxy

    Add the following lines to your `/etc/hosts` or `\Windows\system32\etc\drivers\hosts`

        127.0.0.1       traefik.lcba
        127.0.0.1       eboxing.lcba

    (Windows only) To give traefik access to docker API:

        check "Expose daemon on tcp://localhost:2375 without TLS" in docker settings

    then use:
    - Django eBoxing at http://eboxing.lcba/
    - traefik dashboard at http://traefik.lcba/


### Tests

1. Run all tests

    `docker-compose exec django /entrypoint coverage run -m pytest backend`

2. Generate coverage report

    `docker-compose exec django /entrypoint coverage report`

- You can also run tests for a specific app

    `docker-compose exec django /entrypoint coverage run -m pytest backend/apps/licences`

### Style Guide Enforcement

1. Run style check

    `docker-compose exec django flake8`

- More detailed output with pylint

    `docker-compose exec django pylint --load-plugins pylint_django /app/backend`

- If you have issues with the order of the imports, you can use *isort* to reorder them

    `docker-compose -f local.yml exec django isort backend`

### Import data from old production site

1. Starting with a fresh JSON dump file created with eboxing_data

    `git checkout eae03666a6d339257c693597f00ca790034d2e2f`

    `export COMPOSE_FILE=local.yml`

    `docker-compose build`

    Adapt `backend/apps/eboxing/fixtures/patch-lcba.sh`:

    ```
    #!/bin/bash
    # adapt fixture from eboxing_data to model app moves
    sed  -i.bak  -e 's/eboxing_data.licence/licences.licence/g' ./lcba-anonymized.json
    sed  -i.bak  -E 's/eboxing_data.(club|user|member|achievement|achievementtype|group|groupmember)/members.\1/g' ./lcba-anonymized.json
    sed  -i.bak  -E 's/eboxing_data.(tournament|tournamentcategory|tournamentregistration|assignment|assignmenttask)/tournaments.\1/g' ./lcba-anonymized.json
    sed  -i.bak  -E 's/eboxing_data.(result|trophyrule|weightclasstype|weightclass|match)/matches.\1/g' ./lcba-anonymized.json
    sed  -i.bak  -E 's/eboxing_data.(document|documentcategory)/documents.\1/g' ./lcba-anonymized.json
    sed  -i.bak  -E 's/eboxing_data.(event|eventregistration)/events.\1/g' ./lcba-anonymized.json
    sed  -i.bak  -e 's/eboxing_data/eboxing/g' ./lcba-anonymized.json
    sed  -i.bak  -e 's/adminlc/1/g' ./lcba-anonymized.json
    ```

    `docker-compose exec django /entrypoint python manage.py loaddata --app eboxing lcba-anonymized.json`

2. Change to current version

    `git checkout master`

    Edit .envs/.local/.postgres to reuse username and passwords from https://gitlab.com/lcba/eboxing/-/blob/eae03666a6d339257c693597f00ca790034d2e2f/.envs/.local/.postgres

    `docker-compose build`

    `docker-compose up django`

### Genereated graph from postgres database

`docker-compose -f local.yml exec django /entrypoint python manage.py graph_models -a -o lcba.png`


"# VerificationEboxing" 
