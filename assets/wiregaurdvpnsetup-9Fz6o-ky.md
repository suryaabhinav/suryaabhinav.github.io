# Setting Up WireGuard VPN for Hosting Backend Server with Fixed IP Access

## Introduction

This guide will walk you through setting up a WireGuard VPN to host a backend server (FastAPI with uvicorn) that can be accessed from remote locations using a fixed IP address. This setup is particularly useful for developers who need consistent access to their development server regardless of their network location.

## Prerequisites

- macOS computer (server host)
- Administrative access
- Basic understanding of networking concepts
- Homebrew package manager installed
- Python environment with FastAPI and uvicorn installed

## Understanding the Components

### WireGuard VPN
WireGuard is a modern VPN protocol that offers:
- High performance and low overhead
- Simple yet secure cryptography
- Easy configuration
- Cross-platform support

### Network Architecture
In this setup:
1. The server (your Mac) acts as the WireGuard server
2. Remote devices connect as WireGuard clients
3. All traffic between clients and server flows through an encrypted tunnel
4. Each device gets a fixed private IP within the VPN network

## Step-by-Step Implementation

### 1. Install WireGuard on macOS

```bash
# Install WireGuard using Homebrew
brew install wireguard-tools

# Create WireGuard configuration directory
sudo mkdir -p /etc/wireguard
sudo chmod 700 /etc/wireguard
```

### 2. Generate WireGuard Keys

```bash
# Generate server keys
cd /etc/wireguard
wg genkey | sudo tee server_private.key | wg pubkey | sudo tee server_public.key

# Generate client keys
wg genkey | sudo tee client_private.key | wg pubkey | sudo tee client_public.key
```

### 3. Configure WireGuard Server

Create `/etc/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey = <server_private_key>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.0.0.2/32
```

### 4. Configure WireGuard Client

Create a client configuration file `client.conf`:

```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server_public_key>
Endpoint = <your_server_public_ip>:51820
AllowedIPs = 10.0.0.0/24
PersistentKeepalive = 25
```

### 5. Setup FastAPI Server

Create a simple FastAPI server with WebSocket support:

```python
from fastapi import FastAPI, WebSocket
import uvicorn

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message received: {data}")

@app.get("/")
async def root():
    return {"message": "Server is running"}

if __name__ == "__main__":
    uvicorn.run(app, host="10.0.0.1", port=8000)
```

### 6. Start the VPN Server

```bash
# Start WireGuard server
sudo wg-quick up wg0

# Verify connection
sudo wg show
```

### 7. Generate QR Code for Mobile Clients

Create a Python script to generate QR codes:

```python
import qrcode
import sys

def generate_vpn_qr(config_file, output_file):
    with open(config_file, 'r') as f:
        config = f.read()
    
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )
    qr.add_data(config)
    qr.make(fit=True)
    
    img = qr.make_image(fill_color="black", back_color="white")
    img.save(output_file)

if __name__ == "__main__":
    generate_vpn_qr('client.conf', 'wireguard_config.png')
```

### 8. Connect Remote Clients

1. For desktop clients:
   - Install WireGuard client
   - Import the client configuration file

2. For mobile clients:
   - Install WireGuard app
   - Scan the generated QR code

## Testing the Setup

1. Start the FastAPI server on your Mac
2. Connect a client device to the VPN
3. Test WebSocket connection using a simple client:

```python
import asyncio
import websockets

async def test_connection():
    uri = "ws://10.0.0.1:8000/ws"
    async with websockets.connect(uri) as websocket:
        await websocket.send("Hello Server!")
        response = await websocket.recv()
        print(response)

asyncio.get_event_loop().run_until_complete(test_connection())
```

## Security Considerations

1. Keep private keys secure and never share them
2. Regularly update WireGuard and all dependencies
3. Use strong firewall rules
4. Monitor VPN access logs
5. Regularly rotate keys for better security

## Troubleshooting

Common issues and solutions:

1. Connection Issues
   - Verify server is running: `sudo wg show`
   - Check firewall settings
   - Verify port forwarding on router

2. Performance Issues
   - Check network bandwidth
   - Monitor server resources
   - Adjust MTU settings if needed

## Maintenance

Regular maintenance tasks:

1. Update WireGuard: `brew upgrade wireguard-tools`
2. Monitor logs: `sudo wg show`
3. Backup configuration files
4. Review and update firewall rules
5. Check system resources

## Conclusion

This setup provides a secure and reliable way to access your development server from anywhere. The fixed IP addressing within the VPN network eliminates the need to track changing local network addresses, while the WebSocket support enables real-time bidirectional communication.

Remember to regularly update all components and monitor the system for any security or performance issues.