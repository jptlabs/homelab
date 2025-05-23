# Zipline Setup Guide

> [!NOTE]
> This is an overly simplified guide and it's also customized to __my__ use cases. Read the official setup guide [here](https://github.com/diced/zipline).
>

## Installing with Docker Compose

Run the following commands in your terminal to generate a `.env` file with a PostgreSQL password and a core application secret:

````bash
echo "POSTGRESQL_PASSWORD=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" > .env
echo "CORE_SECRET=$(openssl rand -base64 42 | tr -dc A-Za-z0-9 | cut -c -32 | tr -d '\n')" >> .env
