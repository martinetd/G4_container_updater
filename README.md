# Automatically build G4 container update

This repo's github action workflow automatically builds the container `image:latest` from `image/Dockerfile`, and stores it in `container.swu` for automatic update.

## mkswu setup

Setup mkswu if this hasn't been done (mkswu --init, install `initial_setup.swu` on G4 for keys)

## Using swupdate-url

swupdate-url is a basic poller on fixed url for armadillo G4.

### github/repo side setup

On github, navigate to `Settings`, `Secrets` > `Actions`, and create `New repository secrets` as follow:
 - `SWUPDATE_KEY` with the contents of `~/mkswu/swupdate.key`
 - (if required) `SWUPDATE_KEY_PASS` with `pass:key password`
 - (optionally) `SWUPDATE_AES_KEY` with the contents of `~/mkswu/swupdate.aes-key`
 - update mkswu.conf.d/swupdate.pem with your `~/mkswu/swupdate.pem`

Note it is possible to add as many public keys as required to `/etc/swupdate.pem` on armadillo,
so it might be just as good to generate a new key just for this, but it is not possible to
have multiple AES keys.

```
$ mkswu --config-dir tmp --genkey --cn <yourcn>
# copy swupdate.pem to conf.d + armadillo's /etc/swupdate.pem
$ cp tmp/swupdate.pem conf.d/swupdate.pem
$ cat tmp/swupdate.pem | ssh g4 'cat >> /etc/swupdate.pem; persist_file /etc/swupdate.pem'
$ cat tmp/swupdate.key 
# ... set this to SWUPDATE_KEY on github secrets
$ rm -rf tmp
```

### G4 side setup

As well as the swupdate.pem public key if a new one had been generated, you need to tell the G4 what URL to poll for updates.

The default Deploy steps pushes the generated artifact to the gh-pages branch to have a stable url as actions/upload-artifacts does not have stable URLs.

For example this can be done with `enable_autoupdate.desc` in this repo,
assuming update is copied over ssh:
```
[atde ~/G4_container_update]$ vi enable_autoupdate.desc
# fix url for your use case
[atde ~/G4_container_update]$ mkswu enable_autoupdate.desc
[atde ~/G4_container_update]$ scp enable_autoupdate.swu g4:
[armadillo ~]# swupdate -i enable_autoupdate.swu
```

## Using hawkbit

It is also possible to upload the swu to hawkbit instead of github pages using
`mkswu/hawkbit_push_update` instead of actions-gh-pages in the Deploy step.

You will need to set at least `HAWKBIT_URL`, `HAWKBIT_PASSWORD` in mkswu.conf for this to work,
and setup the G4 to enable swupdate-hawkbit (see mkswu's `examples/hawkbit_register.desc`)
