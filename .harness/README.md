# Run Vagrant Ruby tests on Harness CI

This is a fork of [https://github.com/hashicorp/vagrant](hashicorp/vagrant). This project can be used to demonstrate Ruby [Test Intelligence](https://developer.harness.io/docs/category/test-intelligence) in Harness CI pipelines.

This repository contains over 5,000 Ruby rspec tests. Follow these steps to experiment with Ruby Test Intelligence in your [Harness](https://www.harness.io/) account.

## Setting up this pipeline on Harness CI Hosted Builds

1. Create a [GitHub Account](https://github.com) or use an existing account

2. Fork [this repository](https://github.com/harness-community/vagrant/fork) into your GitHub account

3.
    a. If you are new to Harness CI, signup for [Harness CI](https://app.harness.io/auth/#/signup)
      * Select the `Continuous Integration` module and choose the `Starter pipeline` wizard to create your first pipeline using the forked repo from #2.
      * Go to the newly created pipeline and hit the `Triggers`tab. If everything went well, you should see two triggers auto-created. A `Pull Request`trigger and a `Push`trigger. For this exercise, we only need `Pull Request`trigger to be enabled. So, please disable or delete the `Push`trigger.
   
    b. If you are an existing Harness CI user, create a new pipeline to use the cloud option for infrastructure and setup the PR trigger.

4. To enable Ruby Test Intelligence support in your Harness account, check the [Enable TI for Ruby](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/test-intelligence/ti-for-ruby/) documentation.

5. Insert this YAML into your pipeline's `stages` section.

```yaml
      - stage:
          strategy:
            parallelism: 2
          name: test
          identifier: test
          type: CI
          spec:
            cloneCodebase: true
            platform:
              os: Linux
              arch: Amd64
            runtime:
              type: Cloud
              spec: {}
            execution:
              steps:
                - step:
                    type: Action
                    name: Setup Ruby
                    identifier: setup_ruby
                    spec:
                      uses: ruby/setup-ruby@v1.152.0
                      with:
                        ruby-version: "3.2"
                - step:
                    type: Run
                    name: Dependencies
                    identifier: dependencies
                    spec:
                      shell: Sh
                      command: |-
                        apt-get update -y
                        apt -y install libarchive-tools
                - step:
                    type: RunTests
                    name: Run Tests
                    identifier: run_tests
                    spec:
                      language: Ruby
                      buildTool: Rspec
                      testGlobs: "**/test/unit/**/*_test.rb"
                      runOnlySelectedTests: true
                      enableTestSplitting: true
                      preCommand: bundle install
```

5. The repository already contains a GitHub Actions [workflow file](../.github/workflows/demo.yml). You can choose to enable this workflow from the Actions tab on GitHub.

6. Trigger a pipeline that runs all Ruby tests to generate an [initial call graph](https://developer.harness.io/docs/continuous-integration/use-ci/run-tests/set-up-test-intelligence/#generate-the-initial-call-graph).

   Commit a change to [Gemfile](../Gemfile) file (e.g. add a comment or new line) in a branch and open a pull request. This will trigger your pipeline in Harness CI and run all Ruby tests in the repository, which will take about eight minutes. When the pipeline has completed, merge your change to the 'main' branch.

   This sets a "baseline" for test selection in future pipeline executions.

7. Next, make a change to a Ruby file and open a pull request.

   Add a comment to the file [lib/vagrant/plugin/v2/components.rb](../lib/vagrant/plugin/v2/components.rb) in a new branch and open a pull request (see [this example](https://github.com/harness-community/vagrant/pull/3/files)). This will trigger your pipeline in Harness CI as well as GitHub Actions (enabled in step 5). Only the subset of tests which are relevant to this code change will run, split between two stages. However, the GitHub Actions workflow will run all the unit tests for every PR.
