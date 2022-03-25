# .github-workflows-dependencies-dotnet-whatsnew.2.1.2.nupkg
# This is a basic workflow to help you get started with Actions

name: 'generate what''s new article'

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the default branch
  schedule:
  - cron: '0 0 1 * *' # The first of every month
  workflow_dispatch:
    inputs:
      reason:
        description: 'The reason for running the workflow'
        required: true
        default: 'Manual run'

env:
  DOTNET_VERSION: '6.0.x' # set this to the dot net version to use

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "create-what-is-new"
  create-what-is-new:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
      - name: Setup .NET
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      # Runs a single command using the runners shell
      - name: 'Print manual run reason'
        if: ${{ github.event_name == 'workflow_dispatch' }}
        run: |
          echo "Reason: ${{ github.event.inputs.reason }}"

      # Print dotnet info
      - name: Display .NET info
        run: dotnet --info

      # Install dotnet-whatsnew global tool
      - name: Install dotnet-whatsnew tool
        run: |
          dotnet tool install --global --add-source ./.github/workflows/dependencies/ dotnet-whatsnew

      # Run dotnet-whatsnew tool
      - name: Generate what's new
        id: dotnet-whatsnew
        env:
          GitHubKey: ${{ secrets.GITHUB_TOKEN }}
          OspoKey: ${{ secrets.OSPO_KEY }}
        run: |
          ./.github/workflows/dependencies/run-dotnet-whatsnew.sh -o dotnet -r AspNetCore.Docs -s './aspnetcore/whats-new'

      # Create the PR for the new article
      - name: create-pull-request
        uses: peter-evans/create-pull-request@v3.10.0
        with:
          title: 'What''s new article'
          commit-message: 'Bot 🤖 generated "What''s new article"'
          body: 'Automated creation of What''s new article.'
