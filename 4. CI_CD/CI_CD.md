**CI**

Om ervoor te zorgen dat de testen automatisch worden uitgevoerd wanneer er wordt gemerged naar de main branch, is een CI workflow toegevoegd. Dit staat voor Continuous integration.

Dit is een simpele workflow die de applicatie bouwt en de tests uitvoerd. Voor een uitgebreidere analyse heb ik een Sonarcloud workflow toegevoegd.

    name: .NET

    on:
    push:
        branches: [ "main" ]
    pull_request:
        branches: [ "main" ]

    jobs:
    build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: Test
      run: dotnet test --no-build --verbosity normal

De reden dat deze build gebaseerd is op Ubuntu is omdat de meeste Docker containers op Linux zijn gebaseerd.

**Sonarcloud**

Sonarcloud is een service waarmee je code quality analysis kunt uitvoeren. Wanneer er naar de main branch wordt gepushed, ontvange je op sonarcloud een rapport met een analyse.

De workflow ziet er als volgt uit:

    name: SonarCloud
    on:
    push:
        branches:
        - main
    pull_request:
        types: [opened, synchronize, reopened]
    jobs:
    build:
        name: Build and analyze
        runs-on: windows-latest
        steps:
        - name: Set up JDK 17
            uses: actions/setup-java@v3
            with:
            java-version: 17
            distribution: 'zulu' # Alternative distribution options are available.
        - uses: actions/checkout@v3
            with:
            fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
        - name: Cache SonarCloud packages
            uses: actions/cache@v3
            with:
            path: ~\sonar\cache
            key: ${{ runner.os }}-sonar
            restore-keys: ${{ runner.os }}-sonar
        - name: Cache SonarCloud scanner
            id: cache-sonar-scanner
            uses: actions/cache@v3
            with:
            path: .\.sonar\scanner
            key: ${{ runner.os }}-sonar-scanner
            restore-keys: ${{ runner.os }}-sonar-scanner
        - name: Install SonarCloud scanner
            if: steps.cache-sonar-scanner.outputs.cache-hit != 'true'
            shell: powershell
            run: |
            New-Item -Path .\.sonar\scanner -ItemType Directory
            dotnet tool update dotnet-sonarscanner --tool-path .\.sonar\scanner
        - name: Build and analyze
            env:
            GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
            SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
            shell: powershell
            run: |
            .\.sonar\scanner\dotnet-sonarscanner begin /k:"JaimySemester3_BackEnd" /o:"jaimysemester3" /d:sonar.token="${{ secrets.SONAR_TOKEN }}" /d:sonar.host.url="https://sonarcloud.io"
            dotnet build
            dotnet test --collect "Code Coverage"
            .\.sonar\scanner\dotnet-sonarscanner end /d:sonar.token="${{ secrets.SONAR_TOKEN }}"

In deze workflow heb ik ook geprobeerd om de code coverage op te vragen, maar dit is nog niet gelukt. Hiervoor moet ik via de terminal van mijn applicatie een code coverage report opvragen, en dit doorgeven aan sonarcloud.

