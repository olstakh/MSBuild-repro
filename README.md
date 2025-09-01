# MSBuild-repro
Repro of console project failing to run  if reference project has TargetPathWithTargetPlatformMoniker

# Steps

1. Run `dotnet build --no-incremental`
2. Run `dotnet run --project .\Console\Console.csproj`

Observe the failure

1. Comment out line 12 in `ClassLib2.csproj`, which is: `<GetTargetPathDependsOn>$(GetTargetPathDependsOn);GetDependencyTargetPaths</GetTargetPathDependsOn>`
2. Repeat the steps
3. Observe no failure

Another observation

Current project references are as follows
`Console -> ClassLib1`
`Console -> ClassLib2`
`ClassLib2 -> ClassLib1`

Console project just prints `typeof(ClassLib1).Name`.

Turns out - removing reference from `Console` to `ClassLib` also fixes the issue. Meaning `ClassLib1` can ONLY be used as a transient reference in this setup, NOT as direct reference.