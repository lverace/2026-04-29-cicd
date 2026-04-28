[comment]: # (This presentation was made with markdown-slides)
[comment]: # (Can be found here: https://gitlab.com/da_doomer/markdown-slides)
[comment]: # (Compile this presentation with the command below)
[comment]: # (mdslides slides.md)

[comment]: # (Set the theme:)
[comment]: # (THEME = white)
[comment]: # (CODE_THEME = github)

[comment]: # (controls: true)
[comment]: # (keyboard: true)
[comment]: # (markdown: { smartypants: true })
[comment]: # (hash: false)
[comment]: # (respondToHashChanges: false)

## Continuous Integration and Continuous Delivery (CI/CD) with GitHub Workflows

<br/>

### Andres Rios Tascon

HSF/IRIS-HEP Analysis Reproducibility

April 29, 2026

[comment]: # (!!!)

## What is CI/CD

- Continuous Integration (CI):<br>
  Frequent merging of changes into main with automated quality checks

- Continuous Delivery (CD):<br>
  Code is always in a releasable state; deployment requires a manual trigger

- Continuous Deployment (CD):<br>
  Every passing change is automatically released to production

[comment]: # (!!!)

## What is CI/CD for?

- Running tests (dynamic and static)
  - Check if new changes break any existing functionality
  - Check if code follows formatting guidelines

[comment]: # (||| data-auto-animate)

## What is CI/CD for?

- Building documentation<br>
  - Run Doxygen/Sphinx/etc generate documentation in pdf or html form
  - Check if examples in the documentation work

[comment]: # (||| data-auto-animate)

## What is CI/CD for?

- Build static websites
  - Build and publish a website to GitHub Pages/ReadTheDocs/etc

[comment]: # (||| data-auto-animate)

## What is CI/CD for?

- Generating pull requests for updates and other maintenance
  - Update dependencies
  - Fix typos and coding style

[comment]: # (||| data-auto-animate)

## What is CI/CD for?

- Whatever your project needs!

[comment]: # (!!! data-auto-animate)

## What are the benefits?

- Consistent, controlled environment between runs
- Runs every PR/commit/tag/whatever you choose
- Can't be skipped or forgotten, no contributor setup
- Can run lots of OS's, Python versions, compilers, etc

[comment]: # (!!!)

## Some major CI/CD services

- **Travis CI**: Very popular for years, but not anymore.
- **Jenkins**: A self-host only OSS solution.
- **Circle CI**: The first more “modern” design.
- **GitLab CI**: For years, this was one of the best services. Still very good.
- **Azure Pipelines**: Very modular design is easy to upgrade and maintain.
- **GitHub Actions (GHA)**: Extremely simple and popular. Actions are easy to write and share. The one we will use today.

[comment]: # (!!!)

### CI/CD examples

Let's look at a couple of repositories and see how they use CI/CD

- [Line-Segment Tracking](https://github.com/SegmentLinking/cmssw/pull/245)
- [Uproot](https://github.com/scikit-hep/uproot5/pull/1606)
- [NumPy](https://github.com/numpy/numpy/pull/31040)

[comment]: # (!!!)

### Plan for today

- Go through a bit of background
- Do a hands-on demo
- Test what you learned with a couple of exercises

By the end of the session, you will have written a realistic workflow to test a Python package and another one to deploy a static website.

[comment]: # (!!!)

### A bit of background

<br>

We'll first do a crash course on YAML and exit codes.

[comment]: # (!!!)

### YAML

YAML (YAML Ain’t Markup Language, originally Yet Another Markup Language) is a human-readable data serialization language.

- Easy to read and use
- Very commonly used in CI/CD configuration files.
- File extension is .yml or .yaml

[comment]: # (||| data-auto-animate)

### YAML

Defining scalar values:

```yaml
number-value: 42
boolean-value: true # can also be on or yes
string-value: "Hello world"
another-string: String without quotes
```

[comment]: # (||| data-auto-animate)

### YAML

Defining lists:

```yaml
colors:
  - red
  - green
  - blue

more_colors: [black, white]
```

[comment]: # (||| data-auto-animate)

### YAML

Defining dictionaries:

```yaml
person:
  name: John Smith
  age: 33
  occupation: accountant

same_person: {name: John Smith, age: 33, occupation: accountant}
```

[comment]: # (||| data-auto-animate)

### YAML

Defining multi-line strings:

```yaml
some-text: >
  Multiple
  lines
  of
  text
same-text: "Multiple lines of text\n"
```

[comment]: # (||| data-auto-animate)

### YAML

Defining multi-line strings:

```yaml
some-text: |
  Multiple
  lines
  of
  text
same-text: "Multiple\nlines\nof\ntext\n"
```

[comment]: # (!!! data-auto-animate)

### Exit codes

- Your CI/CD pipeline will check if each command ran successfully and stop if it runs into an error.
- Every time you run a command in a shell there is an exit code that indicates if it ran successfully.
- An exit code of `0` indicates that the command ran successfully, other numbers (usually) indicate an error.
- Sometimes different numbers correspond to different errors.

[comment]: # (||| data-auto-animate)

### Exit codes

```bash [1-4|5-7|8-11]
> echo "hello world" | grep "hello"
hello world
> echo $?
0
> echo "hello world" | grep "python"
> echo $?
1
> grep "pattern" nonexistent_file.txt
grep: nonexistent_file.txt: No such file or directory
> echo $?
2
```

[comment]: # (||| data-auto-animate)

### Exit codes

- CI/CD workflows will typically stop once they encounter a non-zero exit code.
- Sometimes you may need to run a command that might fail, but you want the workflow to proceed.
- Some scripts and binaries don't respect this standard and return non-zero exit codes even when successful.

[comment]: # (||| data-auto-animate)

### Exit codes

Error codes can be ignored using logical or (||)
```bash
> <command_with_failure> || echo "ignore"
ignore
```

It is also useful to use logical and (&&) to run a command only if another one is successful.
```
> <command1> && <command2>
```

[comment]: # (!!! data-auto-animate)

### Setting up GitHub workflows

- Each workflow is configured by a yaml file placed in `.github/workflows`
- Can be set to trigger by a wide variety of events
- Can run your own commands or use actions written by you or third-parties

[comment]: # (||| data-auto-animate)

### Setting up GitHub workflows

This is the basic structure of a workflow file.

```yaml
on: <event or list of events>

jobs:
  job_1:
    name: <name of job>
    runs-on: <type of machine>
    steps:
      - uses: <some third-party action>
      - name: <name of step>
        run: <command>

  job_2:
    runs-on: <type of machine>
    steps:
      - run: <command>
```

[comment]: # (!!! data-auto-animate)

### CI/CD hands-on workshop

Go to the following link:

https://github.com/ariostas-talks/2026-04-29-cicd

[comment]: # (!!! data-auto-animate)

### Exercise 1

Create a new job that does some linting before our matrix of jobs, using `ruff`. You can `pip`-install it, run it with `pipx` (which is pre-installed), or use the `astral-sh/ruff-action` action. It should run `ruff check`, and the testing matrix should not run if this fails.

(In practice, it is more common to use pre-commit for this, but that's a topic for another day)

[comment]: # (!!! data-auto-animate)

### Exercise 2

Create a new workflow that generates these slides and publishes them to GitHub pages. Things you need:

- `actions/checkout`
- `actions/setup-python`
- `markdown-slides` from https://gitlab.com/da_doomer/markdown-slides
- `actions/upload-pages-artifact`
- `actions/deploy-pages`
- Repo > Settings > Pages > Source > GitHub Actions

[comment]: # (||| data-auto-animate)

### Exercise 2

Start by checking out the repo.

```yaml
- name: Check out repo
  uses: actions/checkout@v6
```

[comment]: # (||| data-auto-animate)

### Exercise 2

Then set up Python.

```yaml
- name: Set up Python
  uses: actions/setup-python@v6
  with:
    python-version: "3.11"
```

[comment]: # (||| data-auto-animate)

### Exercise 2

Install `markdown-slides`.

```yaml
- name: Install markdown slides
  run: pip install git+https://gitlab.com/da_doomer/markdown-slides.git
```

[comment]: # (||| data-auto-animate)

### Exercise 2

Generate the slides.

```yaml
- name: Generate slides
  run: mdslides slides.md --output_dir slides
```

[comment]: # (||| data-auto-animate)

### Exercise 2

Upload pages artifact

```yaml
- name: Upload pages artifact
  uses: actions/upload-pages-artifact@v4
  with:
    path: ./slides
```

[comment]: # (||| data-auto-animate)

### Exercise 2

Deploy to GitHub Pages

```yaml
deploy:
  permissions:
    pages: write
    id-token: write
  environment:
    name: github-pages
    url: ${{ steps.deployment.outputs.page_url }}
  runs-on: ubuntu-latest
  needs: build
  steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4
```

[comment]: # (!!! data-auto-animate)

There's many more things to learn, but with this basic knowledge there are lots of things you can do!
