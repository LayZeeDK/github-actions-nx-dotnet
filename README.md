Nx workspace using nx-dotnet. Contains a GitHub Actions CI workflow.

# How to generate this workspace

## Prerequisites

- Node.js
- PNPM
- .NET CLI

## Create Nx workspace

```powershell
# Install the Nx workspace generator
pnpm install --global create-nx-workspace
# Generate a blank Nx workspace
pnpm init nx-workspace github-actions-nx-dotnet --preset=empty --pm=pnpm --npm-scope=dotnet --no-nx-cloud
```

## Configure Nx workspace

```powershell
# Install the "json" utility
npm install --global json
# Set the base branch to "main"
json -I -f nx.json -e "this.affected.defaultBase = 'main';"
```

## Add .NET capability

```powershell
# Install the Nx CLI
npm install --global @nrwl/cli
# Add nx-dotnet
pnpm add --save-dev @nx-dotnet/core
# Initialize nx-dotnet
nx generate @nx-dotnet/core:init
```

## Configure Nx generator defaults

```powershell
# Prefer nx-dotnet generators
json -I -f workspace.json -e "this.cli.defaultCollection = '@nx-dotnet/core';"
# Set defaults for nx-dotnet's "app" and "lib" generators
json -I -f workspace.json -e "this.generators = { '@nx-dotnet/core:app': { language: 'C#', tags: 'type:api', template: 'webapi', testTemplate: 'xunit' }, '@nx-dotnet/core:lib': { language: 'C#', template: 'classlib', testTemplate: 'xunit' } };"
```

## Create web API project

```powershell
# Generate web API and testing projects
nx generate app my-web-api
# Tag testing project with "type:test"
json -I -f nx.json -e "this.projects['my-web-api-test'].tags = ['type:test'].concat(this.projects['my-web-api-test'].tags.slice(1));"
# Set my-web-api as default Nx project
json -I -f workspace.json -e "this.defaultProject = 'my-web-api';"
```

## Generate GitHub Actions CI workflow

```powershell
# Install GitHub Actions .NET template
dotnet new -i TimHeuer.GitHubActions.Templates::1.0.5
# Generate GitHub Actions CI workflow
dotnet new workflow
```

## Use Nx for Build job

1. Remove the _Restore_ step from `.github/workflows/github-actions-nx-dotnet.yaml`.
1. Add _Setup Node.js_ step after _Setup .NET Core SDK_ step:
   ```yml
   - name: Setup Node.js
     uses: actions/setup-node@v1
     with:
       node-version: 14.x
   - name: Install PNPM
     run: npm install --global pnpm
   - name: Install Nx dependencies
     run: pnpm install
   ```
1. Change the `run` command of the _Build_ step to:
   ```
   pnpm build
   ```
1. Change the `run` command of the _Test_ step to:
   ```
   pnpm test
   ```
