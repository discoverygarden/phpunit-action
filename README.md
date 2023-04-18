# phpunit-action
Github action to launch PHPUnit tests configured for a module.

## Inputs
-  **composer-auth:** Composer auth token necessary for accessing our composer project. (Required)
-  **composer_package:** Composer project reference. (optional)(Default: The repository fullname. ex. 'discoverygarden/phpunit-action')
-  **drupal_extension:** The module name for enabling within drupal. (optional)(Default: Repository name. ex. 'phpunit-action')
-  **composer_patches:** Additional composer patches that might be necessary to run the workflow. (optional)
-- Ex. ```yaml
composer_patches: |-
{
  "drupal/core": {
    "Postgres Driver":"https://www.drupal.org/files/issues/2022-01-11/2920527-26.patch"
  }
}```
-  **composer_package_prerequisites:** require-dev modules required for the module that might not otherwise get installed. (optional)
-- Ex. ```yaml
composer_package_prerequisites: >-
  "discoverygarden/relator_creator_processor:^1"```
-  **drupal_root:** The root drupal directory. (optional)(Default: /opt/drupal)
-  **drupal_web_root:** The drupal web root directory. (Default: /opt/drupal/web)
-  **postgres_db:** The postgres database name. (Default: drupal)
-  **postgres_user:** The postgres database user. (Default: drupal)
-  **postgres_password:** The postgress database user password. (Default: drupal)
-  **apache_user:** The apache user. (Default: www-data)
-  **apache_user_group:** The apache user group. (Default: www-data)

## Output
This action does not output any artifacts.

## Secrets
This action specifically uses the composer-auth organization token in order to have read access to our composer projects:
- PRIVATE_PACKAGIST_AUTH_ACCESS

## Usage
Learn more about GitHub Actions in general [here](https://docs.github.com/en/actions/quickstart). 

To use this action in your repo, follow these steps:

 1. Create a YAML file in the `.github/workflows/` directory of your repo labeled 'phpunit.yml'
 2. Copy the following into the YAML file:
```yaml
name: PHPUnit Tests
on:
  pull_request:
    branches:
      - main
jobs:
  PHPUnit:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        drupal-version: ['9.4']
        php-version: ['7.4', '8.0']
        experimental: [false]
        include:
          - drupal-version: '9'
            php-version: '7.4'
            experimental: true
          - drupal-version: '9'
            php-version: '8.1'
            experimental: true
          - drupal-version: '9.4'
            php-version: '8.1'
            experimental: true
    continue-on-error: ${{ matrix.experimental }}
    container:
      image: drupal:${{ matrix.drupal-version }}-php${{ matrix.php-version }}
      ports:
        - 80
    services:
      postgres:
        image: postgres
        env:
          POSTGRES_DB: drupal
          POSTGRES_USER: drupal
          POSTGRES_PASSWORD: drupal
        # Set health checks to wait until postgres has started
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - name: Run PHPUnit Action
        uses: discoverygarden/phpunit-action@implementation
        with:
          composer-auth: ${{ secrets.PRIVATE_PACKAGIST_AUTH_ACCESS }}
```

This action will run the module defined tests against the repository when there is a pull request against the defined branches (Default: `main` branch).

This example is running this action in a matrix strategy of various drupal-project and php version configurations for additional operational security.
