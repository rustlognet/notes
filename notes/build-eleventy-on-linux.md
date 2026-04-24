## Summary

How to clone an Eleventy repo from GitHub and build it on a Linux server.

**Prerequisite:**

- Node.js installation on the server

## Create a working directory

```bash
sudo mkdir -p /var/www/website
sudo chown -R <user>:<group> /var/www/website
```

- Replace `<user>` and `<group>` with your username and group name.

## Clone 11ty repo from GitHub

```bash
cd /var/www/website
git clone git@github.com:<github_username>/<github_repository>.git .
```

- Replace `<github_username>` and `<github_repository>`.
- The command above assumes you clone the repo via SSH (i.e. SSH keys were generated and the public key was added to your GitHub account).

## Install dependencies and build

```bash
cd /var/www/website
npm ci || npm install
npm run build
```

- Installs dependencies exactly as locked in `package-lock.json` (if available). Otherwise `npm install` installs dependencies from `package.json`.
- 11ty is built into the output folder, e.g. `/var/www/website/dist` (depending on the 11ty configuration file).

## Future updates

```bash
cd /var/www/website
git pull origin main
npm install
rm -rf dist
npm run build
```

## Related

- Nginx installation
- Nginx configuration
- SSL certificate