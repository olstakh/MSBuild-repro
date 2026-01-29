As to why this setup is used. In my case it's related to Roslyn code analysis project setup.

We currently have CodeAnalysis project (with regular Roslyn analyzers, source generators, etc) that we hook to regular projects via OutputItemType="Analyzer"

We also have CodeAnalysis.Annotations project, which is vanilla classlib, containing annotation attributes that analyzers / source generators use (i.e. [GenerateAwesomeCode]). We also hook it to all the projects via regular project reference, so they can use those attributes and get awesome code generated.

Obviously we have CodeAnalysis.Tests project that references both of them and contains tests for analyzers / source generators.

CodeAnalysis project depends on CodeAnalysis.Annotations classlib, so it can use those attributes to do its magic. I know that it's possible to convert everything to strings instead (i.e. "GenerateAwesomeCodeAttribute" instead of "nameof(GenerateAwesomeCode)", etc) and remove the project reference. I am trying to avoid this lose coupling, since it will become mainteinability problem.

So for CodeAnalysis project to build - we need regular project reference to CodeAnalysis.Annotations.
But it also needs CodeAnalysis.Annotations library at runtime (i.e. during building other projects, when analyzers execute), and according to this suggestion - we have to add "TargetPathWithTargetPlatformMoniker" target like described above. [Reference local projects in Source Generator · dotnet/roslyn · Discussion #47517](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fgithub.com%2Fdotnet%2Froslyn%2Fdiscussions%2F47517%23discussioncomment-64145&data=05%7C02%7Colstakh%40microsoft.com%7C184e7edfdff54051b9ca08de4718c328%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C639026368984818428%7CUnknown%7CTWFpbGZsb3d8eyJFbXB0eU1hcGkiOnRydWUsIlYiOiIwLjAuMDAwMCIsIlAiOiJXaW4zMiIsIkFOIjoiTWFpbCIsIldUIjoyfQ%3D%3D%7C0%7C%7C%7C&sdata=S%2FADwWIFx2zPhD0OjjrLxt4o9jIIZ%2B10h2jqwqOAYOM%3D&reserved=0)

Everything is fine and dandy. But when we are moving CodeAnalysis.Tests project from xunit.v2 to xunit.v3 - test project becomes standalone console project and this issue manifests (i.e. it cannot locate any types from CodeAnalysis.Annotations at runtime). I believe since xunit.v2 was a classlib - testhost.exe was used to host the tests and it contains custom assembly resolver, which was probably hiding this issue. Now in a regular console project it doesn't work

So, to tie this to the small repro repository from above:

Console.csproj = CodeAnalysis.Tests.csproj (tests for analyzers)
Class1.csproj = CodeAnalysis.Annotations.csproj (attributes used by analyzers)
Class2.csproj = CodeAnalysis.csproj (analyzers themselves)

I'd be very curious to understand where's our mistake in the above project setups or whether it's by design from msbuild point of view and why.