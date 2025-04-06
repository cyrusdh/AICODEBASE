AI Project GitHub Archival System
Project Summary
Project Name: AI Project GitHub Archival System

Description: A streamlined system for archiving AI-assisted projects to GitHub. Consists of PowerShell scripts and templates that allow developers to save complete AI-generated projects as a single document, manually extract individual files, and push them to a structured GitHub repository.

Programming Languages: PowerShell, Batch

Key Features:

Single-document export of AI conversations in markdown format
Clear file delineation for easy manual extraction
GitHub repository integration
Standardized project documentation format
Minimal manual intervention required
Lessons Learned:

Special characters in paths can cause PowerShell script errors
Markdown format is more reliable than custom file markers
Manual extraction is simpler than automated regex-based parsing
Preserving project context between AI sessions requires structured documentation
Using standard markdown improves readability and reduces extraction issues
Usage Examples:

vbnet
1. Create project files with AI assistance  
2. Request project archival using the template prompt  
3. Save the markdown response as a reference document  
4. Extract each file by copying code blocks to individual files  
5. Commit and push to GitHub using standard git commands  
File: AI-Project-Push.ps1
powershell
param(  
    [Parameter(Mandatory=$true)]  
    [string]$ProjectName,  
    
    [Parameter(Mandatory=$true)]  
    [string]$ProjectDescription,  
    
    [Parameter(Mandatory=$true)]  
    [string]$RepoUrl  
)  

$tempDir = Join-Path $env:TEMP "ai_project_push_$(Get-Random)"  

Write-Host "Starting AI project push for: $ProjectName" -ForegroundColor Cyan  
Write-Host "Description: $ProjectDescription" -ForegroundColor Cyan  
Write-Host "=================================" -ForegroundColor Cyan  

# Verify project directory exists  
$projectDir = Join-Path $PWD $ProjectName  
if (!(Test-Path $projectDir)) {  
    Write-Host "Project directory not found: $projectDir" -ForegroundColor Red  
    exit 1  
}  

# Check if files exist in project directory  
$files = Get-ChildItem -Path $projectDir -Recurse -File  
if ($files.Count -eq 0) {  
    Write-Host "No files found in project directory! Aborting." -ForegroundColor Red  
    exit 1  
}  

# Clone the repository  
Write-Host "Cloning repository..." -ForegroundColor Cyan  
New-Item -ItemType Directory -Path $tempDir -Force | Out-Null  
git clone $RepoUrl $tempDir  

if ($LASTEXITCODE -ne 0) {  
    Write-Host "Failed to clone repository!" -ForegroundColor Red  
    exit 1  
}  

# Create projects directory if it doesn't exist  
$projectsDir = Join-Path $tempDir "projects"  
if (!(Test-Path $projectsDir)) {  
    New-Item -ItemType Directory -Path $projectsDir -Force | Out-Null  
}  

# Create target project directory  
$targetProjectDir = Join-Path $projectsDir $ProjectName  
if (Test-Path $targetProjectDir) {  
    Write-Host "Project already exists in repository. Files will be updated." -ForegroundColor Yellow  
    Remove-Item -Path $targetProjectDir -Recurse -Force  
}  
New-Item -ItemType Directory -Path $targetProjectDir -Force | Out-Null  

# Copy project files  
Write-Host "Copying project files..." -ForegroundColor Cyan  
Copy-Item -Path "$projectDir\*" -Destination $targetProjectDir -Recurse  

# Add, commit, and push  
Set-Location $tempDir  
git add .  
git commit -m "Add project: $ProjectName - $ProjectDescription"  
git push  

if ($LASTEXITCODE -eq 0) {  
    Write-Host "Project successfully pushed to repository!" -ForegroundColor Green  
} else {  
    Write-Host "Failed to push to repository. Please check your permissions." -ForegroundColor Red  
}  

# Clean up  
Set-Location $PWD  
Remove-Item -Path $tempDir -Recurse -Force -ErrorAction SilentlyContinue  

Write-Host "Done!" -ForegroundColor Green  
File: AI-Project-Push.bat
batch
@echo off  
if "%~2"=="" (  
    echo Usage: %0 ProjectName "Project Description" RepoUrl  
    exit /b 1  
)  
if "%~3"=="" (  
    echo Usage: %0 ProjectName "Project Description" RepoUrl  
    exit /b 1  
)  

echo INSTRUCTIONS:  
echo 1. Work with the AI to create your project files  
echo 2. Save all files in the '%~1' folder  
echo 3. When ready, press any key to push to GitHub...  

mkdir "%~1" 2>nul  
dir /b /s "%~1\*.*" | find /v "" > nul 2>&1  
if errorlevel 1 (  
    echo No files found in project directory! Aborting.  
    exit /b 1  
)  

powershell -ExecutionPolicy Bypass -File "%~dp0AI-Project-Push.ps1" -ProjectName "%~1" -ProjectDescription "%~2" -RepoUrl "%~3"  
File: README.md
markdown
# AI Project GitHub Archival System  

A streamlined system for archiving AI-assisted projects to GitHub. This tool allows developers to save complete AI-generated projects as a single markdown document, easily extract individual files, and push them to a structured GitHub repository.  

## Overview  

This system solves the problem of preserving knowledge and code from AI-assisted development sessions. It enables:  

- Capturing entire projects from AI conversations in markdown format  
- Clearly delineating files for easy manual extraction  
- Maintaining a structured repository of AI-assisted projects  
- Referencing past projects in future AI sessions  

## Files Included  

- **AI-Project-Push.ps1**: PowerShell script for GitHub operations  
- **AI-Project-Push.bat**: Batch wrapper for easier command-line usage  

## Requirements  

- Windows with PowerShell 5.0+  
- Git command-line tools installed and configured  
- GitHub repository access  

## Usage  

1. At the end of an AI-assisted project, ask the AI to format the project using the archival template  
2. Save the entire markdown response as your project reference document  
3. Create a folder for your project and extract each file by copying code blocks to individual files  
4. Run the script:  

.\AI-Project-Push.bat ProjectName "Project description" https://github.com/username/repository.git

markdown

5. The script will:  
   - Verify files exist in the project directory  
   - Clone your GitHub repository  
   - Copy files to the appropriate location  
   - Commit and push changes  

## Archival Template  

Here's the standardized prompt to use at the end of any AI project:  

I want to archive our project to GitHub. Please help me organize all the code and files we've created into a well-formatted markdown document that I can easily reference and extract files from.

PROJECT NAME: [Name of your project]

PROJECT DESCRIPTION:
[Brief description of what the project does and its purpose]

PROGRAMMING LANGUAGE(S): [e.g., JavaScript, Python, PowerShell]

KEY FEATURES:

[Feature 1]
[Feature 2]
[Feature 3]
LESSONS LEARNED:
[What worked well, challenges faced, solutions found]

USAGE EXAMPLES:
[Basic examples of how to use the code]

Please format your response as a clean markdown document with:

A project summary at the top
Each file clearly marked with a ## File: [filename] heading
Code blocks using triple backticks with the appropriate language specified
Any additional documentation or instructions
I'll save this markdown document as my project reference and manually extract each file.

File: Project-Archival-Template.md
markdown
# Project Archival Template  

Use this template at the end of any AI-assisted project to prepare files for GitHub archival:  

I want to archive our project to GitHub. Please help me organize all the code and files we've created into a well-formatted markdown document that I can easily reference and extract files from.

PROJECT NAME: [Name of your project]

PROJECT DESCRIPTION:
[Brief description of what the project does and its purpose]

PROGRAMMING LANGUAGE(S): [e.g., JavaScript, Python, PowerShell]

KEY FEATURES:

[Feature 1]
[Feature 2]
[Feature 3]
LESSONS LEARNED:
[What worked well, challenges faced, solutions found]

USAGE EXAMPLES:
[Basic examples of how to use the code]

Please format your response as a clean markdown document with:

A project summary at the top
Each file clearly marked with a ## File: [filename] heading
Code blocks using triple backticks with the appropriate language specified
Any additional documentation or instructions
I'll save this markdown document as my project reference and manually extract each file.

sql

Simply fill in the bracketed sections with your specific project details when you use it.  