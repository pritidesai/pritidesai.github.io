# Task Step: From Command to Script

All of my Steps under Tekton Tasks had `command` with `bash` but few months back (to be precise October 2019), a new field `script` was added to the `step` to simplify its usage. I recently started updating my steps and thought of documenting the difference here:

Tekton Pipeline Issue # 781 was fixed with Tekton Pipeline PR # 1432

Before

```
- name: echo-hello-world
  image: ubuntu
  command:
    - bash
  args:
    - -c
    - |
      echo "Hello World!"
```

After

```
- name: echo-hello-world
  image: ubuntu
  script: |
    #!/usr/bin/env bash
    set -xe
    echo "Hello World!"
```

Enjoy! 👍
