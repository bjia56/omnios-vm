# Run GitHub CI in OmniOS ![Test](https://github.com/vmactions/omnios-vm/workflows/Test/badge.svg)

Use this action to run your CI in OmniOS.

The github workflow only supports Ubuntu, Windows and MacOS. But what if you need to use OmniOS?

This action is to support OmniOS.


Sample workflow `test.yml`:

```yml

name: Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    name: A job to run test in OmniOS
    env:
      MYTOKEN : ${{ secrets.MYTOKEN }}
      MYTOKEN2: "value2"
    steps:
    - uses: actions/checkout@v4
    - name: Test in OmniOS
      id: test
      uses: vmactions/omnios-vm@v1
      with:
        envs: 'MYTOKEN MYTOKEN2'
        usesh: true
        prepare: |
          pkg  install socat

        run: |
          if [ -n "test" ]; then
            echo "false"
          fi
          if [ "test" ]; then
            echo "test"
          fi
          pwd
          ls -lah
          whoami
          env
          psrinfo -vp
          cat /etc/release
          psrinfo -v
          echo "::memstat" | mdb -k
          echo "OK"




```


The latest major version is: `v1`, which is the most recommended to use. (You can also use the latest full version: `v1.0.1`)  


If you are migrating from the previous `v0`, please change the `runs-on: ` to `runs-on: ubuntu-latest`


The `envs: 'MYTOKEN MYTOKEN2'` is the env names that you want to pass into the vm.

The `run: xxxxx`  is the command you want to run in the vm.

The env variables are all copied into the VM, and the source code and directory are all synchronized into the VM.

The working dir for `run` in the VM is the same as in the Host machine.

All the source code tree in the Host machine are mounted into the VM.

All the `GITHUB_*` as well as `CI=true` env variables are passed into the VM.

So, you will have the same directory and same default env variables when you `run` the CI script.



The code is shared from the host to the VM via `rsync`, you can choose to use to `sshfs` share code instead.


```

...

    steps:
    - uses: actions/checkout@v4
    - name: Test
      id: test
      uses: vmactions/omnios-vm@v1
      with:
        envs: 'MYTOKEN MYTOKEN2'
        usesh: true
        sync: sshfs
        prepare: |
          pkg  install socat



...


```


When using `rsync`,  you can define `copyback: false` to not copy files back from the VM in to the host.


```

...

    steps:
    - uses: actions/checkout@v4
    - name: Test
      id: test
      uses: vmactions/omnios-vm@v1
      with:
        envs: 'MYTOKEN MYTOKEN2'
        usesh: true
        sync: rsync
        copyback: false
        prepare: |
          pkg  install socat



...


```



You can add NAT port between the host and the VM.

```
...
    steps:
    - uses: actions/checkout@v4
    - name: Test
      id: test
      uses: vmactions/omnios-vm@v1
      with:
        envs: 'MYTOKEN MYTOKEN2'
        usesh: true
        nat: |
          "8080": "80"
          "8443": "443"
          udp:"8081": "80"
...
```


The default memory of the VM is 6144MB, you can use `mem` option to set the memory size:

```
...
    steps:
    - uses: actions/checkout@v4
    - name: Test
      id: test
      uses: vmactions/omnios-vm@v1
      with:
        envs: 'MYTOKEN MYTOKEN2'
        usesh: true
        mem: 4096
...
```



It uses [the OmniOS r151048](conf/default.release.conf) by default, you can use `release` option to use another version of OmniOS:

```
...
    steps:
    - uses: actions/checkout@v4
    - name: Test
      id: test
      uses: vmactions/omnios-vm@v1
      with:
        release: "r151046"
...
```

All the supported releases are here: OmniOS  r151046, r151048, test.releases [See all here](conf)


# Under the hood

We use Qemu and Libvirt to run the OmniOS VM.




# Upcoming features:

1. Runs on MacOS to use cpu accelaration.
2. Support ARM and other architecture.




