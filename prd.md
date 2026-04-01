---
title: 'Mara Triangle Illegal Events SITREP'
repo_name: 'mt-sitrep'
workflow_id: 'mt_sitrep'
created: '2026-04-01'
status: 'ready-for-dev'
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
data_sources: [set_er_connection]
reference_workflows: [wt-download-events, mt-wildlife]
tasks_identified: [set_workflow_details, set_time_range, get_timezone_from_time_range, set_er_connection, get_events, convert_values_to_timezone, apply_reloc_coord_filter, process_events_details, normalize_json_column, drop_column_prefix, sitrep_table, apply_color_map, map_columns, set_string_var, set_base_maps, create_point_layer, draw_ecomap, persist_text, create_map_widget_single_view, draw_table, create_table_widget_single_view, persist_df_wrapper, create_docx, gather_dashboard]
---

# PRD: Mara Triangle Illegal Events SITREP

**Created:** 2026-04-01
**Repo:** `/Users/yunwu/MEP/wt/mt-sitrep/`
**Workflow ID:** `mt_sitrep`

## Overview

### Problem Statement

Mara Triangle conservancy tracks illegal activity events (arrests, snares, poacher camps) in EarthRanger. Currently, generating a SITREP report requires running three separate workflows: one to download events, one to generate a summary table with suspect details, and one to generate a map. This is error-prone and slow.

### Solution

A single consolidated workflow that fetches illegal events from EarthRanger, processes them into a SITREP table (with suspect details), generates a point map colored by event type, produces a DOCX report with both the map and table, and displays a dashboard with map and table widgets.

### Scope

**In Scope:**
- Fetch illegal events (arrest_rep, snare_rep, poacher_camp_rep) from EarthRanger
- Process event details (normalize, extract suspect info)
- Generate SITREP table: Date, Type, Comments, Number of People, Place of Origin, Sex, Details
- Generate point map of events colored by event type
- Dashboard with map widget and table widget
- DOCX report with map images and SITREP table
- CSV export of SITREP table

**Out of Scope:**
- Historical trend analysis across months
- Patrol correlation with illegal events
- Real-time event alerting

## Data Sources & Connections

| Source | Type | Connection | Notes |
| ----- | ---- | ---------- | ----- |
| EarthRanger | Events (arrest_rep, snare_rep, poacher_camp_rep) | `set_er_connection` (`mmnr`) | Bounding box: min_x=33, min_y=-3, max_x=36, max_y=0 |

## Reference Workflows

| Workflow | Relevance | Patterns Borrowed |
| -------- | --------- | ----------------- |
| wt-download-events | EarthRanger event fetching & processing | get_events → process_events_details → normalize → drop_prefix pipeline, point map generation |
| mt-wildlife | Point map + DOCX report for SMART events | apply_sql_query pattern (adapted), create_docx with images + tables, report groupers |

## Task Pipeline

### Identified Tasks

| # | Task | ID | Group | Purpose |
|---|------|----|-------|---------|
| 1 | set_workflow_details | workflow_details | — | Dashboard metadata |
| 2 | set_time_range | time_range | — | User time range |
| 3 | get_timezone_from_time_range | get_timezone | — | Extract timezone |
| 4 | set_er_connection | er_client_name | — | EarthRanger connection |
| 5 | get_events | get_event_data | — | Fetch illegal events |
| 6 | convert_values_to_timezone | convert_tz | Process | Convert time column |
| 7 | apply_reloc_coord_filter | filter_events | Process | Filter bad coordinates |
| 8 | process_events_details | process_details | Process | Resolve event detail enums |
| 9 | normalize_json_column | normalize_event_details | Process | Flatten event_details JSON |
| 10 | drop_column_prefix | drop_details_prefix | Process | Remove event_details__ prefix |
| 11 | sitrep_table (ext-mt) | sitrep_data | Process | Extract suspect details into table |
| 12 | apply_color_map | events_colormap | Process | Color by event_type |
| 13 | set_string_var | set_map_title | Map | Map widget title |
| 14 | set_base_maps | base_map_defs | Map | Tile layer config |
| 15 | map_columns | rename_display_cols | Map | Rename for tooltips |
| 16 | create_point_layer | event_point_layer | Map | Single point layer |
| 17 | draw_ecomap | event_ecomap | Map | Render map |
| 18 | persist_text | ecomap_html_url | Map | Save map HTML |
| 19 | create_map_widget_single_view | map_widget | Map | Map widget |
| 20 | set_string_var | set_table_title | Table | Table widget title |
| 21 | draw_table | sitrep_html_table | Table | Render HTML table |
| 22 | persist_text | persist_table_html | Table | Save table HTML |
| 23 | create_table_widget_single_view | table_widget | Table | Table widget |
| 24 | persist_df_wrapper | persist_sitrep_csv | — | Export SITREP as CSV |
| 25 | create_docx | sitrep_report | — | Generate DOCX report |
| 26 | gather_dashboard | dashboard | — | Dashboard assembly |

### Spec.yaml Draft

```yaml
id: mt_sitrep

requirements:
  - name: ecoscope-workflows-core
    version: ">=0.22.18, <0.23.0"
    channel: https://repo.prefix.dev/ecoscope-workflows/
  - name: ecoscope-workflows-ext-ecoscope
    version: ">=0.22.18, <0.23.0"
    channel: https://repo.prefix.dev/ecoscope-workflows/
  - name: ecoscope-workflows-ext-custom
    version: ">=0.0.41, <0.1.0"
    channel: https://repo.prefix.dev/ecoscope-workflows-custom/
  - name: ecoscope-workflows-ext-mt
    version: ">=0.0.1"
    channel: https://repo.prefix.dev/ecoscope-workflows-custom/

rjsf-overrides:
  properties:
    get_event_data.properties.event_types.default: ["arrest_rep", "snare_rep", "poacher_camp_rep"]
    get_event_data.properties.event_types.description: >-
      Illegal event types to include in the SITREP report.
    Process Events.properties.filter_events.properties.bounding_box.default:
      min_x: 33.0
      min_y: -3.0
      max_x: 36.0
      max_y: 0.0
    base_map_defs.properties.base_maps.default:
      - url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}"
        opacity: 1
    persist_sitrep_csv.properties.filetypes.default: ["csv"]
    persist_sitrep_csv.properties.filetypes.items.enum: ["csv"]
    sitrep_report.properties.template_path.default: "https://raw.githubusercontent.com/wildlife-dynamics/mt-sitrep/main/resources/templates/mt_sitrep_report_template.docx"

  uiSchema:
    Process Events.filter_events.filter_point_coords.items.ui:options.label: false

task-instance-defaults:
  skipif:
    conditions:
      - any_is_empty_df
      - any_dependency_skipped

workflow:
  # Setup
  - name: Workflow Details
    id: workflow_details
    task: set_workflow_details

  - name: Time Range
    id: time_range
    task: set_time_range
    partial:
      time_format: '%d %b %Y %H:%M:%S %Z'

  - name: Extract Timezone
    id: get_timezone
    task: get_timezone_from_time_range
    partial:
      time_range: ${{ workflow.time_range.return }}

  - name: Data Source
    id: er_client_name
    task: set_er_connection

  # Data Fetching
  - name: Get Illegal Events
    id: get_event_data
    task: get_events
    partial:
      client: ${{ workflow.er_client_name.return }}
      time_range: ${{ workflow.time_range.return }}
      event_columns: null
      raise_on_empty: false
      include_details: true
      include_updates: false
      include_related_events: false
      include_display_values: true
      include_null_geometry: true

  # Phase 2 stand-in (uncomment to develop with local data):
  # - name: Load Events
  #   id: get_event_data
  #   task: load_df
  #   partial:
  #     deserialize_json: false

  # Processing
  - title: Process Events
    type: task-group
    description: "Process illegal events: filter, normalize details, generate SITREP table."
    tasks:
      - name: Convert to Timezone
        id: convert_tz
        task: convert_values_to_timezone
        partial:
          df: ${{ workflow.get_event_data.return }}
          timezone: ${{ workflow.get_timezone.return }}
          columns: ["time"]

      - name: Filter Event Coordinates
        id: filter_events
        task: apply_reloc_coord_filter
        partial:
          df: ${{ workflow.convert_tz.return }}
          roi_gdf: null
          roi_name: null
          reset_index: false
          filter_point_coords:
            - {x: 180.0, y: 90.0}
            - {x: 0.0, y: 0.0}
            - {x: 1.0, y: 1.0}

      - name: Process Event Details
        id: process_details
        task: process_events_details
        partial:
          df: ${{ workflow.filter_events.return }}
          client: ${{ workflow.er_client_name.return }}
          map_to_titles: true
          ordered: true

      - name: Normalize Event Details
        id: normalize_event_details
        task: normalize_json_column
        partial:
          df: ${{ workflow.process_details.return }}
          column: "event_details"
          skip_if_not_exists: true
          sort_columns: false

      - name: Remove Event Details Prefix
        id: drop_details_prefix
        task: drop_column_prefix
        partial:
          df: ${{ workflow.normalize_event_details.return }}
          prefix: "event_details__"
          duplicate_strategy: "suffix"

      - name: Generate SITREP Table
        id: sitrep_data
        task: ecoscope_workflows_ext_mt.tasks.sitrep_table
        partial:
          df: ${{ workflow.drop_details_prefix.return }}

      - name: Events Colormap
        id: events_colormap
        task: apply_color_map
        partial:
          df: ${{ workflow.drop_details_prefix.return }}
          input_column_name: "event_type"
          colormap: "Dark2"
          output_column_name: "event_type_colormap"

  # Map
  - title: Generate Map
    type: task-group
    description: "Generate point map of illegal events colored by event type."
    tasks:
      - name: Set Map Title
        id: set_map_title
        task: set_string_var
        partial:
          var: "Illegal Events Map"

      - name: Base Maps
        id: base_map_defs
        task: set_base_maps

      - name: Rename Columns for Display
        id: rename_display_cols
        task: map_columns
        partial:
          df: ${{ workflow.events_colormap.return }}
          drop_columns: []
          retain_columns: []
          raise_if_not_found: false
          rename_columns:
            time: "Event Time"
            event_type: "Event Type"
            serial_number: "Serial Number"

      - name: Create Point Layer
        id: event_point_layer
        task: create_point_layer
        skipif:
          conditions:
            - any_is_empty_df
            - any_dependency_skipped
            - all_geometry_are_none
        partial:
          geodataframe: ${{ workflow.rename_display_cols.return }}
          layer_style:
            fill_color_column: "event_type_colormap"
            get_radius: 5
          legend:
            label_column: "Event Type"
            color_column: "event_type_colormap"
          tooltip_columns: ["Serial Number", "Event Time", "Event Type"]

      - name: Draw Ecomap
        id: event_ecomap
        task: draw_ecomap
        partial:
          geo_layers: ${{ workflow.event_point_layer.return }}
          title: null
          tile_layers: ${{ workflow.base_map_defs.return }}
          north_arrow_style:
            placement: "top-left"
          legend_style:
            title: "Event Type"
            format_title: false
            placement: "bottom-right"
          static: false
          max_zoom: 20

      - name: Persist Ecomap
        id: ecomap_html_url
        task: persist_text
        partial:
          text: ${{ workflow.event_ecomap.return }}
          root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
          filename_suffix: "illegal_events_map"

      - name: Create Map Widget
        id: map_widget
        task: create_map_widget_single_view
        skipif:
          conditions:
            - never
        partial:
          title: ${{ workflow.set_map_title.return }}
          data: ${{ workflow.ecomap_html_url.return }}

  # Table Widget
  - title: SITREP Table Display
    type: task-group
    description: "Render SITREP table as dashboard widget."
    tasks:
      - name: Set Table Title
        id: set_table_title
        task: set_string_var
        partial:
          var: "SITREP Summary"

      - name: Draw SITREP Table
        id: sitrep_html_table
        task: draw_table
        partial:
          dataframe: ${{ workflow.sitrep_data.return }}
          widget_id: ${{ workflow.set_table_title.return }}

      - name: Persist Table HTML
        id: persist_table_html
        task: persist_text
        partial:
          text: ${{ workflow.sitrep_html_table.return }}
          root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
          filename_suffix: "sitrep_table"

      - name: Create Table Widget
        id: table_widget
        task: create_table_widget_single_view
        skipif:
          conditions:
            - never
        partial:
          title: ${{ workflow.set_table_title.return }}
          data: ${{ workflow.persist_table_html.return }}

  # Data Export
  - name: Persist SITREP CSV
    id: persist_sitrep_csv
    task: persist_df_wrapper
    partial:
      df: ${{ workflow.sitrep_data.return }}
      root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
      filename_prefix: "sitrep"
      filetypes: ["csv"]
      sanitize: false
    skipif:
      conditions:
        - never

  # Report
  - name: Create SITREP Report
    id: sitrep_report
    task: ecoscope_workflows_ext_custom.tasks.results.create_docx
    partial:
      context:
        items:
          - item_type: timerange
            key: report_date
            value: ${{ workflow.time_range.return }}
            format: "%b %Y"
          - item_type: image
            key: event_map
            value: ${{ workflow.ecomap_html_url.return }}
            screenshot_config:
              wait_for_timeout: 1000
              max_concurrent_pages: 2
              device_scale_factor: 1.0
          - item_type: table
            key: sitrep_table
            value: ${{ workflow.sitrep_data.return }}
      output_dir: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
      filename_prefix: mt_sitrep_report
    skipif:
      conditions:
        - never

  # Dashboard
  - name: Create Dashboard
    id: dashboard
    task: gather_dashboard
    partial:
      details: ${{ workflow.workflow_details.return }}
      widgets:
        - ${{ workflow.map_widget.return }}
        - ${{ workflow.table_widget.return }}
      time_range: ${{ workflow.time_range.return }}
```

### Task Gaps

| Task | Status | Action |
|------|--------|--------|
| sitrep_table | Exists in ext-mt, not published | Publish `ecoscope-workflows-ext-mt` to `repo.prefix.dev/ecoscope-workflows-custom/` channel when publishing workflow |

All other tasks exist in core/ecoscope/custom libraries.

## Development Strategy

### Data Source Approach

3-phase approach — EarthRanger is a remote API.

### Phase 1 — Bootstrap Data

**Download workflow:** Use `wt-download-events` with illegal event params to fetch data locally.

| Data | Source Task | Output Format | Output Path |
| ---- | ----------- | ------------- | ----------- |
| Illegal events | get_events (arrest_rep, snare_rep, poacher_camp_rep) | Parquet | `resources/mock-data/illegal_events.parquet` |

### Phase 2 — Develop with Local Data

**load_df stand-in configuration:**

```yaml
# Replace get_events with:
- name: Load Events
  id: get_event_data
  task: load_df
  partial:
    deserialize_json: false
```

Comment out `set_er_connection` and `process_events_details` (which requires client). Adjust pipeline to skip `process_details` step — the downloaded data already has processed details.

**What to complete in Phase 2:**
- [ ] Full processing pipeline (fetch → normalize → sitrep_table + colormap → map + table)
- [ ] Map output validated visually
- [ ] Table widget validated
- [ ] CSV export generated
- [ ] DOCX report renders with map images and sitrep table
- [ ] rjsf form configured and validated
- [ ] Dashboard layout finalized
- [ ] Base test case passing (`mock_io: true`)

### Phase 3 — Reconnect Live Data Source

**Final data source configuration:**

```yaml
- name: Get Illegal Events
  id: get_event_data
  task: get_events
  partial:
    client: ${{ workflow.er_client_name.return }}
    time_range: ${{ workflow.time_range.return }}
    event_columns: null
    raise_on_empty: false
    include_details: true
    include_updates: false
    include_related_events: false
    include_display_values: true
    include_null_geometry: true
```

**Test case updates:**
- `base` case: `mock_io: true` (mocked API)
- `integration` case: `mock_io: false` with real mmnr connection

## Output Configuration

### Maps

| Map | Layer Type | Style Config | Legend | Notes |
| --- | ---------- | ------------ | ------ | ----- |
| Illegal Events | `create_point_layer` | `fill_color_column: event_type_colormap`, `get_radius: 5` | label: Event Type, color: event_type_colormap, bottom-right | Single map, all events, Dark2 colormap |

Tooltips: Serial Number, Event Time, Event Type — columns renamed via `map_columns` before layer creation.

### Charts

None.

### Data Exports

| Export | Format | Sanitize | Notes |
| ------ | ------ | -------- | ----- |
| SITREP table | CSV | false | Clean tabular data from sitrep_table, skipif: never |

### DOCX Report

| Item | Type | Source | Notes |
| ---- | ---- | ------ | ----- |
| Report date | timerange | time_range | Format: "%b %Y" |
| Event map | image | ecomap_html_url | Screenshot with 1s wait, single map |
| SITREP table | table | sitrep_data | Full suspect details table |

### Widget & Dashboard Assembly

| Widget | Type | Title Source | Content Source |
| ------ | ---- | ------------ | -------------- |
| Illegal Events Map | create_map_widget_single_view | set_map_title ("Illegal Events Map") | ecomap_html_url |
| SITREP Summary | create_table_widget_single_view | set_table_title ("SITREP Summary") | persist_table_html |

**gather_dashboard:**
```yaml
details: ${{ workflow.workflow_details.return }}
widgets:
  - ${{ workflow.map_widget.return }}
  - ${{ workflow.table_widget.return }}
time_range: ${{ workflow.time_range.return }}
```

## Form Configuration (rjsf)

### Parameter Visibility

| Parameter | Basic / Advanced | Default | Notes |
| --------- | ---------------- | ------- | ----- |
| workflow_details | Basic | — | Name and description |
| time_range | Basic | — | Since/until with timezone |
| er_client_name | Basic | — | EarthRanger connection |
| get_event_data.event_types | Basic | ["arrest_rep", "snare_rep", "poacher_camp_rep"] | Illegal event types |
| filter_events.bounding_box | Advanced | Mara Triangle bbox | Coordinate bounds |
| filter_events.filter_point_coords | Advanced | Bad coord filter | Labels hidden via uiSchema |
| base_map_defs | Advanced | ArcGIS World Topo | Map tiles |
| persist_sitrep_csv.filetypes | Advanced | ["csv"] | Export format |
| sitrep_report.template_path | Advanced | GitHub-hosted template | DOCX template URL |

### rjsf-overrides Draft

```yaml
rjsf-overrides:
  properties:
    get_event_data.properties.event_types.default: ["arrest_rep", "snare_rep", "poacher_camp_rep"]
    get_event_data.properties.event_types.description: >-
      Illegal event types to include in the SITREP report.
    Process Events.properties.filter_events.properties.bounding_box.default:
      min_x: 33.0
      min_y: -3.0
      max_x: 36.0
      max_y: 0.0
    base_map_defs.properties.base_maps.default:
      - url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/{z}/{y}/{x}"
        opacity: 1
    persist_sitrep_csv.properties.filetypes.default: ["csv"]
    persist_sitrep_csv.properties.filetypes.items.enum: ["csv"]
    sitrep_report.properties.template_path.default: "https://raw.githubusercontent.com/wildlife-dynamics/mt-sitrep/main/resources/templates/mt_sitrep_report_template.docx"

  uiSchema:
    Process Events.filter_events.filter_point_coords.items.ui:options.label: false
```

### Validation Checklist

- [x] Event type defaults match Mara Triangle illegal categories
- [x] Bounding box defaults to Mara Triangle area
- [x] Base map defaults to ArcGIS World Topo (good for Africa)
- [x] Export format constrained to CSV
- [x] DOCX template URL defaults to main branch
- [x] Coordinate filter point labels hidden

## Test Strategy

### Test Cases

| Case Name | mock_io | Purpose |
| --------- | ------- | ------- |
| integration | false | Real EarthRanger API with mmnr connection |

### test-cases.yaml Draft

```yaml
integration:
  name: Integration Test
  mock_io: false
  params:
    workflow_details:
      name: "MT SITREP"
    time_range:
      since: "2026-01-01T00:00:00Z"
      until: "2026-01-31T23:59:59Z"
      timezone:
        label: "Africa/Nairobi (UTC+03:00)"
        tzCode: "Africa/Nairobi"
        name: "(UTC+03:00) Nairobi"
        utc: "+03:00"
    er_client_name:
      data_source:
        name: "mmnr"
    get_event_data:
      event_types: ["arrest_rep", "snare_rep", "poacher_camp_rep"]
    filter_events:
      bounding_box:
        min_x: 33.0
        min_y: -3.0
        max_x: 36.0
        max_y: 0.0
    set_map_title:
      var: "Illegal Events Map"
    set_table_title:
      var: "SITREP Summary"
    persist_sitrep_csv:
      filename_prefix: "sitrep"
      filetypes: ["csv"]
    sitrep_report:
      template_path: "resources/templates/mt_sitrep_report_template.docx"
      filename_prefix: "mt_sitrep_report"
```

### Data Source Testing

| Data Source | mock_io: true behavior | mock_io: false requirements |
| ----------- | ---------------------- | --------------------------- |
| EarthRanger | Mock API responses, validate pipeline logic | Real mmnr connection, Jan 2026 data with illegal events |

### skipif Conditions

| Task | Conditions | Rationale |
| ---- | ---------- | --------- |
| (all tasks) | any_is_empty_df, any_dependency_skipped | Global default |
| event_point_layers | + all_geometry_are_none | Skip if no valid geometries |
| map_widget | never | Always create placeholder |
| table_widget | never | Always create placeholder |
| persist_sitrep_csv | never | Always export even if empty |
| sitrep_report | never | Always generate report |

### Validation Approach

- Verify get_events returns data for Jan 2026 illegal event types
- Verify sitrep_table produces expected columns (Date, Type, Comments, Number of People, Place of Origin, Sex, Details)
- Verify point map renders with colored markers by event type
- Verify table widget displays SITREP data
- Verify DOCX report contains map images and sitrep table
- Verify CSV export is written

## Dashboard Layout

### layout.json Draft

```json
[
    {
        "i": "0",
        "x": 0, "y": 0,
        "w": 10, "h": 12,
        "minW": 5, "minH": 10,
        "widget_id": 0,
        "static": false
    },
    {
        "i": "1",
        "x": 0, "y": 12,
        "w": 10, "h": 10,
        "minW": 4, "minH": 8,
        "widget_id": 1,
        "static": false
    }
]
```

- Widget 0: Illegal Events Map (full width, top)
- Widget 1: SITREP Summary Table (full width, below map)

## Implementation Plan

### Tasks

- [ ] Task 1: Bootstrap illegal event data from EarthRanger
  - Action: Run `wt-download-events` with illegal event params, save as `resources/mock-data/illegal_events.parquet`

- [ ] Task 2: Scaffold repo and create spec.yaml with Phase 2 load_df
  - File: `spec.yaml`
  - Action: Full pipeline from load_df through dashboard

- [ ] Task 3: Create DOCX report template
  - File: `resources/templates/mt_sitrep_report_template.docx`
  - Action: Template with placeholders for report_date, event_maps (grouped images), sitrep_table (table)

- [ ] Task 4: Create test-cases.yaml and layout.json
  - Files: `test-cases.yaml`, `layout.json`
  - Action: Base test case with mock_io, layout for 2 widgets

- [ ] Task 5: Compile, test, validate outputs
  - Action: Compile, run tests, visually inspect map/table/report

- [ ] Task 6: Configure rjsf-overrides
  - File: `spec.yaml` (rjsf-overrides section)
  - Action: Set defaults for event types, bounding box, base map

- [ ] Task 7: Reconnect EarthRanger API (Phase 3)
  - Files: `spec.yaml`, `test-cases.yaml`
  - Action: Restore get_events, add integration test case

- [ ] Task 8: Publish ecoscope-workflows-ext-mt
  - Action: Build and publish ext-mt to `repo.prefix.dev/ecoscope-workflows-custom/`

### Acceptance Criteria

- [ ] AC 1: Given illegal event data for Jan 2026, when the workflow runs, then point map shows colored markers by event type (arrest, snare, poacher camp) with legend and tooltips
- [ ] AC 2: Given illegal events with suspect details, when processed by sitrep_table, then table shows Date, Type, Comments, Number of People, Place of Origin, Sex, Details
- [ ] AC 3: Given sitrep data, when rendered as table widget, then dashboard shows both map and table widgets
- [ ] AC 4: Given all outputs, when DOCX report is generated, then report contains event maps and SITREP summary table
- [ ] AC 5: Given no illegal events for the time range, when the workflow runs, then skipif conditions prevent errors and exports/report are still generated
- [ ] AC 6: Given the rjsf form, when rendered, then event types default to arrest/snare/poacher_camp and bounding box defaults to Mara Triangle

## Additional Context

### Dependencies

- ecoscope-workflows-core >= 0.22.18
- ecoscope-workflows-ext-ecoscope >= 0.22.18
- ecoscope-workflows-ext-custom >= 0.0.41
- ecoscope-workflows-ext-mt >= 0.0.1 (must be published before workflow release)
- DOCX template hosted on GitHub (wildlife-dynamics/mt-sitrep)

### Notes

- `sitrep_table` uses fully-qualified name (`ecoscope_workflows_ext_mt.tasks.sitrep_table`) since it's in the mt extension
- `create_docx` uses fully-qualified name (`ecoscope_workflows_ext_custom.tasks.results.create_docx`)
- ext-mt is installed as editable pip dependency locally via ecoscope-hub/environment-setup/pixi.toml — compiles and tests work locally without publishing
- ext-mt must be published to `repo.prefix.dev/ecoscope-workflows-custom/` channel before workflow release
- Consolidates three previously separate workflows: mt_illegal_event_download, mt_sitrep_table, mt_sitrep_map
- No groupers — all events rendered in a single map and single table (no split/merge pattern)
- All widget and map tasks operate on single DataFrames (no mapvalues)
