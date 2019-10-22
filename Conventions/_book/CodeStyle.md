# Configuration Requirements

To make sure everyone can work together reasonably well. We're going to implement automated code style linting, ans static analysis.

## Laravel Code Style

You **MUST** install these packages

Packages:
* [BrainMaestro/composer-git-hooks](https://github.com/BrainMaestro/composer-git-hooks)
* [matt-allan/laravel-code-style](https://github.com/matt-allan/laravel-code-style)

```shell script
composer install --dev brainmaestro/composer-git-hooks matt-allan/laravel-code-style
```

### Configuration - laravel-code-style

```shell
php artisan vendor:publish --provider="MattAllan\LaravelCodeStyle\ServiceProvider"
```

Publishing the config will add a `.php_cs` configuration file to the root of your project.  You may customize this file as needed.  The `.php_cs` file should be committed to version control.

A cache file will be written to `.php_cs.cache` in the project root the first time you run the fixer.  You should ignore this file so it is not added to your version control system.

```shell
echo '.php_cs.cache' >> .gitignore
```

### Configuration - composer.json

Add to your  `composer.json`

```json
    {
      "extra": {
        "hooks": {
          "pre-commit": [
            "vendor/bin/php-cs-fixer fix --dry-run --diff"
          ],
          "commit-msg": "grep -q '[A-Z]+-[0-9]+.*' $1",
          "pre-push": [
                "vendor/bin/php-cs-fixer fix --dry-run .",
                "phpunit"
          ],
          "post-merge": "composer install"
        }
      },
      "scripts": {
        "check-style": "php-cs-fixer fix --dry-run --diff",
        "fix-style": "php-cs-fixer fix"
      }
    }
```
