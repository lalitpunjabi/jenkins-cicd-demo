# Linux Server Hardening

## 1. Introduction

Linux server hardening improves system security by reducing unnecessary access, services, and vulnerabilities.

For a DevOps environment, the main areas are:

- User and access control
- SSH security
- Firewall configuration
- File permissions
- Service management
- Docker security
- System monitoring

---

## 2. System Updates

Keep the operating system and security packages updated.

```bash
sudo apt update
sudo apt upgrade -y
````

Check the OS version:

```bash
cat /etc/os-release
```

Check the kernel:

```bash
uname -r
```

---

## 3. User Management

Use a dedicated administrative user instead of working directly as `root`.

Create a user:

```bash
sudo adduser devops
```

Add the user to sudo:

```bash
sudo usermod -aG sudo devops
```

Check user information:

```bash
id devops
```

---

## 4. SSH Hardening

Disable direct root login.

Edit:

```bash
sudo nano /etc/ssh/sshd_config
```

Set:

```text
PermitRootLogin no
```

Validate the configuration:

```bash
sudo sshd -t
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## 5. SSH Key Authentication

Generate an SSH key:

```bash
ssh-keygen -t ed25519
```

Copy the public key to the server:

```bash
ssh-copy-id devops@SERVER_IP
```

Secure the SSH directory:

```bash
chmod 700 ~/.ssh
```

Secure the private key:

```bash
chmod 600 ~/.ssh/id_ed25519
```

---

## 6. Firewall Configuration

Use a firewall to restrict unnecessary network access.

Check UFW:

```bash
sudo ufw status
```

Allow SSH:

```bash
sudo ufw allow 22/tcp
```

Enable the firewall:

```bash
sudo ufw enable
```

Allow web traffic if required:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

---

## 7. Check Open Ports

Check listening ports and services:

```bash
sudo ss -tulpn
```

Only required ports should be exposed.

Example:

```text
22    SSH
80    HTTP
443   HTTPS
```

Application ports should preferably remain internal when a reverse proxy is being used.

---

## 8. File Permissions

Check file permissions:

```bash
ls -la
```

Protect sensitive files:

```bash
chmod 600 secret.txt
```

Protect sensitive directories:

```bash
chmod 700 sensitive-directory
```

SSH files should also use restrictive permissions.

---

## 9. Service Management

List running services:

```bash
systemctl --type=service --state=running
```

Check a service:

```bash
sudo systemctl status SERVICE_NAME
```

Stop an unnecessary service:

```bash
sudo systemctl stop SERVICE_NAME
```

Disable it:

```bash
sudo systemctl disable SERVICE_NAME
```

Only disable services that are not required.

---

## 10. Docker Security

Check running containers:

```bash
docker ps
```

Inspect a container:

```bash
docker inspect CONTAINER_NAME
```

Avoid using privileged containers unless required:

```text
--privileged
```

Docker access should also be restricted because access to the Docker daemon provides significant host-level privileges.

---

## 11. Jenkins Security

Jenkins should be protected because it can execute commands on the CI/CD server.

Recommended practices:

* Use Jenkins authentication.
* Store secrets in Jenkins Credentials.
* Keep Jenkins and plugins updated.
* Avoid running Jenkins as `root`.
* Restrict Jenkins network access.

Example credential usage:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'dockerhub-credentials',
        usernameVariable: 'DOCKER_USERNAME',
        passwordVariable: 'DOCKER_PASSWORD'
    )
]) {
    // Secure commands
}
```

---

## 12. Log Monitoring

Check system logs:

```bash
journalctl
```

Check SSH logs:

```bash
journalctl -u ssh
```

Follow logs in real time:

```bash
journalctl -f
```

Check recent login activity:

```bash
last
```

Logs should be reviewed for unexpected or suspicious activity.

---

## 13. Disk Monitoring

Check disk usage:

```bash
df -h
```

Check Docker disk usage:

```bash
docker system df
```

Low disk space can cause Jenkins, Docker, and application failures.

---

## 14. Security Checklist

```text
[ ] System updated
[ ] Dedicated user created
[ ] Root SSH login disabled
[ ] SSH keys configured
[ ] Firewall enabled
[ ] Unnecessary ports closed
[ ] Unnecessary services disabled
[ ] File permissions reviewed
[ ] Docker access secured
[ ] Jenkins credentials protected
[ ] Logs monitored
[ ] Disk usage monitored
```

---

## 15. Conclusion

Linux hardening provides a secure foundation for DevOps infrastructure.

The key principles are:

```text
Least Privilege
      +
Secure Authentication
      +
Firewall
      +
Minimal Services
      +
Secure Permissions
      +
Continuous Monitoring
```

These practices complement the security controls implemented in the Jenkins CI/CD pipeline.