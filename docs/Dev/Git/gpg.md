# tell git using gpg sign key

## generate

``` shell
gpg --full-generate-key
```

## list

``` shell
gpg --list-secret-keys --keyid-format LONG
```

## tell git to use the gpg key

``` shell
git config --global user.signingkey xxx
```

## add public key to Github

``` shell
gpg --armor --export xxx
```

go to [https://github.com/settings/gpg/new](https://github.com/settings/gpg/new) to add public key.
