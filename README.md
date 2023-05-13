# Linux Remote Desktop Workflow

This GitHub Actions workflow sets up a secure reverse SSH tunnel between a remote Linux machine and your local machine, allowing you to access the remote machine's desktop environment through a VNC viewer.

## Workflow

The workflow consists of the following steps:

1. Checkout the code
2. Install dependencies
3. Configure SSH connection
4. Set up VNC server
5. Set up reverse SSH tunnel
6. Connect to remote desktop

## Usage

To use this workflow, you will need to create a `secrets` file in your GitHub repository with the following information:

- `HOST`: The IP address or hostname of the remote Linux machine
- `USER`: The username to use for SSH authentication
- `PASSWORD`: The password to use for SSH authentication
- `VNC_PASSWORD`: The password to use for VNC authentication

You will also need to configure your VNC viewer to connect to `localhost:5901`.

## Example Workflow File

Here's an example of how you can use this workflow in your own project:

```yaml
name: Linux Remote Desktop

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Install Dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y tigervnc-standalone-server sshpass
    - name: Configure SSH Connection
      run: |
        ssh-keyscan ${{ secrets.HOST }} >> ~/.ssh/known_hosts
        echo ${{ secrets.PASSWORD }} | sshpass ssh -o StrictHostKeyChecking=no ${{ secrets.USER }}@${{ secrets.HOST }}
    - name: Set Up VNC Server
      run: |
        echo ${{ secrets.VNC_PASSWORD }} | vncpasswd -f > ~/.vnc/passwd
        chmod 0600 ~/.vnc/passwd
        vncserver :1 -localhost no
    - name: Set Up Reverse SSH Tunnel
      run: ssh -f -N -R 5901:localhost:5901 ${{ secrets.USER }}@${{ secrets.HOST }}
    - name: Connect to Remote Desktop
      run: |
        vncviewer -SecurityTypes VncAuth -passwd ~/.vnc/passwd localhost:1
