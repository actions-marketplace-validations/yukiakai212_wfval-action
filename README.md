# wfval

Generate GitHub Actions step outputs from path and URL operations without shell string concatenation, `npx`, or a separate Node setup step.

## Usage

```yaml
steps:
  - uses: actions/checkout@v6

  - id: values
    uses: yukiakai212/wfval-action@v1
    with:
      commands: |
        path join dist assets --name artifact_path
        url https://some:400/path --format 'port: {port} with host: {host}' --name endpoint

  - run: |
      echo "${{ steps.values.outputs.artifact_path }}"
      echo "${{ steps.values.outputs.endpoint }}"
```

The example creates these outputs:

```txt
artifact_path=dist/assets
endpoint=port: 400 with host: some
```

Every command must include `--name <output-name>`.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `commands` | Yes | — | One wfval command per line. |
| `working-directory` | No | `.` | Working directory for every command. Relative paths resolve from `GITHUB_WORKSPACE`. |

Blank lines and lines starting with `#` are ignored.

## Path commands

```txt
path <operation> [arguments...] --name <output-name>
```

Supported operations:

- `basename`
- `dirname`
- `join`
- `normalize`
- `relative`
- `resolve`
- `extname`

```yaml
with:
  commands: |
    path join dist assets --name artifact_path
    path relative src src/components --name component_path
    path resolve packages/action --name action_directory
```

Path operations use `pathe`, so returned paths consistently use `/`.

## URL commands

```txt
url <url> [base-url] [--format <template>] --name <output-name>
```

The default template is `{url}`.

```yaml
with:
  commands: |
    url https://some:400/path --name normalized_url
    url /api https://example.com --format '{origin}{pathname}' --name api_url
    url https://some:400/path --format 'port: {port} with host: {host}' --name endpoint
```

Available template values:

```txt
hash
hostname
host
href
origin
password
pathname
port
protocol
search
url
```

`{url}` returns `URL#toString()`.

## Command parsing

Commands are parsed as argument arrays, not executed through a shell. Quote an argument when it contains whitespace:

```yaml
with:
  commands: |
    path join "directory with spaces" dist --name artifact_path
```

Shell variables are not expanded. Use GitHub expressions when a workflow value is needed:

```yaml
with:
  commands: |
    path join "${{ github.workspace }}" dist --name artifact_path
```

`--output-format` is reserved by the action. The action always forces JSON internally so it can validate command outputs safely.

## Failure behavior

The action fails when:

- no executable command is provided;
- a command fails;
- a command does not produce a named output;
- a command returns invalid output;
- multiple commands produce the same output name.

Commands run sequentially. Outputs are written only after every command succeeds.
