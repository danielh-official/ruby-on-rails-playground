# Copilot Instructions for RubyOnRailsPlayground

## Project Overview
This is a Rails 8.0 application using the modern Rails stack with Solid Queue, Solid Cache, and Solid Cable for background jobs, caching, and WebSocket connections respectively. The app is containerized with Docker and configured for deployment via Kamal.

## Architecture & Key Technologies

### Modern Rails Stack (Rails 8.0)
- **Database**: SQLite3 with multiple database configuration (primary, cache, queue, cable)
- **Asset Pipeline**: Propshaft (not Sprockets)
- **JavaScript**: Import maps with Stimulus controllers, no build step required
- **Styling**: Basic CSS in `app/assets/stylesheets/application.css`
- **Background Jobs**: Solid Queue (runs in Puma process via `SOLID_QUEUE_IN_PUMA=true`)

### Database Architecture
Production uses 4 separate SQLite databases:
- `storage/production.sqlite3` - Primary app data
- `storage/production_cache.sqlite3` - Solid Cache storage
- `storage/production_queue.sqlite3` - Solid Queue jobs
- `storage/production_cable.sqlite3` - Solid Cable connections

Each has dedicated migration paths: `db/cache_migrate`, `db/queue_migrate`, `db/cable_migrate`.

## Development Workflow

### Setup & Running
```bash
bin/setup          # Initial setup (bundle install, db setup)
bin/dev            # Start development server (shortcut to rails server)
bin/rails server   # Standard Rails server
```

### Key Development Commands
```bash
bin/rails generate  # Use Rails generators
bin/rubocop        # Code style checking (Rails Omakase style)
bin/brakeman       # Security vulnerability scanning
bin/importmap      # Manage JavaScript imports
```

### Testing
- **Framework**: Built-in Rails testing with Capybara for system tests
- **Run tests**: `bin/rails test` and `bin/rails test:system`
- **Test files**: Organized in `test/` directory with standard Rails structure

## JavaScript & Frontend

### Stimulus Controllers
- Located in `app/javascript/controllers/`
- Auto-imported via `pin_all_from "app/javascript/controllers"`
- Example: `hello_controller.js` demonstrates basic Stimulus pattern
- Use Turbo for SPA-like navigation without build complexity

### Asset Management
- **No build step required** - uses import maps
- Add new JS dependencies via `bin/importmap pin package-name`
- CSS goes in `app/assets/stylesheets/`

## Deployment & Production

### Kamal Deployment
- Configuration in `config/deploy.yml`
- Service name: `ruby_on_rails_playground`
- Deploys to containerized environment with SSL via Let's Encrypt
- Solid Queue runs inside Puma process (single-server setup)

### Docker
- Multi-stage build optimized for production
- Uses Thruster for HTTP caching/compression
- SQLite storage mounted as persistent volumes

### Environment Variables
- `SOLID_QUEUE_IN_PUMA=true` - Runs background jobs in web process
- `JOB_CONCURRENCY` - Controls Solid Queue worker processes
- `RAILS_MASTER_KEY` - Required for production (from `config/master.key`)

## Code Patterns & Conventions

### Rails 8 Specific Patterns
- Use `config/application.rb` autoload settings: `config.autoload_lib(ignore: %w[assets tasks])`
- Multi-database setup requires specifying database in models if not using primary
- Solid Queue jobs inherit from `ApplicationJob` and use standard Active Job patterns

### File Organization
- Controllers: Standard Rails MVC in `app/controllers/`
- Models: Standard Rails with concerns in `app/models/concerns/`
- Views: ERB templates in `app/views/`
- Jobs: Background jobs in `app/jobs/`
- Helpers: View helpers in `app/helpers/`

### Code Style
- Follows Rails Omakase conventions via `rubocop-rails-omakase`
- Run `bin/rubocop` before committing
- Security scanning with `bin/brakeman`

## Common Tasks

### Adding Background Jobs
1. Generate job: `bin/rails generate job ProcessSomething`
2. Jobs automatically use Solid Queue (no Redis needed)
3. Monitor via Rails console: `SolidQueue::Job.all`

### Adding New Routes/Controllers
1. Add routes in `config/routes.rb`
2. Generate controller: `bin/rails generate controller Posts index show`
3. Add corresponding views in `app/views/`

### Database Changes
- Migrations go in `db/migrate/` for primary database
- Cache/queue/cable migrations auto-generated and separate
- Use `bin/rails db:migrate` for all databases

## PWA Support
- Configured but commented out in routes
- Manifest and service worker templates in `app/views/pwa/`
- Uncomment PWA routes in `config/routes.rb` to enable