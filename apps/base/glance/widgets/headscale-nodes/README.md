# Headscale VPN Nodes Widget

This directory contains the complete Headscale nodes widget for Glance dashboard.

## Files

- **widget.yaml** - Widget metadata and configuration (title, URL, cache settings)
- **template.html** - Go template HTML markup
- **style.css** - Widget-specific CSS styles

## Usage

Since Glance doesn't support external template references, the content from these files must be assembled into `glance.yml` manually.

**Location in glance.yml**: `pages[0].columns[2].widgets` (Right sidebar)

## Widget Features

- Displays all Headscale VPN nodes
- Online/offline status indicators (green/red dots)
- Router indicator for nodes with approved routes (light blue dot)
- ACL tags display (e.g., "homelab")
- Hover to see IP address
- Click title to open Headplane admin interface

## API Endpoint

**URL**: `https://headscale.sarma.love/api/v1/node`
**Auth**: Bearer token from `HEADSCALE_API_KEY` environment variable
