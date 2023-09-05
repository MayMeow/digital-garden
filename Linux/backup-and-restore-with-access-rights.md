# Backup and restore using TAR and keep access rights

Can be used when you need to move folder between servers and want to keep access rights. Or just for backup. I used this commands when I want move docker volume (data folders) between servers. Used this just few times but its good to know

## To backup

```bash
tar -cpf data.tar data
```

- `-c` create 
- `-p` extract information about file permission
- `-f` use file, if its now user tar automaticallu examine nevionment viarable `TAPE`

## To restore

```
sudo tar -xpf data.tar --same-owner
```

- `-x` Extract
- `--same-owner` Try extracting files with the same ownership as exists in the archive

**❗ Both commands need to be run as ROOT**

For more information about tar you can check [Man pages](https://man7.org/linux/man-pages/man1/tar.1.html)