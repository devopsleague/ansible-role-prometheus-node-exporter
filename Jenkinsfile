library(identifier: 'ableton-utils@0.21', changelog: false)
library(identifier: 'groovylint@0.12', changelog: false)
library(identifier: 'python-utils@0.11', changelog: false)


devToolsProject.run(
  setup: { data ->
    Object venv = virtualenv.create('python3.8')
    venv.run('pip install -r requirements-dev.txt')
    data['venv'] = venv
  },
  test: { data ->
    parallel(failFast: false,
      'ansible-lint': {
        String stdout = data.venv.run(
          label: 'ansible-lint',
          returnStdout: true,
          script: 'ansible-lint -c .ansible-lint.yml',
        )

        // If only warnings are found, ansible-lint will exit with code 0 but still write
        // an error summary to stdout. It's not possible to treat warnings as errors, and
        // likely will never be. See:
        // https://github.com/ansible-community/ansible-lint/issues/236
        if (stdout) {
          error 'ansible-lint exited with warnings, check the output of the previous step'
        }
      },
      black: { data.venv.run('black --check .') },
      groovylint: { groovylint.check('./Jenkinsfile') },
      molecule: { data.venv.run('molecule --debug test --all') },
    )
  },
  deployWhen: { devToolsProject.shouldDeploy(defaultBranch: 'main') },
  deploy: { data ->
    String versionNumber = readFile('VERSION').trim()
    version.tag(versionNumber)
  },
)
