# Ticket Tracker Monitor

A TypeScript-based monitoring service that tracks ticket availability and pricing data from Vivid Seats. This worker service powers the backend monitoring for the [Ticket Tracker web application](https://github.com/RightNxw/ticket-tracker), automatically fetching event information and updating the centralized API with real-time ticket data.

## Overview

The Ticket Tracker Monitor is designed to continuously monitor events on Vivid Seats, tracking ticket prices, availability, and venue information. It runs as a background worker that:

- Fetches configured events from a central API
- Monitors Vivid Seats for ticket availability and pricing updates
- Updates the central database with latest ticket information
- Runs automatically every 12 hours

## Features

- **Automated Monitoring**: Continuously tracks ticket data at 12-hour intervals
- **Multi-Event Support**: Monitors multiple events and venues simultaneously
- **Detailed Logging**: Winston-based logging system with colored console output and file logging
- **API Integration**: Seamless integration with the main ticket tracker API
- **Venue Metadata**: Automatically fetches and stores artist names, stadium information, and URLs
- **Error Handling**: Robust error handling and logging for reliable operation

## Architecture

The application consists of three main components:

### 1. Main Entry Point (`index.ts`)
- Fetches events from the central API
- Initializes monitoring for each event
- Runs on a 12-hour interval cycle

### 2. Vivid Seats Monitor (`modules/vivid-seats.ts`)
- `VividSeatsMonitor` class handles all Vivid Seats API interactions
- Fetches venue/production data using performer and venue IDs
- Updates ticket counts and pricing information
- Retrieves additional venue metadata (artist, stadium, URLs)

### 3. Logger (`helpers/logger.ts`)
- Winston-based logging with custom levels
- Color-coded console output for different log levels
- File logging to `combined.log`
- Custom log levels: error, warn, info, queue, debug

## Prerequisites

- Node.js 18.16.0 or higher
- npm or yarn
- TypeScript 5.1.6+

## Installation

1. Clone the repository:
```bash
git clone https://github.com/RightNxw/ticket-tracker.git
cd ticket-tracker-monitor
```

2. Install dependencies:
```bash
npm install
```

3. Build the TypeScript code:
```bash
npm run build
```

## Usage

### Development Mode
Run with automatic recompilation and restart on changes:
```bash
npm run dev
```

### Production Mode
Build and run the compiled JavaScript:
```bash
npm run build
npm start
```

### Deployment

The application is configured for Heroku deployment with:
- `Procfile` specifying the worker process
- Automatic TypeScript compilation via `heroku-postbuild` script
- Node.js version pinned to 18.16.0

To deploy to Heroku:
```bash
heroku create your-app-name
git push heroku main
heroku ps:scale worker=1
```

## Configuration

### API Endpoints

The monitor communicates with the following API endpoints:

- **Get Events**: `GET https://ticket-tracker-omega.vercel.app/api/event`
  - Returns list of events to monitor with `performerId` and `venueId`

- **Update Venue**: `POST https://ticket-tracker-omega.vercel.app/api/venue`
  - Updates venue information including ticket counts, pricing, and metadata

- **Get Venue**: `GET https://ticket-tracker-omega.vercel.app/api/venue?id={venueId}`
  - Retrieves existing venue data

### Vivid Seats API

The monitor accesses Vivid Seats public API endpoints:
- `https://www.vividseats.com/hermes/api/v1/productions/related`
- `https://www.vividseats.com/hermes/api/v1/listings`

## Project Structure

```
ticket-tracker-monitor/
├── dist/                  # Compiled JavaScript output
├── helpers/
│   └── logger.ts         # Winston logger configuration
├── modules/
│   └── vivid-seats.ts    # Vivid Seats monitoring logic
├── index.ts              # Main entry point
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
├── Procfile             # Heroku process configuration
└── README.md            # This file
```

## Data Flow

1. **Event Retrieval**: Main process fetches event list from central API
2. **Monitor Initialization**: Creates `VividSeatsMonitor` instance for each event
3. **Venue Monitoring**:
   - Fetches related productions from Vivid Seats
   - Extracts ticket count, min price, and date information
   - Posts updates to central API
4. **Metadata Enrichment**: If venue is missing artist/stadium data, fetches additional info
5. **Continuous Loop**: Process repeats every 12 hours

## Logging

Logs are output to both console and `combined.log` file with the following format:
```
YYYY-MM-DD HH:mm:ss [Label] level: message
```

Log levels (in order of severity):
- `error` (red): Critical errors
- `warn` (yellow): Warnings
- `info` (grey): General information
- `queue` (green): Queue operations
- `debug` (blue): Debug information

## Dependencies

### Production
- `axios`: HTTP client for API requests
- `winston`: Advanced logging library

### Development
- `typescript`: TypeScript compiler
- `@types/node`: Node.js type definitions
- `@types/winston`: Winston type definitions

## Error Handling

The application includes comprehensive error handling:
- API request failures are logged with detailed error messages
- Individual venue processing errors don't stop the entire monitoring cycle
- HTTP status codes are checked and logged
- All errors include contextual information for debugging

## Notes

- The monitor runs every 12 hours to balance data freshness with API rate limits
- User-Agent headers mimic a Chrome browser to ensure successful API requests
- All timestamps use Unix epoch format (milliseconds)
- The service is designed to run continuously as a background worker
