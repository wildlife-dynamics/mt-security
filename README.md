# Mara Triangle Security Report Workflow

## Introduction

This workflow helps you generate security reports for the Mara Triangle conservancy by consolidating illegal activity event data from EarthRanger.

**What this workflow does:**
- Downloads illegal activity events (arrests, snare recoveries, poacher camps) from **EarthRanger**
- Processes event details and extracts suspect information
- Generates a security report table with date, event type, suspect details, and comments
- Creates an interactive map showing event locations colored by type
- Produces a Word document (DOCX) report with the map and summary table
- Exports the security report data as CSV

**Who should use this:**
- Conservation managers tracking illegal activity in the Mara Triangle
- Security teams generating monthly situation reports
- Anyone needing a consolidated view of arrests, snare recoveries, and poacher camp discoveries

## Prerequisites

Before using this workflow, you need:

1. **Ecoscope Desktop** installed on your computer
   - If you haven't installed it yet, please follow the installation instructions for Ecoscope Desktop

2. **EarthRanger Data Source** configured in Ecoscope Desktop
   - You must have already set up a connection to your EarthRanger server (e.g., "mmnr")
   - Your data source should be configured with proper authentication credentials

3. **Illegal Activity Events** recorded in EarthRanger
   - Your EarthRanger system should have events of type `arrest_rep`, `snare_rep`, and/or `poacher_camp_rep`
   - You can verify available event types at `https://<your-site>.pamdas.org/admin/activity/eventtype/`

## Installation

1. Select "Workflow Templates" tab
2. Click "+ Add Template"
3. Copy and paste this URL `https://github.com/wildlife-dynamics/mt-security` and wait for the workflow template to be downloaded and initialized
4. The template will now appear in your available template list

## Configuration Guide

### Basic Configuration

#### 1. Workflow Details
Add a name and optional description to identify this workflow run.

- **Workflow Name** (required): A descriptive name for this run
  - Example: `"MT SECURITY"`

#### 2. Time Range
Choose the period of time to analyze.

- **Since** (required): Start date and time
  - Example: `2026-01-01T00:00:00`
- **Until** (required): End date and time
  - Example: `2026-01-31T23:59:59`
- **Timezone** (optional): Your local timezone
  - Example: `Africa/Nairobi (UTC+03:00)`

#### 3. Data Source
Select your configured EarthRanger data source.

- **Data Source** (required): The name of your EarthRanger connection
  - Example: `"mmnr"`

#### 4. Event Types
Specify which illegal event types to include in the report.

- **Event Types** (required): List of EarthRanger event type identifiers
  - Default: `["arrest_rep", "snare_rep", "poacher_camp_rep"]`
  - Note: These must match the event type identifiers in your EarthRanger system. You can find them at `https://<your-site>.pamdas.org/admin/activity/eventtype/`

#### 5. Create Security Report
Configure the Word document report generation.

- **Template Path** (required): Path or URL to the DOCX template file
  - Default: A pre-configured template hosted on GitHub
  - Note: You can provide your own template with Jinja2 placeholders for `report_date`, `event_map`, and `summary`

### Advanced Configuration

These optional settings provide additional control:

#### Bounding Box Filter
Filter events to a specific geographic area (under **Process Events > Filter Event Coordinates**).

- **Min X / Max X**: Longitude range
- **Min Y / Max Y**: Latitude range
- Default: Mara Triangle area (33.0 to 36.0 longitude, -3.0 to 0.0 latitude)

#### Base Map
Change the map tile layer (under **Generate Map > Base Maps**).

- Default: ArcGIS World Topo Map
- Options include: Open Street Map, Roadmap, Satellite, Terrain, and more

## Running the Workflow

Once you've configured all the settings:

1. **Review your configuration**
   - Double-check your time range, data source, and event types

2. **Save and run**
   - Click "Submit" and the workflow will show up in "My Workflows" table
   - Click on "Run" and the workflow will begin processing

3. **Monitor progress and wait for completion**
   - You'll see status updates as the workflow runs
   - Processing time depends on:
     - The size of your date range
     - Number of illegal events in the system
   - The workflow completes with status "Success" or "Failed"

## Understanding Your Results

After the workflow completes successfully, you'll find your outputs in the designated output folder.

### Data Outputs

#### Security Report CSV
- **File format**: CSV
- **Opens in**: Microsoft Excel, Google Sheets
- **Contents**: One row per illegal event with the following columns:
  - `Date`: Date of the event
  - `Type`: Event type (arrest, snare, poacher camp)
  - `Comments`: Event comments from the field report
  - `Number of People`: Count of suspects involved
  - `Place of Origin`: Nationality or origin of suspects
  - `Sex`: Sex of suspects
  - `Details`: Additional event details

#### Word Document Report
- **File format**: DOCX
- **Opens in**: Microsoft Word, Google Docs
- **Contents**: A formatted report containing:
  - Report date header
  - Map of event locations
  - Summary table with all security report details

### Visual Outputs (Dashboard)

The workflow creates an interactive dashboard with 2 main visualizations:

#### Security Report Locations (Map)
- **Format**: Interactive point map
- **Features**:
  - Each event is shown as a colored dot on the map
  - Colors indicate event type (using the Dark2 color palette)
  - Hover over a point to see: Serial Number, Event Time, Event Type
  - Legend shows the color mapping for each event type
  - North arrow in top-left, legend in bottom-right

#### Security Reports (Table)
- **Format**: Interactive data table
- **Features**:
  - Sortable columns
  - Shows: Date, Type, Comments, Number of People, Place of Origin, Sex, Details

## Common Use Cases & Examples

### Example 1: Monthly Security Report
**Goal**: Generate a monthly security report for January 2026

**Configuration**:
- **Time Range**:
  - Since: `2026-01-01T00:00:00`
  - Until: `2026-01-31T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Data Source**: `"mmnr"`
- **Event Types**: `["arrest_rep", "snare_rep", "poacher_camp_rep"]`

**Result**:
- CSV file with all illegal events for January 2026
- DOCX report with map and summary table
- Interactive dashboard with event locations and details

---

### Example 2: Arrests Only
**Goal**: Focus report on arrests only, excluding snare and poacher camp events

**Configuration**:
- **Time Range**:
  - Since: `2026-01-01T00:00:00`
  - Until: `2026-03-31T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Data Source**: `"mmnr"`
- **Event Types**: `["arrest_rep"]`

**Result**:
- CSV and DOCX report containing only arrest events for Q1 2026

---

### Example 3: Snare Recovery Analysis
**Goal**: Analyze snare recovery patterns over a quarter

**Configuration**:
- **Time Range**:
  - Since: `2025-10-01T00:00:00`
  - Until: `2025-12-31T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Data Source**: `"mmnr"`
- **Event Types**: `["snare_rep"]`

**Result**:
- Map showing snare recovery locations
- Table with recovery details and suspect information

## Troubleshooting

### Common Issues and Solutions

#### Workflow Fails to Start
**Problem**: The workflow fails immediately after clicking "Run"

**Solutions**:
- Verify your EarthRanger data source is properly configured with valid credentials
- Check that your EarthRanger server is accessible from your network
- Try re-entering your data source password in Ecoscope Desktop

#### No Events Returned
**Problem**: The workflow completes but the table and map are empty

**Solutions**:
- Check that your date range contains illegal activity events
- Verify the event type identifiers match your EarthRanger configuration (e.g., `arrest_rep`, not `arrest`)
- Try expanding your date range to confirm events exist
- Check your bounding box filter isn't excluding your area of interest

#### Map Shows Wrong Area
**Problem**: The map doesn't show the expected region

**Solutions**:
- Verify the bounding box coordinates under **Process Events > Filter Event Coordinates**
- Default is set for the Mara Triangle area (longitude 33-36, latitude -3 to 0)
- Adjust coordinates to match your area of operation

#### Report Template Error
**Problem**: The workflow fails at "Create Security Report" with a template error

**Solutions**:
- Ensure the template path is valid and accessible
- If using a custom template, verify it contains the required Jinja2 placeholders: `report_date`, `event_map`, and `summary`
- Try using the default template by leaving the template path unchanged

#### Authentication Errors
**Problem**: Error mentioning "server", "token", or "credentials"

**Solutions**:
- Re-enter your EarthRanger credentials in Ecoscope Desktop data source configuration
- Verify your account has permission to access events data
- Contact your EarthRanger administrator if the issue persists

#### Workflow Runs Slowly
**Problem**: The workflow takes a long time to complete

**Solutions**:
- Reduce the date range to a shorter period
- Note that the first run may take longer as the system initializes
- Large numbers of events (hundreds+) will naturally take more time to process
