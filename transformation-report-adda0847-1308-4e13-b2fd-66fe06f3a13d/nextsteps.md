# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Summary

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas that may cause runtime issues.

---

## 3. Run Unit Tests

Execute the test project to verify that existing business logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output carefully. Any failing tests should be investigated to determine whether they indicate a regression introduced during the migration or a pre-existing issue.

---

## 4. Validate the Data Layer

Since `Bookstore.Data` likely handles database access, verify the following:

- **Connection strings** in `appsettings.json` or environment-specific configuration files are correctly set for the target environment.
- If Entity Framework Core is in use, confirm that migrations are up to date by running:

```bash
dotnet ef migrations list --project app/Bookstore.Data
```

If there are pending migrations or the migration history looks incorrect, generate a new migration to capture any model changes:

```bash
dotnet ef migrations add PostMigrationCheck --project app/Bookstore.Data
dotnet ef database update --project app/Bookstore.Data
```

---

## 5. Run and Validate the Web Application Locally

Start the web application locally to confirm it runs without runtime errors:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Manually navigate through the key areas of the application and verify:

- Pages load without errors.
- Data is read from and written to the database correctly.
- Authentication and authorization flows work as expected, if applicable.
- Any static assets (CSS, JavaScript) are served correctly.

Check the console output and application logs for any runtime exceptions or warnings.

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project likely defines infrastructure. Review it to ensure:

- Any environment-specific values (account IDs, regions, resource names) are correctly configured for the target deployment environment.
- The CDK project targets a compatible version of the AWS CDK library for .NET.

Synthesize the CDK stack to validate the infrastructure definition:

```bash
cd app/Bookstore.Cdk
cdk synth
```

Resolve any synthesis errors before proceeding to deployment.

---

## 7. Deploy the Infrastructure and Application

Once local validation is complete, deploy the infrastructure using the CDK project:

```bash
cdk deploy
```

After the infrastructure is provisioned, publish and deploy the web application to the target environment:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Copy the contents of the `./publish` directory to the appropriate hosting environment and confirm the application starts correctly in that environment.

---

## 8. Post-Deployment Validation

After deployment, perform the following checks:

- Confirm the application is reachable at the expected URL.
- Verify database connectivity from the deployed environment.
- Review application logs for any errors that did not surface during local testing.
- Run any available smoke tests or integration tests against the deployed environment.