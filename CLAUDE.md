# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Bitrise CI/CD step written in Go that deploys build artifacts, test reports, and pipeline intermediate files to Bitrise.io. The step handles uploading various file types (APK, AAB, IPA, XcArchive, and generic files) to the Bitrise platform for distribution and sharing.

## Development Commands

### Core Commands
```bash
# Run tests
bitrise run test

# Run complete CI pipeline (includes downloading sample artifacts and running tests)  
bitrise run ci

# Generate README from step.yml
bitrise run generate_readme
```

### Go Commands
```bash
# Build the step
go build -o deploy-to-bitrise-io

# Run tests with coverage
go test ./... -cover

# Run specific test files
go test ./main_test.go
go test ./uploaders/...
go test ./report/...
```

## Architecture

### Core Components

- **main.go** - Entry point and orchestration logic for the step
- **uploaders/** - Handles uploading different artifact types (APK, AAB, IPA, XcArchive, generic files)  
- **deployment/** - Manages deployable items collection and intermediate file handling
- **report/** - Handles test result parsing/uploading and HTML report deployment
- **test/** - Parses various test result formats (JUnit XML, XCResult) 
- **fileredactor/** - Redacts secrets from files before deployment

### Key Workflows

1. **File Collection**: Collects deployable files from `deploy_path` and processes pipeline intermediate files
2. **File Redaction**: Redacts secrets from specified files using environment variable values  
3. **Artifact Upload**: Uploads files concurrently using specialized uploaders for each file type
4. **Test Results**: Parses and uploads test results to the Test Reports addon
5. **HTML Reports**: Deploys HTML reports from `BITRISE_HTML_REPORT_DIR`

### File Type Handling

The step has specialized uploaders for:
- **APK/AAB**: Android artifacts with metadata parsing using bundletool
- **IPA**: iOS apps with metadata extraction  
- **XcArchive**: Zipped Xcode archives (.xcarchive.zip)
- **Generic files**: All other file types

### Configuration

Main configuration is defined in `step.yml` with inputs for:
- `deploy_path` - Files/directory to deploy
- `pipeline_intermediate_files` - Files to share between pipeline workflows
- `files_to_redact` - Files to redact secrets from
- Notification settings (user groups, emails)
- Upload concurrency control via `BITRISE_DEPLOY_UPLOAD_CONCURRENCY`

### Testing

- Uses `bitrise.yml` for test orchestration
- Downloads sample artifacts from a separate repository for testing
- Includes comprehensive test scenarios for different file types and deployment modes
- Mock objects available in `mocks/` directories for unit testing

### Dependencies

- Built with Go 1.20
- Uses Bitrise's go-utils, go-steputils, and go-xcode libraries
- Android parsing via go-android library with bundletool integration
- HTTP retries with hashicorp/go-retryablehttp