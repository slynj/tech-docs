# Change Hostname on Linux with CLI

## 1. Check your current hostname

Check what your current hostname is with `hostnamectl` or `hostname` command.

![image.png](Change%20Hostname%20on%20Linux%20with%20CLI%20181011ff45b980e38a50eea810ec0493/image.png)

## 2. Change hostname

Change the hostname with `hostnamectl` command.

```
hostnamectl set-hostname new-hostname
```

I want to change mine to `lyn`, so 

```
hostnamectl set-hostname lyn
```

## 3. Confirm change

```
hostname
```

```
hostnamectl
```

![image.png](Change%20Hostname%20on%20Linux%20with%20CLI%20181011ff45b980e38a50eea810ec0493/image%201.png)

## 4. (Optional) Set pretty hostname

> A pretty hostname is the name displayed to the user, not the name used by other computers on a network. Computers identify each other using the static hostname.
> 

To update the pretty hostname of a machine, you can use the `hostnamectl` command with the `--pretty` option.

```
hostnamectl set-hostname new-hostname --pretty
```