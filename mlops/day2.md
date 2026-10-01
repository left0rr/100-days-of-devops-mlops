# Day 2 — JupyterLab Configuration
## 🎯 Task

Fix the JupyterLab configuration and start the server
on port 8888, bound to 0.0.0.0, using /root/notebooks/
as the notebook directory.

## 🧠 Concepts

- JupyterLab configuration
- Jupyter Server
- Ports and network binding
- Notebook root directory
- Troubleshooting configuration
- Running JupyterLab as root

## 🔧 Commands
# Create notebook directory
mkdir -p /root/notebooks/

# Inspect configuration
cat /root/code/jupyter_lab_config.py

# Start JupyterLab
/root/code/ml-env/bin/jupyter lab \
  --config /root/code/jupyter_lab_config.py \
  --allow-root

# Verify listening port
ss -lntp | grep 8888

# Configuration:

c.ServerApp.root_dir = '/root/notebooks/'
c.ServerApp.port = 8888
c.ServerApp.ip = '0.0.0.0'

# 💡 Key Takeaway

JupyterLab needs to be correctly configured for
network access, port, and notebook directory.

0.0.0.0:8888 means JupyterLab listens on all
network interfaces on port 8888.