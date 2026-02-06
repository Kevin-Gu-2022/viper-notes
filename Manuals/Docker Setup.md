- Follow instructions [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) using Docker's `apt` repository
- Will need to add user permissions in order to run Docker commands:
```bash
sudo usermod -aG docker $USER
```
- Make sure to reboot afterwards for it to take effect