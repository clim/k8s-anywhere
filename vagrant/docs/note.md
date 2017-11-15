# Vagrant Notes

## Adhoc Commands

1. Generate Vagrant file
    ```
    vagrant init centos/7 --box-version=v1710.01
    ```

```
cat ~/.ssh/id_rsa.pub > ssh-key.pub
```


To generate the public key from a private key:
```
ssh-keygen -y -e -f "$PRIVKEY"
```
