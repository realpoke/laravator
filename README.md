# Laravator multi-server deployment Workflow for Laravel Application

This repository contains a GitHub Actions workflow for deploying a Laravel application to multiple servers with zero downtime. The deployment process includes compiling assets, creating deployment artifacts, uploading them to the servers, and activating the new release without affecting the current users.

## Prerequisites

### LEMP Stack

Ensure your server is set up with the following components:

- **Linux**: Ubuntu 24.04 is recommended.
- **Nginx**: Web server to serve your Laravel application.
- **MySQL/MariaDB**: Database server.
- **PHP**: Version 8.3 with required extensions (mbstring, ctype, fileinfo, openssl, pdo, bcmath, json, tokenizer, xml).

### Required PHP Extensions

Make sure the following PHP extensions are installed:
- mbstring
- ctype
- fileinfo
- openssl
- pdo
- bcmath
- json
- tokenizer
- xml

## GitHub Actions Workflow

The workflow leverages several GitHub Actions, including those from [appleboy](https://github.com/appleboy):

- `appleboy/scp-action`: Securely copy files to your server.
- `appleboy/ssh-action`: Execute commands on your server over SSH.

### Workflow Steps

1. **Checkout Code**: Checks out the code from the repository.
2. **Setup Node**: Sets up Node.js for asset compilation.
3. **Compile CSS and Javascript**: Installs dependencies and compiles the assets using `yarn`.
4. **Setup PHP**: Configures PHP 8.3 with the necessary extensions.
5. **Install Composer Dependencies**: Installs PHP dependencies.
6. **Create Deployment Artifacts**: Creates a tarball of the application for deployment.
7. **Store Artifacts**: Uploads the deployment artifacts to GitHub Actions.
8. **Export Deployment Matrix**: Exports the deployment configuration for multiple servers.
9. **Prepare Release on Server**: Uploads and extracts the deployment artifacts on the server.
10. **Run Before Hooks**: Executes pre-defined hooks before activating the release.
11. **Activate Release**: Activates the new release by updating symbolic links.
12. **Run After Hooks**: Executes post-deployment hooks.
13. **Clean Up**: Removes old releases and artifacts from the server.

## Before Hooks

**Purpose**: The commands or scripts in the before hooks are executed before the new release is activated.

**Common Use Cases**:
- **Database migrations**: Applying schema changes before the new code is live ensures the application will have the required database structure when it starts running.
- **Stopping services or background jobs**: Temporarily stopping services that might interfere with the deployment.
- **Cache clearing**: Ensuring that any old cached data is cleared before the new release is used.

## After Hooks

**Purpose**: The commands or scripts in the after hooks are executed after the new release is activated.

**Common Use Cases**:
- **Restarting services or background jobs**: Ensuring that the services are restarted and are using the new code.
- **Cache warming**: Pre-populating caches with the new code to improve performance.
- **Notification**: Sending notifications about the successful deployment.


## Example `deployment-config.json`

Here is an example of what your `deployment-config.json` should look like:

```json
[
  {
    "name": "production-server-1",
    "ip": "192.168.1.1",
    "username": "deploy",
    "port": "22",
    "path": "/var/www/your-app",
    "beforeHooks": "php artisan migrate --force && php artisan cache:clear",
    "afterHooks": "php artisan queue:restart && php artisan config:cache"
  },
  {
    "name": "production-server-2",
    "ip": "192.168.1.2",
    "username": "deploy",
    "port": "22",
    "path": "/var/www/your-app",
    "beforeHooks": "php artisan migrate --force && php artisan cache:clear",
    "afterHooks": "php artisan queue:restart && php artisan config:cache"
  }
]
```

## Usage

1. **Create `deployment-config.json`**: Ensure this file exists in your repository with the correct server configuration.
2. **Set GitHub Secrets**: Add the necessary secrets to your GitHub repository settings:
   - `SSH_KEY`: Private SSH key for server access.
   - `LARAVEL_ENV`: Environment configuration for your Laravel application, aka the `.env` file that should be in production.
3. **Push to Production Branch**: Push your code to the `production` branch to trigger the workflow.

## Acknowledgment

- **appleboy GitHub Actions**: The workflow uses actions from [appleboy](https://github.com/appleboy), such as `scp-action` and `ssh-action`.

## License

This project is licensed under the MIT License.
