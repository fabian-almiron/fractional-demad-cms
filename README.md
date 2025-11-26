# Fractional Demand Backend

Strapi CMS backend with PostgreSQL running in Docker.

## Prerequisites

- [Docker](https://www.docker.com/get-started) (version 20.10+)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 2.0+)

## Quick Start

### 1. Clone and Setup Environment

```bash
# Copy the example environment file
cp env.example .env

# Edit .env and update the security keys (especially for production!)
```

### 2. Generate Secure Keys (Important!)

Generate secure random keys for production:

```bash
# Generate keys using openssl
openssl rand -base64 32  # Run 4 times for APP_KEYS
openssl rand -base64 32  # JWT_SECRET
openssl rand -base64 32  # ADMIN_JWT_SECRET
openssl rand -base64 32  # API_TOKEN_SALT
openssl rand -base64 32  # TRANSFER_TOKEN_SALT
```

### 3. Start the Services

```bash
# Build and start containers
docker-compose up -d --build

# View logs
docker-compose logs -f strapi
```

### 4. Access Strapi Admin

Once the containers are running:

- **Admin Panel**: http://localhost:1337/admin
- **API Endpoint**: http://localhost:1337/api

On first launch, you'll be prompted to create an admin account.

## Services

| Service    | Port | Description           |
|------------|------|-----------------------|
| Strapi     | 1337 | Headless CMS          |
| PostgreSQL | 5432 | Database              |

## Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Rebuild containers
docker-compose up -d --build

# View Strapi logs
docker-compose logs -f strapi

# View PostgreSQL logs
docker-compose logs -f postgres

# Access Strapi container shell
docker-compose exec strapi sh

# Access PostgreSQL CLI
docker-compose exec postgres psql -U strapi -d strapi_db

# Reset everything (WARNING: deletes all data!)
docker-compose down -v
```

## Project Structure

```
fractional-deman-backend/
├── docker-compose.yml      # Docker services configuration
├── Dockerfile              # Strapi container build config
├── env.example             # Environment variables template
├── README.md               # This file
└── strapi/                 # Strapi application
    ├── config/             # Strapi configuration
    │   ├── admin.ts        # Admin panel config
    │   ├── api.ts          # API settings
    │   ├── database.ts     # PostgreSQL connection
    │   ├── middlewares.ts  # CORS and security
    │   ├── plugins.ts      # Plugin settings
    │   └── server.ts       # Server settings
    ├── src/
    │   ├── admin/          # Admin customization
    │   ├── api/            # Content types & APIs
    │   └── index.ts        # App bootstrap
    ├── database/
    │   └── migrations/     # Database migrations
    ├── public/             # Static files
    ├── package.json        # Dependencies
    └── tsconfig.json       # TypeScript config
```

## Environment Variables

| Variable             | Default          | Description                     |
|---------------------|------------------|---------------------------------|
| `DATABASE_NAME`     | strapi_db        | PostgreSQL database name        |
| `DATABASE_USERNAME` | strapi           | PostgreSQL username             |
| `DATABASE_PASSWORD` | strapi_password  | PostgreSQL password             |
| `DATABASE_SSL`      | false            | Enable SSL for database         |
| `HOST`              | 0.0.0.0          | Strapi host binding             |
| `PORT`              | 1337             | Strapi port                     |
| `NODE_ENV`          | development      | Environment mode                |
| `JWT_SECRET`        | -                | JWT token secret (required)     |
| `ADMIN_JWT_SECRET`  | -                | Admin JWT secret (required)     |
| `APP_KEYS`          | -                | App keys, comma-separated       |
| `API_TOKEN_SALT`    | -                | API token salt (required)       |
| `TRANSFER_TOKEN_SALT`| -               | Transfer token salt (required)  |

## Creating Content Types

After starting Strapi:

1. Open http://localhost:1337/admin
2. Navigate to **Content-Type Builder**
3. Click **Create new collection type**
4. Define your fields
5. Save and restart server if prompted

Content types are automatically saved in `strapi/src/api/`.

## CORS Configuration

The CORS is configured in `strapi/config/middlewares.ts` to allow requests from:

- http://localhost:3000
- http://localhost:3001
- http://127.0.0.1:3000

Add additional origins as needed for your frontend applications.

## Development vs Production

### Development (default)

- Uses Strapi `develop` command with hot-reload
- Relaxed security settings
- Admin panel building on changes

### Production

Update `.env`:

```bash
NODE_ENV=production
DATABASE_SSL=true
```

Update `Dockerfile` CMD:

```dockerfile
CMD ["npm", "run", "start"]
```

## Troubleshooting

### Container won't start

```bash
# Check logs
docker-compose logs -f

# Rebuild from scratch
docker-compose down -v
docker-compose up -d --build
```

### Database connection issues

```bash
# Ensure postgres is healthy
docker-compose ps

# Check postgres logs
docker-compose logs postgres
```

### Permission errors

```bash
# Fix strapi folder permissions
sudo chown -R $USER:$USER ./strapi
```

## Data Persistence

- PostgreSQL data is stored in the `postgres_data` Docker volume
- Strapi uploads are stored in `strapi/public/uploads/`
- Node modules are cached in `strapi_node_modules` volume

To backup your database:

```bash
docker-compose exec postgres pg_dump -U strapi strapi_db > backup.sql
```

To restore:

```bash
cat backup.sql | docker-compose exec -T postgres psql -U strapi strapi_db
```

## License

MIT
