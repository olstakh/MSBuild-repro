# MSBuild-repro
Repro of console project failing to run  if reference project has TargetPathWithTargetPlatformMoniker

# Steps

1. Run `dotnet build --no-incremental`
2. Run `dotnet run --project .\Console\Console.csproj`

Observe the failure

1. Comment out line 12 in `ClassLib2.csproj`, which is: `<GetTargetPathDependsOn>$(GetTargetPathDependsOn);GetDependencyTargetPaths</GetTargetPathDependsOn>`
2. Repeat the steps
3. Observe no failure
