# Next Steps

## Overview

The solution build produced no errors across all five projects after transformation:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the migration to cross-platform .NET was completed without introducing any compilation errors. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run the following commands from the solution root to confirm a clean restore and build:

```bash
dotnet restore
dotnet build --configuration Release
```

Verify that the output shows no errors or warnings that could indicate runtime issues.

---

## 2. Run the Unit Tests

Execute the test project to confirm that existing test coverage passes under the new framework:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any failed tests that previously passed
- Any skipped tests that may indicate incompatible test attributes or dependencies
- Any runtime exceptions that do not surface as build errors

---

## 3. Verify Data Layer Functionality

The `Bookstore.Data` project likely contains database access logic (e.g., Entity Framework Core migrations and DbContext configuration). Perform the following checks:

- Confirm the correct EF Core provider is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql`, or `Sqlite`).
- If using EF Core migrations, run:

```bash
dotnet ef migrations list --project app/Bookstore.Data
dotnet ef database update --project app/Bookstore.Data
```

- Confirm connection strings in `appsettings.json` or environment variables are correctly configured for the target environment.

---

## 4. Validate the Web Application at Runtime

Start the web application locally to verify runtime behavior:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following:
- The application starts without runtime exceptions.
- All routes and pages load as expected.
- Any authentication or authorization middleware is functioning correctly.
- Static files are served properly.

---

## 5. Review the CDK Project

The `Bookstore.Cdk` project is likely an AWS CDK infrastructure project. Verify the following:

- All NuGet dependencies (e.g., `Amazon.CDK.Lib`) are restored and compatible with the target .NET version.
- Run a CDK synthesis to confirm the infrastructure definitions compile and resolve correctly:

```bash
dotnet run --project app/Bookstore.Cdk/Bookstore.Cdk.csproj
```

Or, if using the CDK CLI:

```bash
cdk synth
```

Review the synthesized CloudFormation template for correctness.

---

## 6. Check for Removed or Changed APIs

Even without build errors, some .NET APIs behave differently across versions. Review the following areas manually:

- Any use of `System.Web` namespaces, which are not available in cross-platform .NET. These should have been replaced with `Microsoft.AspNetCore` equivalents.
- Any platform-specific code paths (e.g., Windows registry access, COM interop) that may fail on non-Windows environments.
- Configuration loading (e.g., `ConfigurationManager` replaced by `IConfiguration`).

---

## 7. Perform a Smoke Test Against a Staging Environment

Before deploying to production, deploy to a staging environment and perform a manual or automated smoke test covering:

- Core user workflows (e.g., browsing books, placing orders)
- Database read and write operations
- Any external service integrations (e.g., payment providers, email services)

---

## 8. Deploy to Production

Once validation in staging is complete:

1. Publish the web application:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

2. Deploy the published output to your target hosting environment (e.g., AWS Elastic Beanstalk, App Service, or a self-hosted server).

3. Deploy infrastructure changes using the CDK project if applicable:

```bash
cdk deploy
```

4. Monitor application logs after deployment for any runtime errors that did not appear during local testing.