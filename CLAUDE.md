# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Campfire is a web-based chat application built with Ruby on Rails. It supports multiple rooms with access controls, direct messages, file attachments, search, notifications via Web Push, @mentions, and API support for bot integrations. The application is single-tenant with public rooms accessible to all users in the system.

## Development Commands

### Setup and Development
- **Initial setup**: `bin/setup`
- **Start development server**: `bin/rails server` or `bin/dev`
- **Reset database**: `bin/setup --reset`

### Testing
- **Run all tests**: `bin/rails test`
- **Run system tests**: `bin/rails test:system`
- **Run CI suite**: `bin/ci` (includes style checks, security audits, and all tests)

### Code Quality
- **Ruby style checks**: `bin/rubocop`
- **Security audit**: `bin/bundler-audit` and `bin/brakeman`
- **Importmap audit**: `bin/importmap audit`

### Docker Development
- **Build image**: `docker build -t campfire .`
- **Run container**: See README.md for complete Docker run commands with environment variables

## Architecture Overview

### Core Models
- **Account**: Single-tenant container for all users and rooms
- **User**: Can be regular users or bots (User::Bot)
- **Room**: Base class with subtypes (Rooms::Open, Rooms::Closed, Rooms::Direct)
- **Message**: Core chat functionality with attachments, mentions, and search
- **Membership**: Links users to rooms with roles and permissions
- **Session**: User authentication and device management

### Key Features Architecture
- **Authentication**: Session-based auth with device tracking and transfers
- **Real-time**: ActionCable for WebSocket connections and live updates
- **File Handling**: Active Storage with image processing via libvips
- **Search**: Full-text search across messages with pagination
- **Notifications**: Web Push API integration with VAPID keys
- **Bot API**: Webhook-based bot integration with unique keys per bot

### Frontend Stack
- **Hotwire**: Turbo + Stimulus for reactive UI without complex JavaScript
- **Importmap**: ES modules without bundling
- **Propshaft**: Asset pipeline for modern Rails

### Background Jobs
- **Resque**: Redis-backed job processing
- **Resque Pool**: Multi-worker job management

## File Structure

### Controllers
- Organized by resource with nested modules (e.g., `accounts/`, `rooms/`, `users/`)
- Concerns in `app/controllers/concerns/` for shared functionality
- Bot API endpoints under `messages/by_bots_controller.rb`

### Models
- STI pattern for Room types (`rooms/open.rb`, `rooms/closed.rb`, `rooms/direct.rb`)
- Concerns for shared model behavior (e.g., `message/searchable.rb`)
- Namespace organization (e.g., `opengraph/` for link unfurling)

### Configuration
- Environment-specific configs in `config/environments/`
- Kamal deployment config in `config/deploy.yml`
- Redis configuration in `config/redis.conf`

## Environment Variables

### Required for Production
- `SECRET_KEY_BASE`: Rails secret key
- `RAILS_MASTER_KEY`: Encryption key for credentials

### Optional Features
- `SSL_DOMAIN`: Enable Let's Encrypt SSL
- `DISABLE_SSL`: Serve over HTTP instead
- `VAPID_PUBLIC_KEY`/`VAPID_PRIVATE_KEY`: Web Push notifications
- `SENTRY_DSN`: Error reporting

## Deployment

The application uses Kamal for deployment with the following setup:
- Docker-based deployment to AWS EC2
- SSL termination via proxy
- Single server architecture with built-in Redis and SQLite
- Image hosted on Docker Hub (`tatsu0218/once-chat`)

## Testing Strategy

Tests are organized by type:
- **Unit tests**: Models and helpers in `test/models/` and `test/helpers/`
- **Controller tests**: Full coverage in `test/controllers/`
- **System tests**: End-to-end scenarios in `test/system/`
- **Performance tests**: Load testing scripts in `test/performance/`

Test data uses fixtures with realistic relationships between accounts, users, rooms, and messages.